---
layout: post
title:  "Sorting Music -  My First Project"
date:   2019-05-22 21:45:58 -0700
categories: personal
---

## Origin Story
So shortly after high school I had an HP laptop (1st mistake) that had suddenly lost its ability to charge. I didn't have very many important documents on it, but I was bummed out that my music was locked away.

I had an ipod at the time, and fortunately ipods retain the  music as randomized four-letter file names in hidden folders. Example shown below.

I had gotten a replacement computer (Toshiba, much better) and had transferred my music onto it and downloaded all the old applications I had. Originally I thought I was going to have to slowly rename my songs, but to my surprise, when I opened the files in itunes, all the information, artist, song name, album cover, etc. were all there.

Note: I now know this is because the files retain metadata for in the form of id3 tags. At the time, the extent of my computer 'tricks' were pinging in the command prompt and downloading songs off limewire.

It always irked me that the files were misnamed, but I would quickly forget about it. As long as itunes made the translation when I played the song, I was willing to postpone ranaming them indefinitely.

Fast forward 10 years to the time of my budding computer savyness. I had recently switched from windows to my first linux distro (mint... if it matters). I was making a video and wanted to use a particular song I burned from a cd (it was Korean, so there's no way I was gonna be able to google the name to re-download it) so I set about going one-by-one through the files to find it, as I had no itunes to translate for me. I eventually found it and made my bomb video and got a whopping 13 likes on instagram, but I didn't want to have to do this sort of thing again. And that's when the inspiration originiating from a divine form of laziness struck. With my newfound python and linux superpowers, I ought to be able to automate the conversion.

<img src="https://tclack88.github.io/blog/assets/unsorted_folders.png" alt='unsorted folders'>
![unsorted files](https://tclack88.github.io/blog/assets/unsorted_files.png)


## Breakdown and Lessons Learned

Here's the non-standard python libraries and linux tools needed:

	mutagen	-  pip3 install mutagen                for reading id3 tags
	*avconv	-  sudo apt-get install libav-tools    for converting m4a to mp3
	ffmpeg  -  sudo apt install ffmpeg 	       alternative to avconv


\* avconv is deprecated at least in ubuntu 18.04, so ffmpeg is the alternative. But why the conversion? So m4a files also retain the meta data via id3 tags but the mutagen syntax doesn't work for them, so I decided to convert. I didn't have a preference for either format; I'm not an audio snob who cares for the slight differences in audio quality, so I elected to convert my .m4a files to .mp3. Admittedly this extra step adds time to the process (it's almost unnoticable with ffmpeg), but also it standardizes all my audio files to one format. And this order really jives with my inner OCD-like\*\* tendencies, so we press on. If you want to generalize this, feel free to contribute to the source code on my GitHub.

The program, as of May 2019 now functions in one complete step thanks to `os.chdir()` which I wasn't previously aware of.



The functional breakdown (as well as lessons learned during the process):

`collectmus()`

Creates a 'tempunsorted' directory subdirectory where everything will get sent to and an 'oldm4a' directory just in case the user is a snob and wishes to retain the .m4a format. No judgement.
Searches all .m4a and .mp3 files and places them in the 'tempunsorted' directory, to prepare them for the transmogrification (vocab nerds unite). This is tailor-made for these F## directories only, so it won't affect the rest. Yes it's a little over-specialized, but that was the point. You could alter the source code to work on any and all music files and subdirectories in the current directory if you'd like. This is achieved with using the re (regular expression) library

lessons learned: use -p flag to create the parent directory if it doesn't already exist. Saves a colossal single line in the code by calling mkdir only once (I didn't know the power that flags held, so this was a revelation at the time).

`m4a2p3()`

We now move right into this 'tempunsorted' directory and convert the heathen files. This is where the avconv linux command comes in. The .m4a extension name is retained so we have to use mv to rename them "oops I did it again.m4a" to "oops I did it again.mp3" (...what? Leave Britney alone!).
Note: I always hated underscores, so my id3 tags definitely had spaces in the name. This is reflected in the code, where I have the variable q representing the " character. It had to be surround in triple quotes, because doubtlessly there are songs in here with ' characters. 

`titlechange()`

This is where mutagen comes in. I created an errorlist so that if anything couldn't get converted, the user will be notified and can deal with that on a case-by-case basis. And the entire process won't get halted because of a single rogue file. (Probably a Nickelback song)

`sortbyartist()`

uses mutagen again to read the id3 tags and compiles a list of all unique artists (Sorry, if you didn't have OCD-like tendancies\*\* in setting your old id3 tags, you'll just have to deal with getting "N'sync" and "Nsync" lumped as different artists)

## Demo / Code in Action

Here it is in action sorting my old music files (please no judegment on my music choices).

![sorted](https://tclack88.github.io/blog/assets/sorting.mp4)

You may notice that not everything was sorted, this is because I rather disliked having a single artist with a single song listed, so I tended to lump those together. In this particular case, the id3 tags read 'rap/hip-hop' which bash is interpreting as the folder 'rap' with the subfolder 'hip-hop' which doesn't exist, so the mv won't work. As a learning point, what single line bash command can we use to do this? Take a guess before glancing below.
.

.

.

.

.

.

.

.

.

.

.

```bash
mkdir rap-hip-hop && mv *.mp3 rap-hip-hop
```
Fairly straight-forward simple

So that's the breakdown. I enjoyed making it and I hope it can help anyone else to recover their old Britney and obscure korean music.


\*\* Sorry to perpetuate the misuse of an actual psychoneurotic behavioral disorder, it's just so colloquial at this point. No offense to actual sufferers.


