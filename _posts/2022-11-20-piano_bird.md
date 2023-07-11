---
layout: post
title:  "Piano Playing Bird"
date:   2022-11-20 21:01 -0700
img:    assets/mechanical/mechanical_bird_thumbnail_final.png
blurb:  In a group, I designed and built a mechanical piano-playing "bird". It's less functional and exciting than it sounds however. There were much bigger ideas, however plans and extensions were scraped in the interest of time. That doesn't stop the fact that it happened. Watch a video overview of the project here! 
categories: engineering
---

Unlike most posts on here, this project was not code-heavy, rather it was a physical engineering feat. I was in a mechanical system design project (at the University of Melbourne) and we were tasked with creating an interactive "pet". After some discussion of mutual interests (music), we opted to make a pet that played the piano. A bird was chosen simply because it was the only animal our unimaginative brains could conjur that would strike a key note by pecking.

Credit where credit is due: The captivating images of the title were AI generated using a [huggingface](https://huggingface.co/spaces/stabilityai/stable-diffusion) stable diffusion model.

# Demo

Please enjoy this short video I made briefly explaining the project as well as a fun demonstration of it in action.

<iframe width="560" height="315" src="https://www.youtube.com/embed/YjfrNXyN_rI" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<br>


# The Design Process

Throughout the project, components were designed and manufactured from scratch, using a laser cuter on MDF (Medium Density Fibreboard) or acrylic. This involved measuring and calculating the forces involved in pressing a key down on the electric keyboard available to us and calculating the torques and required motor power. Much of those mathematical details are too cumbersome to type up and in the end there was still a fair bit of trial and error with a splash of analytics.

Beyond designing parts, as an attempt to practice the [3 R's](https://en.wikipedia.org/wiki/Waste_hierarchy), we salvaged components from previous disassembled projects, such as the primary cam performing the action, a few gears and an aluminum shaft we needed to tap to connect with other components.

# The Shortcomings and Excuses
> I've always believed that education gets in the way of true learning, and this simply confirms it.

I thought it would be really neat if the bird would hear a note then move to the desired key and then play it. I had written a note detector in python, the idea was to use python as a sort of API and the outcome of the note detection would then go to the Arduino an instruct it exactly how far to move to reach the note it heard. That unused code is available to browse [here](https://github.com/Tclack88/AIMusic/tree/master/music_note_detection). (Also featured is the unused clap detector script. That ultimately was just enabled in the Arduino code). What happened with the plan? Our motor simply was not powerful enough. To imagine a motor of equal power moving the full platform would have been too daunting in the short time we had left. This university considers 4 simultaneous classes as full time and there simply wasn't enough time to commit on top of it all. I've always believed that education gets in the way of true learning, and this simply confirms it.
