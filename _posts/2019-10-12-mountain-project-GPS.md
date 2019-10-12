---
layout: post
title:  "Mountain Project GPS"
date:   2019-10-12 11:20:00 -0700
categories: code
---

The following is a write-up for my project to gather area-specific GPS coordinates for ease of use with a GPS device (when the area will be out of a cell service zone). The code can be found one my GitHub [here](https://github.com/Tclack88/MountainProject).

Non-standard python libraries and Linux utilities needed:

        BeautifulSoup   -       pip3 install python3-bs4
        gpsbabel        -       sudo apt install gpsbabel



## The final product

Easy, no-effort gathering of climbing area GPS coordinates and route details.

![climbing areas stored in GPS]({{site.baseurl}}/assets/mproj/gps_collage.png)
## Inspiration   


I got into geocaching after I got out of the Navy, it's one of the few of many things I wrote down in a "post-freedom to-do list" (climbing was one of the others that didn't go into the idea graveyard that was the destination of scuba diving certification and spelunking). As a result I got a GPS which not only makes the geocaches more accurate, but also mostly free. I haven't cached a ton, but since I stopped doing it consistently, I still have h
ad plenty of use for my GPS.

When I'm out on a trip and I'm trying to find a wall within a larger climbing area, I found out I could click "navigate to" from the Mountain Project app and my phone would show the GPS coordinates. I could just manually plug them in and be on my way. That worked well f
or a while, but it's annoying and can be sped up.

Another thing I used to do when I geocached actively was plan out a big bike route and find all geocaches along the way. I found it was annoying to plug in the coordinates for the next cache manually so I figured out how to format a text file with the header: latitude, longitude, name, and comment, then uploading it to a website like [GPSvisualizer.com][gpsvisualizer.com]. While that is great, it adds a step to the process and I want to avoid any manual transaction that I wish to avoid.

These experiences came together to create the perfect storm that gave birth to the idea:
Let's create something which scrapes all routes in a specified area and compile it into a gpx file so I no longer need to plug the walls in manually when I'm lost.


## Code Overview

Much of this is HTML parsing with beautiful soup, interacting with the Mountain Project API and manipulating dataframes. An initial URL serves as the "main" URL from which all sub-areas and routes therein.

I noticed that when I'm in a branch (an area with sub areas) the areas are stored in a \<div> with class `lef-nav-row`. So in these instances, I get deeper sublinks by recursively calling `find_sub_areas`.

When I'm at the lowest where no more sub areas are displayed, there's instead a table with id `left-nav-route-table`. In this instance I call `find_routes` which parses a soup object for all the links (which are just in \<a> tags, fortunately the only ones so there's no hoops to jump through). `find_routes` is called within `find_sub_areas` and ultimately is what returns the URLs to all routes.

Now, we don't really need the URLs because of the existence of the API, so no more BeautifulSoup parsing is required. The only thing we need from these URLs are the 9 digit route ID's and getting these are as simple as dealing with regex.

The API requires route ID's in groups of 100 or less, so I break them up into lists of 100 with `group_routes`. My assumption/hope is that if I have something like 158 routes, it will be considered as just two requests, this shouldn't overload the API too much and thus should still heed their admonition:
> "We track every request — be sure your code caches data and does not make excessive requests or your account will be deactivated."

The output of the API request is a JSON object. All the routes can be accessed by just calling it like a dictionary entry `request.json()['routes']` (forgive me if this is basic, but this is among my first times dealing with JSONs). The dictionary therein contains all the routes which can be easily turned into a dataframe and parsed with pandas for only the relevant information.

I then just created a string and saved that as a csv file. I'm now unsure why I did that rather than just saving the dataframe as a csv, probably something to do with groupby objects being confusing. Regardless, at this point, gpsbabel completes the final conversion.

## Lessons Learned

I had a little trouble working the recursion in my `find_sub_areas` function. The final, kind of ugly version is
``` python
def find_sub_areas(url):
    source = requests.get(url).text
    soup = BeautifulSoup(source,'html.parser')
    sublinks = []
    lef_navs = soup.find_all('div',class_='lef-nav-row')
    if len(lef_navs) == 0:  
        return find_routes(url)
    for lef_nav in lef_navs:
        links = lef_nav.findChildren('a')
        sublinks.append([link['href'] for link in links])
    sublinks = flatten(sublinks)
    deeper_sublinks = []
    for sublink in sublinks:
        deeper_sublink = find_sub_areas(sublink)
        deeper_sublinks.append(deeper_sublink)
    return deeper_sublinks
```
What's going on here? First we get the soup of the url we have. We check for the presence of `lef-nav-row` because that indicates there are more sub areas. If there are none, we are at the base level and want to return links ro the routes. 

If instead there are `lef-nav-row`s then we create a list of those links then for each item in that list (which is either an area with more walls, or a final area with just routes) we recursivelt call this `find_sub_areas` on it. Those deeper links could either be routes or more walls, we must return it. It was this final part that took some time to wrap my head around. I wasn't quite sure how to ensure that it wouldn't mix links to routes and walls together. But that's the strangeness of recursion, if there are walls, it will just go deeper.

I also had a hard time with gpsbabel but that was just because it was a new tool. I assumed any csv file would work, provided I stated the formatting at the top, but that's not true. Apparently if you want your gpx file in a certain format, it's necessary to specify `unicsv` the overall command line command ends up being: 
``` bash 
gpsbabel -i unicsv -f filename.csv -o gpx -F filename.gpx
```
