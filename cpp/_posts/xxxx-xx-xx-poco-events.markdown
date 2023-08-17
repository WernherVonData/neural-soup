---
layout: post
title:  "Poco Events"
date:   xxxx-xx-xx 00:00:00 +0200
categories: cpp update
---

## Introduction

[Poco](https://pocoproject.org/) is a C++ library providing a lot of useful tools. One of them is the event system. In this post I'll show briefly how to use it.

## Go and do push-ups

Let's imagine we have a trainer who is telling the students to do push-ups. Trainer knows perfectly at the time of code execution that student can do 41 push-ups, so he'll challange him to do 42. Without Poco you could do it like this:

```cpp
class NormalStudent {
  public:
  void doPushUps(int n) {
    std::cout << "Normal  Student: I have to do: " << n << " push-ups." << std::endl;
  }
};

class NormalTrainer {
  public:
  void sayNumberOfReps(NormalStudent& student) {
    student.doPushUps(push_ups_nb_);
  }
  private:
  int push_ups_nb_ = 42;
};

int main() {
  NormalTrainer normal_trainer_;
  NormalStudent normal_student_;
  normal_trainer_.sayNumberOfReps(normal_student_);
}
```

This approach adds particular problem - direct coupling. The trainer must know and operate directly on the particular student. In case of many students or additional trainers operating on students your code might get really messy.

## Potential solution - observer pattern

Observer pattern is pattern where have object that is make observable that has its observers.
When the observable object changes something it notifies all its observers about the change.

In our case we can make the trainer observable and the student observer. The trainer will notify the student about the number of push-ups he has to do. Trainer can have a lot of different students and will send to all of them the same information.

```cpp
class TrainerObserver
{
public:
  virtual void onNumberOfReps(int n) = 0;
};

class TrainerObservable
{
public:
  void addObserver(TrainerObserver* observer) { observers_.push_back(observer); }
  void removeObserver(TrainerObserver* observer)
  {
    observers_.erase(std::remove(observers_.begin(), observers_.end(), observer), observers_.end());
  }
  void notifyObservers(int n)
  {
    for (auto& observer : observers_) {
      observer->onNumberOfReps(n);
    }
  }

private:
  std::vector<TrainerObserver*> observers_;
};

class StudentObserver : public TrainerObserver
{
  public:
  void onNumberOfReps(int n) override
  {
    std::cout << "StudentObserver: I have to do: " << n << " push-ups." << std::endl;
  } 
};

int main() {
  TrainerObservable trainer_observable_;
  StudentObserver student_observer_;
  trainer_observable_.addObserver(&student_observer_);
  StudentObserver student_observer2_;
  trainer_observable_.addObserver(&student_observer2_);
  trainer_observable_.notifyObservers(42);
  trainer_observable_.removeObserver(&student_observer2_);
  trainer_observable_.removeObserver(&student_observer_);
}

```

As a result we will get:
```
StudentObserver: I have to do: 42 push-ups.
StudentObserver: I have to do: 42 push-ups.
```

This approach however makes us to write observer interface and observable class. It's not a lot of code, but we can still get it a little easier.

## Observer pattern in Poco - Events and Delegates

Poco in it's core (foundation) module offers us an event system. It's based on delegates and events. Delegate is a class that holds a pointer to a method. Event is a class that holds a list of delegates. When the event is fired it calls all the delegates that are registered to it.

Despite the fancy name in its internals event system uses observer pattern with C++ templates, to staticaly generate code for each delegate. It's a little bit more complicated than that, but it's not the point of this post.

Let's see how we can use it in our example:

First the includes:

```cpp
#include <Poco/BasicEvent.h>
#include <Poco/Delegate.h>
```

The trainer class:

```cpp
class EventTrainer
{
public:
  Poco::BasicEvent<int> reps_event_;
  void sayNumberOfReps(int n) { reps_event_(this, n); }
};
```

Trainer holds `Poco::BasicEvent<int>` which is a list of delegates that take `int` as an argument. The trainer can fire the event by calling `reps_event_(this, n)`. The first argument is the sender of the event (the `EventTrainer`), the second is the argument of the event.

The student class:

```cpp
class DelegateStudent
{
public:
  void onNumberOfReps(const void* sender, int& arg)
  {
    std::cout << "DelegateStudent: I have to do: " << arg << " push-ups." << std::endl;
  }
};
```

The student class holds a function onNumberOfReps which has the same signature as the delegate in `EventTrainer`.

The main function:

```cpp
int main () {
  EventTrainer trainer;
  DelegateStudent student;
  trainer.reps_event_ += Poco::delegate(&student, &DelegateStudent::onNumberOfReps);
  trainer.sayNumberOfReps(42);
  trainer.reps_event_ -= Poco::delegate(&student, &DelegateStudent::onNumberOfReps);
}
```
In main we are creating a trainer and a student. Then trainer event object adds over overloaded operator+ a delegate to the student. The delegate is created by `Poco::delegate` function which takes a pointer to the object and a pointer to the method. The method must be public and have the same signature as the delegate.

Then the trainer fires the event and the student receives notification. At the end the student is removed from the event.

This approach is much easier than the previous one. We don't have to write any observer interface or observable class. This way we can have different types of object that do not inherit over one interface - the only one requirement is that the method has the same signature as the delegate. 

To be honest we still have here a little inheritance - our `DelegateStudent` is wrapped in `Poco::delegate` call, but it's done internally over the Poco library. DelegateStudent is not inheriting from any interface.

Is the inheritance bad? No, but overuse of inheritance can lead to design problems in a long run. In this case we can avoid it.

## Conclusion

Poco events are a nice way to implement observer pattern. They are easy to use and don't require any inheritance. They are also thread safe. If you want to know more about Poco events you can read the [documentation](https://docs.pocoproject.org/current/Poco.AbstractEvent.html).

## References

* [Poco Events](https://docs.pocoproject.org/current/Poco.AbstractEvent.html)
* [Poco Delegates](https://docs.pocoproject.org/current/Poco.AbstractDelegate.html)
* [Poco Foundation](https://docs.pocoproject.org/current/00100-GuidedTour.html)