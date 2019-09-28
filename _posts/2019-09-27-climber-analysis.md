---
layout: post
title:	"Trad Climbers, Sport Climbers and Boulderers: an Analyis"
date:	2019-09-27 11:09:00 -0700
categories: personal
---

## Who Climbs best?

![all climbers]({{site.baseurl}}/assets/mproj/trad_boulder_sport.jpeg)

## Introductions - for those unfamiliar with climbing styles
### The Noble Boulderer
![bolderers]({{site.baseurl}}/assets/mproj/boulderer.jpg)
Bouldering is almost a spiritual persuit. It's man vs. the rock. Boulderers typically climb short, intense routes. They need no rope for protection, merely a pad and possibly some spotters.
### The O.G. Trad Climber
![trad climbers]({{site.baseurl}}/assets/mproj/trad_climber.jpg)
Trad (traditional) climbers, as the name implies was the original style of climbing, placing protection into the wall as they go. They need to follow cracks or at the very least, climb walls with enough features where safety gear can be placed frequently.
### The light and fast Sport Climbers
![sport climbers]({{site.baseurl}}/assets/mproj/sport_climber.jpg)
Sport climbers have no time to think about anchoring themselves into the wall, they need only think about the climb ahead. Unlike trad climbers, sport climbers are not restricted to following cracks, bolts placed on otherwise featureless terrain allow them to safely protect their climbs.

<br>
<hr>
We can devise all sorts of arguments about who climbs the best:
> "boulderers focus on climbing only the hardest possible sequences, routes are nothing for them" 

> "Sure but boulders only travel 15 feet, they lack the long term endurance of a trad climber" 

> "Trad climbers spend too much time placing gear, and they have to carry so much weight"

> "sport climbers get to focus on the climbing and don't have to worry about gear placement" 

> "Yeah, but sport climbers have no ethics, they spawn from the gym and inflate the grades"

This kind of arguing back and forth will get us nowhere, let's see what the data says
<hr>
<br>
## Analysis
I randomly sampled around 3200 climbers from a popular climbing website [Mountain Project](https://www.mountainproject.com/) and here's what I've found. 

(Full disclosure, determining who is a boulderer, sport climber, and trad climber is somewhat subjective. I've defined sport climbers as those who have not listed any trad climbs on the website and vice versa, they may do a fair amount of bouldering, but it's difficult to tell. Boulders have no sport or trad climbs ticked, but they have routes, and these were likely top-roped which comes with its own psychological advantage. Boulders also make up the minority of the three groups in this dataset)

### Bouldering
Boulderers of course possess the peak values in their domain, bouldering. But they are on par with route climbers when we average out their max bouldering grades. Trad climbers fall behind in this regime, but sport climbers are about equal to the boulderers once averaged.

![trad climbers are weak boulderers]({{site.baseurl}}/assets/mproj/strongest_boulders.png)

And their average max grades by discipline

![trad climbers are weak boulderers]({{site.baseurl}}/assets/mproj/trad_climbers_weak_boulderers.png)


### Route Climbing
When it comes to route climbing, sport climbers dominate the field. They sit consistantly on the high end above trad climbers and boulderers.
![sport climbers dominate route climbing]({{site.baseurl}}/assets/mproj/sport_climbers_dominate_routes.png)

And their average max grades by discipline

![sport climbers dominate route climbing]({{site.baseurl}}/assets/mproj/sport_climbers_dominate_routes2.png)

(note: it may seem like the data is being "skewed" because these graphs aren't starting at 0, but if you don't climb you'll need to trust me. There is no significat difference between 5.0 to about 5.7 . Most gyms don't set routes below 5.6 or 7 for this reason. This was originally thought of as a terrain classification than a scale for climbing difficulty)

## A Brief Possible Explanation

It may come as no surprise that the more you climb, the better you get. Plotting the mean grade against the max grade. Now unless you don't have many climbs, your mean grade will just about equal your mean grade. But the higher that max grade sits (i.e. you sit above that diagonal), the better the indication of a strong climber.

I eliminated a lot of noise (climbers who only posted only couple of climbs), then sized their dots by how many total climbs they've done in their life, and the dots that sit the highest are the ones which climb the most.

![The more you climb, the better you climb]({{site.baseurl}}/assets/mproj/more_climbs_raise_grade.png)

## Tl;dr
Climb a lot and focus on sport climbing to charge up those grades!

<br>
<hr>


This data is open source, so here's a GitHub link to the [data, datascraping and analysis code](https://github.com/Tclack88/MountainProject)
