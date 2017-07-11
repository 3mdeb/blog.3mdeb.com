---
author: Michał Olszewski
layout: post
title: "Kanban method in a consulting company"
date: 2017-05-12 12:00:00 +0100
comments: true
categories: Managament 
---

# Kanban method in a consulting company



Before we start we need to know what is that Kanban. Let me show you a short
explanation (according to [Wikipedia](https://en.wikipedia.org/wiki/Kanban)):

> **Kanban** is a method for managing resources necessary for production process
so that everyone produces much as need at a given moment. Work items are
visualized to give participants a view of progress and process, from task
definition to customer delivery. Team members "pull" work as capacity permits,
rather than work being "pushed" into the process when requested.

> In software development, Kanban provides a visual process-management system
which aids decision-making about what, when and how much to produce. Although
the method (inspired by the Toyota Production System and lean manufacturing
originated in software development and IT, it may be applied to any professional
service whose work outcome is intangible rather than physical.


## Tools

The basic work tools are boards and stickers, but in our company we use visual
boards in [Trello](https://trello.com/), which streamlines our work in kanban.
Our boards are divided into appropriate categories: Backlog, To Do, In Progress,
Verification, Done and Project Status, where we can discuss project features
or problems.

![1](http://imageshack.com/a/img922/6156/YfxSE1.png)

## Cards (Kanbans)

According to the Kanban method we give out a kanban cards. Every card must have:

* Title - As needed, it can be very general or low-level,
* Description - With some documentation, links or to do list,
* Asignee - If you want to plan something with others You can assing more than
one person, but we recommend to assing one person / per task,
* Label - According to the task,
* Due date - We set a maximum of two weeks to complete the tasks.

The card is very important in the whole project process. That way, we measure 
our work and set time for each stage in the design, develop and testing 
solutions in embedded systems projects. Additionally inside the
card we can discuss the features and see, what happened since last time.

![img](http://imageshack.com/a/img924/2586/spqDrv.png)

## Workflow
 
The card life begins in Backlog, tasks here have lowest priority and they
waiting for select to development - but it's not a rule, sometimes we add cards
with high priority directly to ToDo column. ToDo is the queue for the most
important column - In Progress. From there our goal is to safely move this task
to done. By analyzing the boards we are able to find the blocker very quickly.
In embedded systems company, we are dealing mostly with blockers in the form
not working or missing hardware, incompatibility of the kernel with hardware and 
frequent changes in priorities.

![kanban-flow](http://imageshack.com/a/img923/2471/u20Zzr.jpg)

## Rules

To make kanban work, each employee must follow these rules:

* In Progress have biggest priority, next is ToDo and Backlog,
* Never take more than two tasks at a time - it is interesting to note that when
switching between two tasks we need about 13 minutes to return to the previous
concentration,
* Do not throw unfinished tasks for verification, because due date is comming,
* Listen and read communication, ask questions
* Discuss inside cards - about problems and progress,
* Suggest changes in places where you see such a need.

These rules are only a sample but improve the whole process. We know situations
where tasks is going fast to verification, after which it turns out that the
task is incomplete and we need turn it back to the left. When the checks show
such errors, we try to first determine the degree of completion of the card, and
then decide whether the card must return or sometimes we split cards and move 
part of result to Verification and other part to ToDo. In our field there are 
frequent changes, a very good flow of information is important to save 
unnecessary work.

## Pros and cons

The advantages are:

* Universal - can be used to visualize company overview, project or employee
individual workflow
* Creating a WIP (Work In Progress) limit - it prevents overproduction and 
quickly detects bottlenecks
* Clarity - one look and you know the whereabouts of the project
* Stream management - systematic time and effort measurement to optimize
processes.

 And cons:

* Need for continuous control
* Commitment of each employee
* Incorretly managed will turn into a battlefield where employees struggle, who
has done more and create a nasty mess on the work cards.

## Summary

Kanban is designed for companies looking for ways to optimize task allocation
and for teams that are overloaded with tasks. It works because introducing lower
task settlements and by the way it adds very good tools for measuring work
progress and overproduction managing. Our specialization requires the division
of tasks into very low levels and the rules of work are very similar to the
production where the downtime blocks the whole group, so choice of Kanban was
very useful in our company. By using it, we have increased productivity, order
in the organization of our tasks and make it easy to find the needed information
by reducing the volume of communication and grouping it into individual pages.
At this moment we are able to follow tasks of every employee in our company
individually so now we don't have to look for another tool but maybe in future
we will try to mix some metodologies for better work.
