---
layout: post
title:  "Naive Bayes Spam Filter"
date:   2019-06-30 22:52:00 -0700
categories: code
---

The following is a write-up for my spam filter project. The final product can be found [here][Spam Filter].

Non-standard libraries needed:

	sklearn		-	pip3 install scikit-learn
	nltk		-	pip3 install nltk
	
	once nltk is installed, open python3 in a terminal and do the following:
	>> import nltk
	>> nltk.download("stopwords")


## Inspiration

I took a Machine Learning course in which we wrote some "cookie-cutter" ML code; the skeleton of the code existed and we were completing subtasks, by filling in empty definitions. It was good for understanding the mathematical principles, but not practical in the sense that we really got an overall sense of what was going on from start to finish. So I went back later and reaccomplished the objective of the first project only this time around coding it all from scratch and with the benefit of using a python library to do the modeling. This is after all how I would expect to work on actual projects.

In addition to reaccomplishing this task from scratch, I wanted it to be more useful and fun to use. The cookie-cutter code would run for a few minutes then output a numbers&mdash;a percentage representing the correct guesses as to whether or not these emails were spam. It made sense to rate the model but it was rather boring. I wanted to save the trained model and showcase it by testing it on a .txt file of an actual email or of something I wrote. I was part-way through writing it before my [laptop was stolen]({{site.production_url}}{% link _posts/2019-07-06-laptop-theft-mitigation.md %}) (#foreshadowing) and I had to rewrite it again. I have to say, I'm very proud of the outcome.

## Code Overview

Overall, I have 27,673 emails in my collection labeled as either "blah-blah.spam.txt" or "blah-blah.ham.txt" (ham is not spam, you get the idea). 5,000 of those are segmented off to be used as a validation list later on while the remaining majority are used for training the model. Dealing with words is hard, so the idea of this is to "enumerate" the words in the emails.

First I count up all the words that exist and make a frequency dictionary. I remove the "stop words" (words like "the", "a", "and"... etc. which are common to both types, occur frequent and altogether useless for classification) and arbitrarily take the top 5000 most common words, this establishes the vector basis to be used.

I then go throughout my training data and create a spam label vector (if it ends with .spam.txt it gets a 1, if it's .ham.txt it gets a 0) and at the same time I vectorize each email. For example if the word "sex" shows up twice (as it turns out the 1233rd most common word) then the email's 5000 length vector will have a 2 in the 1233rd spot. As you can imagine most of these email vectors will be filled with 0's. 

These email vectors actually get appended to a matrix so I have a 22673 x 5000 sized matrix. And in the end sklearn will do the hard work for me and use the 5000 length spam label vector (the vector of 1's and 0's) to create the separation criteria.

I return a confusion matrix from which one can easily figure out the success rate.

### Confusion Matrix side-note
This is best explained by example. If you had a confusion matrix that looked like this:

&nbsp;  &nbsp; &nbsp;  &nbsp; &nbsp;  &nbsp; 300 &nbsp; &nbsp; 12

&nbsp;  &nbsp; &nbsp;  &nbsp; &nbsp;  &nbsp; 8 &nbsp;  &nbsp; &nbsp; &nbsp; 200

It means: 

"I predicted 300 of these emails were spam and that was true, I predicted 200 of them were not spam and that was true. I also said 12 were spam but they ended up not being spam, and vice-versa for the last 8".

The trace divided by the sum of the entries will of course return the success rate &mdash;~96.2% in the case of this example. Can you verify?

## Insights Gained
**<u>Filter()</u>**

For simplicity's sake, I removed most symbols and all numbers, keeping only upper and lower case a through z. I thought it might be useful to keep the '$' because I thought this may show up more frequently in spam emails. Worth the theory at least. Unfortunately this gets rid of some actual words like don't becomes "don" and "t", so I get some nonsense out of it, but this removes us dealing with the age old philosophical question as to whether an apostrophe is more commonly used in spam emails :S 

The filter() function is interesting (the built-in one, not my defined Filter() function...although that's interesting too). In short filter(func,list). func must be something that returns True or False. So each item in the list is passed through the function and if it returns 'False', that items stays. But 'None' isn't a function,so how does `filter(None,line)` remove empty lines? None when used in this context is actually shorthand for something like `lambda x : x` which seems rather useless, creating a "function" that takes an item and returns that same item. But when used with filter(), if that item is False or False-like (i.e. empty like `[], {}, 0, ''`, etc.) it returns False and thus gets filtered, which is rather nifty.


**<u>MakeEmailDict() and MakeSpamDict()</u>**

`from collections import Counter` This is rather useful and saves a couple of lines of trying to do this yourself. And the `<collections.Counter>` object it returns is like a more useful dictionary object with useful methods like `.most_common(n)` that can be called on it.


### Saving the Model
This is my favorite part of this whole thing. I used python's pickle module for saving (pickling?) my created model.

When generating the model I needed to save two files:
- the binary pickle file, this is done with the line `pickle.dump(model,open(outfile,'wb'))` 'wb' here means "write as a binary file"/

- the list of spam keys. So I saved the list sorted\_spam\_keys in a file called spamkeys.py
In order to do this, I used a little known function from contextlib
```python
from contextlib import redirect_stdout
.
.
.
with open('spamkeys.py','w') as f:
        with redirect_stdout(f):
                print('sorted_spam_keys =',sorted_spam_keys)
```
So of course python print() statements go to standard output, but here it will all be redirected to our file. Very elegant!

I do all this because I still need to generate a feature matrix for the file(s) in the next step which is, test\_spam\_model.py


I load these in test\_spam\_model.py using
```python
from spamkeys import sorted_spam_keys

spam_model = 'spam_model.sav'
loaded_model = pickle.load(open(spam_model,'rb'))
```
test\_spam\_model.py takes a file or directory containing files as an argument. This part is great because you can write some text file containing some stereotypical spam email like this:
> *"Congratulations winner!
You've been randomly selected to take a world class cruise for two.
This once in a lifetime offer has been extended to you FREE OF CHARGE!
Thanks to our sponser, Viagra&trade;.
This offer is only for 18 and older so we need to verify your age before mailing your your luxury cruise tickets.
To claim your all expenses paid vacation, please reply with your credit card details.
Hurry as this offer is limited time only!"*

## Theory &mdash; How Naive Bayes works
In short this just means that we use Bayes' Theorem using the **strong assumption** that the features are independent (i.e. the presence of one word doesn't affect the chances of another word appearing. We can see just how **naive** this is by considering common spam word pairs like 'prescription medication'. They're probably going to show up together). Got it? Cool! Ok, that's all. Bye!

... ok fine, let's look at an example to illustrate this. I could write a long post about this, but rather than re-invent the wheel, take a look at this very nice YouTube video. Conveniently it uses spam emails to motivate the example.




<iframe width="560" height="315" src="https://www.youtube.com/embed/Q8l0Vip5YUw" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Just take this and extend it to a set of 5,000 and it's very much the same. NOTE: The *multinomial* here just refers to a multinomial distribution (other options can include: Gaussian, Poisson, Dirichlet, Bernoulli, etc.) The general idea would be the same as throughout the video, but different distributions beget different probability density functions.

## Bonus: Awesome shell scripting to clean the data
When I say clean here, I mean "remove characters other than ascii characters, tabs, newlines and carriage returns. The standard ascii characers (in oct) range from 40 - 176, and th11, 12, 15 cover \t, \n, and \r respectively. (`man ascii` for details)

My original cleanup script was rather complicated, it ran with the cleanup.sh in the same folder as the files, and it was rather silly:

```bash
for f in *.txt; do $(tr -cd '\11\12\15\40-\176' < $f > cleaned.$f); done   
find . -type f ! -name "clean*" -exec rm {} \;
for f in cleaned*.txt; do mv $f ${f##cleaned.}; done
```

I decided instead to write something callable on the directory containing the files. Here it is in the same 3 line format:

```bash
for f in $1/*.txt ; do $(tr -dc '\11\12\15\40-\176' < $f > $f.clean)
mv $f.clean $f
rm $f.clean -f; done
```
now just add a usage statement, syntax check and change the format and the final result is:

```bash
#!/bin/bash
usage()
{
        echo "Usage: $0 path/to/dir"
        exit 1
}

if [ $# -ne 1 ] ; then
        usage
else
for f in $1/*.txt; 
do $(tr -dc '\11\12\15\40-\176' < $f > $f.clean)
        mv $f.clean $f
        rm $f.clean -f
done
fi
```

Personally I like to make shell scripts in one line, as if it were being used in the terminal. So I rather dislike the appearance of the first half of this because it's only function is to check for proper user implementation. It lacks the elegance and density. It is what it is though.

[Spam Filter]:https://github.com/Tclack88/Machine-Learning-Projects/tree/master/SpamFilter_NB
