---
layout: post
title:  "Smart Intersection - A first attempt"
date:   2020-06-07 21:01 -0700
img:	assets/intersection/smart_intersection_screenshot.png
blurb:  It’s that unbearable feeling I get when I’m cruising down a highway at 45 mph only to see a yellow light bring me and my nearby convoy to a halt. All so this one dinky Subaru can make a left turn. It hurts because of all the gas we all squandered away to friction and heat to brake, the gas that’s wasted while we all idle, and then more as we accelerate back to speed. I try my best to predict such things, coast when I approach a light from afar (to the annoyance of the vehicle behind me), I even slow my approach to lights when I think I’ll cause a group of cross traffic to stop. The traffic light was invented in 1912. There has to be a better way.
categories: code
---

[See the source code on GitHub](https://github.com/Tclack88/smart_intersection)

One of the first things I wanted to do when I started programming was to make a "smart intersection" simulation, that is an intersection that allows cars to pass through without needing to stop at lights. I wasn't sure when I'd have the capability to program such a thing, but all it took was a job post requesting a code sample to start the ball rolling.

non-standard dependencies:

		pygame		-       pip install pygame

## Inspiration

I hate driving. There's nothing more frustrating then wasting your time commuting from one place to another, especially if that place is somewhere you don't want to be. It's fun at first when you're 16 and suddenly gain this independence you never had. But when that feeling wanes, all you're left with is a time sink during which you have to watch for cars and follow lines in the road. A boring task that if performed incorrectly can leave you seriously injured or worse. 

I'm sure we've all heard the statistic that the average American spends about 40 hours per year stuck in traffic, which is a lot, but that's just traffic! Have you ever considered how much time you spend driving? The easiest way to calculate this is to multiply the number of minutes you drive each day by 6, then call it hours. Even an unrealistic 1 minute per day is *6 hours* per year. I hate this time sink so much that I try to fill it with podcasts or phone conversations, something that will return some of that time back to me. The thing that makes driving for me even worse (apart from traffic) is coming to a sudden stop.

It's that unbearable feeling I get when I'm cruising down a highway at 45 mph only to see a yellow light bring me and my nearby convoy to a halt. All so this one dinky Subaru can make a left turn. It hurts because of all the gas we all squandered away to friction and heat to brake, the gas that's wasted while we all idle, and then more as we accelerate back to speed. I try my best to predict such things, coast when I approach a light from afar (to the annoyance of the vehicle behind me), I even slow my approach to lights when I think I'll cause a group of cross traffic to stop. The traffic light was invented in 1912. There has to be a better way.


## The slot system

The way I envisioned this was fairly straightforward. When a car crosses a critical threshold from an intersection. It makes a request to pass through during a specific time interval. That request is based on the approaching velocity and the distance from the intersection. A controller listens to this request and checks it against its current reservations. If there are no conflicts, the car continues along at the velocity it's going at (I'm only dealing with straight-through traffic, no turning yet. Sorry to spoil it). In the event that there is a conflict, the controller checks for an opening with that requested time difference, otherwise it's slotted in at the end. The car is then given specific speed instructions to follow, speeding up or slowing down to make the slot. It's that simple.

|![Smart intersection theory]({{ site.baseurl }}/assets/intersection/time_requests_smart_intersection.png)|
| A car makes a request but there's a conflict. Some slight speed adjustments will resolve that conflict, allowing for safe and efficient passage |


## Steps to creating this

It's always fun to see the development process rather than just the final product. Here's a brief compilation of screen recordings I took of the project at various stages during its development.


<iframe width="560" height="315" src="https://www.youtube.com/embed/AWXWtLZF7qg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>
For a bit more detail, here are some steps either glossed over or omitted entirely from the video:

**The simulation is performed with Pygame**

I really only used its built-in rectangle class because it has a useful method call to detect when any rectangles overlap. This is particularly useful for determining when a crash occurs (not implemented) or when a boundary has been crossed. This would would have taken too much time to write from scratch (much like the 1 minute/6 hour rule, if I think it'll take half a day to create, it probably will take me half a week). I used this to initiate the crossing requests from the car to the intersection controller. Also, all intersections are created with the road rectangle overlaps.

**The class hierarchy starts from the roads**

The roads spawn the cars because it's the most straightforward way of making a car move along a specified path rather than hardcoding it for each road configuration. The intersections are created automatically from the road overlaps, as well as each intersection's controller of course. The advantage of this is that I only need to specify the road position and orientation (e.g. `Road('v',.5)` &#x2190; vertical road in the middle of the screen), then everything else is taken care of.

**The time requests take into account the length of the car**

Fortunately due to the crossing request occurring as a boundary is crossed rather than a specific point of the car. I know it will always occur when the nose of the car has just crossed the boundary\*, so I know the pixel distance is the sum of the intersection length, the distance to intersection and the car length. 

\*Given that pixels on a screen are discrete and velocity is determined by how many pixels are jumped after each frame, this simulation doesn't accurately portray the probably continuous nature of the universe. This is a shortcoming because the car is "further ahead" than assumed, which can convert "near misses" into actual collisions if the controller just barely approves the requested time slot. (see the 3rd point below in the next section)

## Shortcomings

**Ghost cars**

Cars travelling in the same direction may perform a pass-through if the car in front is moving slowly and a reasonable speed adjustment can be made by the rear car to move ahead. This works fine when there's slower cross traffic and the car can boost its speed to "beat" the slower car. This can be resolved when a more general set of allowed crossings simultaneous crossings are coded.

**No acceleration.**

The cars will receive velocity orders and immediately change speeds. This manifests as jerky movement. This can be addressed by implementing an acceleration method that adjusts by 1 px/sec each frame such that the average speed since the speed order was issued matches the speed, but I fear this level of detail won't bring much until the issue of speed is resolved. What issue of speed? Well...

**Only integer amounts of speed are allowed.**

This makes sense because what is a computer supposed to do when it's told to advance a sprite by 29.48 pixels? Here of course it rounds (Implicitly. No errors are thrown if I leave it as a float). A potential fix is to let the car accelerate (see previous point) as to maintain that fractional speed most closely as an average of integer speeds. If a car approaches an intersection with a speed of 32 px/sec and it is instructed to go at 29.48 px/sec, It's a very simple algebra to figure out what the missing value ought to be. Sprinkle in some looping and logic and here's how to generate a list of 30 successive speeds for a car to follow:
```python
vels = [32]
v = 29.48
for i in range(30):
    next_v = v * (len(vels) + 1) - sum(vels)
    if next_v < vels[-1]:
            vels.append(vels[-1] - 1)
    elif next_v > vels[-1]:
            vels.append(vels[-1] + 1)
    else:
        vels.append(vels[-1])

>>> [32, 31, 30, 29, 28, 27, 28, 29, 30, 31, 30, 
29, 30, 29, 30, 29, 30, 29, 30, 29, 30, 29, 30, 
29, 29, 30, 29, 30, 29, 30, 29]
```
This is a better solution if there's a long distance the car must travel. For big adjustments or relatively short distances (such as crossing the intersection), this isn't as helpful. I'm mostly throwing it in here for my future self. Regardless, it deals with the acceleration issue and rounding errors all at once (the original code would just suggest the car to move at 29 px/sec which at the end of this results in a deficit of 16 pixels, about half a car length, plus it would be jerky). More brainstorming: perhaps the adjustments can be made to be 2 or 3 pixels per frame, depending on just how far away the requested speed is from the current speed.

**Backup/Traffic**

So as the video states, I fooled myself into thinking it was working at one point. My mind even saw a car slow down or speed up when it really didn't (see the red/green cars in the video, they're moving at constant speeds). Once I made the cars turn blue only when they were following commands from the controller, I noticed something. Those videos would start out with a lot of backup early on then it would just clear out. This wasn't the program  working itself out; what was happening in fact was that cars' requests were denied and the suggested crossing time was pushed to the end. As time went on, the time suggested time slots were so far away (40+ seconds) that the cars were just driving and not receiving commands to slow down until a time that would occur long after they were destroyed. The result was what you saw: Random crossings that only appeared to be working and occasional "accidents". I fixed this by implementing the "allowed pairs" rule, (e..g. A car going N to S should be unaffected by a car that reserved a S to N crossing. It will only get better as I implement turning, speaking of.

**No turning exists**

This is a very good example of the 80/20 rule; getting this simulation to work with only vertical and horizontal movement is easy. Adding in the ability to turn brings along a slew of other issues:
- More complex rules for allowed pair crossing
- Lane tracking
- Animating the rotation

This is clearly an endeavor that will take me more than a few days, but I'll have to push these to another time.

## MIT beat me to it (and obviously did a lot better)

Finally, why am I doing this? As soon as I thought of the idea, I looked it up and found that researchers at MIT solved this problem in 2014, check out their much higher quality work:

<iframe width="560" height="315" src="https://www.youtube.com/embed/kh7X-UKm9kw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>
But the process of discovery is more appealing to me than the final product. I'm sure that's the same with any endeavor any of us pursue; there's always someone who can and probably has done a better job. But what we learn in the process of figuring it out ourselves is far more valuable than the product we make. Thanks for reading.
