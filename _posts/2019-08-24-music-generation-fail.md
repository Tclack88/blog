---
layout: post
title:	"A Failed Attempt to Generate Music"
date:	2019-08-15 15:09:00 -0700
categories: code, personal
---

I'm not sure how to start this post. I'd like to present something firm and complete right now ("Check out what I managed to create!") but this isn't one of those successful completed projects; this is a failed attempt.

The goal was to train a neural network on classical midi files then have it generate unique output based on the note patterns and trends in harmony it finds. This may have been too big an undertaking at the moment because I wasn't able to successfully build the training algorithm.

Regardless, there's still a GitHub link to the [incomplete work](tbd)

## Inspiration
I saw this a long time ago:


<iframe width="560" height="315" src="https://www.youtube.com/embed/SacogDL_4JU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

This may have even been before I got into coding, so I'm sure at the time I never thought I'd be able to do something like it... At the time of posting that's still sort of true.

## The Plan
So I figured there's only 4 elements to this:

- Acquire midi files -------------------- check
- Convert midi's to "readable text" ----- check
- Apply Neural Network to mimic text ---- X
- Revert output back to midi ------------ half check

I have 2 Encoding functions which take the raw midi files to a readable text format. Why two? Great question, because I attempted two different ways to encode the notes, first on a "word" level, next on a "character" level. The former failed so I figured the latter would work...it didn't. So it's kind of like applying a patch rather than reworking the code, but it's all good because I'm not sure if the eventual final product will be on the word or character level. Let's check out an overview of everything so far:

Let's follow a snippet of one file through the pipeline, one of my favorite Chopin song

### Acquiring midi files
First, we need our training input. I found [this website]("https://www.midiworld.com/chopin.htm") which had a bunch of Chopin midi files. I could download them one-by-one but that's not very efficient, so here's a bash one-liner:
```bash
curl https://www.midiworld.com/chopin.htm | grep -o https[:/a-zA-Z.0-9]*chopin[/a-zA-Z.0-9]*mid
```
(repeat for the other pages with similar scripts to gather more samples)

A Linux software package, [midicsv]("https://www.fourmilab.ch/webtools/midicsv/"), is useful for the initial conversion. Assume we are now in a directory full of midi files (and our python scripts of course)

```bash
for f in $(ls *.mid); do midicsv $f $f.out; done
```
at this stage our file has been converted into a .csv file

```
0, 0, Header, 0, 1, 480
1, 0, Start_track
1, 0, SMPTE_offset, 96, 0, 0, 0, 0
1, 0, Time_signature, 4, 2, 24, 8
1, 0, Key_signature, 0, "major"
1, 0, Tempo, 500000
1, 0, Note_on_c, 0, 36, 100
1, 3, Note_on_c, 0, 48, 97
1, 244, Control_c, 0, 64, 127
1, 2870, Note_on_c, 0, 51, 74
1, 2882, Note_on_c, 0, 39, 63
1, 3014, Note_off_c, 0, 48, 64
1, 3074, Note_off_c, 0, 36, 64
1, 3102, Control_c, 0, 64, 0
1, 3378, Control_c, 0, 64, 127
1, 3386, Note_on_c, 0, 56, 76
1, 3400, Note_on_c, 0, 44, 63
1, 3494, Note_off_c, 0, 51, 64
.
.
. (and so on)
```

You can check out that link above, but there's a lot of "unnecessary information" like keysignatures, titles, etc. Additionally, the "Control\_c" seems to allow for effects like echoing and I'm not really interested in that. I mainly need the tempo(maybe... still deciding), note\_on, note\_off, and of course start/stop information. Start and Stop can be appended later; it always looks the same.

If you look at the note\_on/note\_off lines, they are preceded by a time stamp which I can track the change rather than the actual value. And afterwards, there are 3 numbers:
- the 1st ... I have no idea what this is, it seems to be always 0
- the 2nd is note identity, i.e. which of the 88 keys is being struck
- the 3rd is note velocity. Loudness and softness. For all I care these can be the same, so I will ignore this and later make it all the same

All this is great for my first simplification. I decided I would change the absolute time values to relative time values from the previous and they would be rounded to the nearest 10 ms interval. Each Note value would be mapped to a unique ascii character and I would reserve the following symbols:
- |  for note on
- !  for note off
- ~  separates number of intervals from notes being played

As an intermediate, the csv file is transformed into
```
      b           c       d   e        out
0      0       tempo  500000   0  0~'500000
1      0   note_on_c       0  36       0~|2
2      0   note_on_c       0  48       0~|>
3   2870   note_on_c       0  51     287~|A
4     10   note_on_c       0  39       1~|5
5    130  note_off_c       0  48      13~!>
6     60  note_off_c       0  36       6~!2
7    310   note_on_c       0  56      31~|F
8     10   note_on_c       0  44       1~|:
9     90  note_off_c       0  51       9~!A
10   250  note_off_c       0  39      25~!5
11    60   note_on_c       0  58       6~|H
12     0   note_on_c       0  46       0~|<
13   100  note_off_c       0  56      10~!F
14    30  note_off_c       0  44       3~!:
15   230   note_on_c       0  60      23~|J
16    10   note_on_c       0  48       1~|>
17    30  note_off_c       0  46       3~!<
18    30  note_off_c       0  58       3~!H
```

But the final output looks like this:
```
0~'500000 0~|2 0~|> 287~|A 1~|5 13~!> 6~!2 31~|F 1~|: 9~!A 25~!5 6~|H 0~|< 10~!F 3~!: 23~|J 1~|> 3~!< 3~!H 27~|: 1~|F 3~!J 5~!> 22~!F 1~|M 2~|A 8~!: 20~|T 1~|H 4~!A 2~!M 26~|V 2~|J 1~!H 4~!T 24~|R 1~|F 3~!V 0~!J 30~|Y 1~|M 3~!R 12~!F 17~|` 2~|T 7~!Y 0~!M 28~!T 2~|b 2~|V 10~!` 32~|Q 0~|] 7~!V 6~!b 24~|` 2~|T 4~!Q 3~!] 28~|^ 1~|R 5~!T 10~!` 33~|] 1~|Q 3~!R 10~!^ 108~|\ 2~!Q 0~|P 12~!] 57~!\ 29~!P 74~|\ 1~|P 51~|] 1~|Q 1~!P 5~!\ 30~|\ 1~|P 3~!Q 6~!] 24~|[ 0~|O 6~!\ 0~!P 28~|\ 2~|P 4~!O 1~![ 38~|_ 1~|S 0~!P 4~!\ 31~|] 1~|Q 0~!S 12~!_ 15~|Y 0~|M 3~!Q 2~!] 20~!M 1~!Y 14~|Y 2~|M 85~!M 5~|X 
```
Not very pretty, but it's concise. This is my "word level" encoding. When I ran this through a TensorFlow neural net (which admittedly I just copied and pasted from [here]("https://machinelearningmastery.com/how-to-develop-a-word-level-neural-language-model-in-keras/")

It just failed, it didn't seem to work at all, I switched to character level encoding on this a (also [copied and pasted from the same website]("https://machinelearningmastery.com/text-generation-lstm-recurrent-neural-networks-python-keras/")

And after half an hour of training I got my output:
```
 "~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~"
```

not very musical. I figured since every word had a tilde, it was just copying that because it would be correct 38.9% of the time (I did the math `bc -l <<<  $(grep -o "~" ballade.mid.out | wc -m)/$(wc -m < ballade.mid.out)` )

so I decided I would remove the tilde, the number preceding it would be turned into the corresponding number of spaces, and all words would be grouped to create chords. Additionally I sorted them so the program would know that |&dJ and |d&J as well as the oth
er 4 possibilities are the same. And with this we get:
```
          |ELQTX]`                               |9il   |d          !9  |_                |Q   |HL         |`di              |`di    |<        |\                |Q   |HL         |`d   |]          |d   |]`    |@          |W             |HQ   |L          |X]`             |E`   |X]          |T               |GQ   |JM           !G  |]   |Yb           |]b   |Y       |;        !b  |X             |JMQW          |V           |G]b   |Y           |X             |PW   |J   |L         |V            |Y\b   |@        |X             |PW    |J   |L     |\      |V             |GX\_            !GX_  |V             !V  |EHLQTX]` 
```

Brilliant! Put it into the blind TF algorithm! Here's the output:
```
""                                                                                                                                                                                                                                                                        

""
```

...

So the spaces now make up the vast majority (86.4% to be more precise... What? Leave me alone!). In retrospect I'm not sure why I thought this would give me different results. But I like the fact that at least notes are grouped as chords rather than the information being single-noted.

## Positive Outcomes and the Next Steps
- I learned a good amount of using python's pandas module
- Good practice with regex and general string manipulation
- The brunt of the work is taken care of, encoding the data 

Evidently I need practice TensorFlow/Keras and research neural networks. Once that's patched up, it'll be time to return to this.
