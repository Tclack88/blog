---
layout: post
title:  "Smart Intersection - A first attempt"
date:   2020-06-07 21:01 -0700
img:	assets/intersection/time_requests_smart_intersection.png
blurb:  I've been experimenting with adding a GUI to make some of my programs more accessible. After all not everyone is willing to open code and run it in a terminal. I feel this would make a product that can be used by the more general consumer
categories: code
---

[See the source code on GitHub](https://github.com/Tclack88/practice-scheduler)

One of the first things I wanted to do when I started programming was to make a "smart intersection" simulation, that is an intersection that allows cars to pass through without needing to stop at lights. I wasn't sure when I'd have the capability to program such a thing, but all it took was a job post requesting a code sample to start the ball rolling.

non-standard dependencies:

		pygame		-       pip install pygame

## Inspiration

I hate driving. There's nothing more frustrating then spending your life commuting from one place to another, especially if that place is somewhere you don't want to be. It's fun at first when you're 16 and suddenly gain this independence you never had. But when that feeling wanes, all you're left with is a time sink during which you have to watch cars and lines in the road. A boring task that if performed incorrectly can leave you seriously injured or worse. 

I'm sure we've all heard the statistic that the average American spends about 40 hours per year stuck in traffic, which is a lot, but that's just traffic. If you consider how long you're driving. Just think about your average day. Do you spend 1 hour driving? Well that's 1/24 of your life! (Or rather,the 50 years or so that you're actively working and need to commute). I hate this time sink so much that I try to fill it with podcasts or phone conversations, something that will return that time back to me. The thing that makes driving for me even worse (apart from traffic) is coming to a sudden stop.

It's that unbearable feeling I get when I'm crusing down a highway at 45 mph only to see a yellow light bring me and my nearby convoy to a hault. All so this one dinky Subaru can make a left turn. It hurts because of all the gas we all squandered away to friction and heat. The gas that's wasted while we all idle, then more as we accelerate back to speed. I try my best to predict such things, coast when I approach a light from afar (to the annoyance of the vehicle behind me), I even slow my approach to lights when I think I'll cause a group of cross traffic to stop. The traffic light was invented in 1912. There has to be a better way.

## The slot system

The way I envisioned this was fairly straightforward. When a car crosses a critical threshold from an intersection. It makes a request to pass through during a specific time interval. That request is based on the approaching velocity and the distance from the intersection. A controller listens to this request and checks it against its current reservations. If there are no conflicts, the car continues along at the velocity it's going at (I'm only dealing with straight-through traffic, no turning yet. Sorry to spoil it). In the even that there is a conflict, the controller checks for an opening with that requested time difference, otherwise it's slotted at the end. The car is then given specific speed instructions to follow, speeding up or slowing down to make the slot. That's it.

|![Smart interssection theory]({{ site.baseurl }}/assets/intersection/time_requests_smart_intersection.png)|
| A car makes a request but there's a conflict. Some slight speed adjustments will resolve that conflict, allowing for safe and efficient passage |


## Steps to creating this

d

## Shortcomings

- same direction cars "pass-through" when the time gap in the front is large

- no acceleration

- backup - leading to random choices

## MIT beat me to it (and obviously did a lot better)
