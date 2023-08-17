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

## Solution - observer pattern

## Observer patter in Poco - Events and DelegatesS
