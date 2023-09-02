---
layout: post
title:  "Quest for Legends - menu bar height"
date:   2023-09-01 08:00:00 +0200
categories: qfl update
---

## Introduction

The UI elements in the game is powered by the [imgui](https://github.com/ocornut/imgui) library. It's pretty powerful library that gives you a lot of possibilities and integrate well with different rendering engines.

Currently the menu bar is neatly placed and the tiles are redered below him.
![Menu bar]({{site.url}}/{{site.baseurl}}/qfl/img/2023-08-29-menu-bar.png)

However in the beginning it doesn't look like this, the menu bar was drawn over the tiles:
![Menu bar bad]({{site.url}}/{{site.baseurl}}/qfl/img/2023-08-30-menu-bar-bad.png)

It's definetely a visual problem that had to be solved in some way.

## Solution

Solution is pretty straightforward: by knowing the height of the bar, I can draw the tiles on screen just below the bar. The height of the bar is known during the runtime. By call to `ImGui::GetWindowSize()` which provider a window object of currently drawing element (menu bar in this case).

However the code responsible from drawing the tiles and UI is separated into two different classes.
To handle it I created a `RenderingUtils` class responsible to act as a container for different information that is shared between rendering classes (e.g. the size of the tile which is used both to draw the tiles, but also to fit the player and npc sprite).

But how to inform the `RenderingUtils` about the size of the menu bar? I could pass the `ImGui::GetWindowSize()` to the `RenderingUtils` class, but it would be a bit messy. Instead I decided to use the observer pattern. Created a `ImGuiDataObserver` that sends a notification to the `RenderingUtils` class when the size of the menu bar is known/changes.

Diagram below shows how the class are connected:

![ImGuiDataObserver]({{site.url}}/{{site.baseurl}}/qfl/img/2023-08-30-imgui-data-observer.png)

## Conclusion

The solution is pretty simple, but it shows how to effectively use the observer pattern to communiate indirectly between different classes.

## Reference

- [Observer Pattern](https://refactoring.guru/design-patterns/observer)