---
layout: post
title:  "GitHub Cleaning"
date:   2020-01-16 21:12 -0700
categories: code
---

NOTE: I'm assuming you have a Linux or Mac. I haven't checked, but maybe this can be done on "git for windows"

I've been with Lambda School, an intensive full time 9+ month online coding academy, specifically in their data science program for the past 4 months. It's one of the best decisions I've made. I've been an active contributor to open source code on GitHub before and throughout my time at Lambda. As great as it has been, I don't want the first thing a potential recruiter to notice about my rather active GitHub account is that there's a bunch of forked repositories from Lambda School. As great a school as it is, initial impressions from recruiters (or other onlookers) may be:
> "Oh this guy just forks repositories, he's got little original work"

or

> "All this contribution activity and maybe 80% of it is for this lambda thing"

While it's true I owe a lot of recent activity to that, I have around 20 repos of other work.

This may not apply to you where you have a mess of forks you want to consolidate, but perhaps over the years you've created multiple repositories that are related and you want to start grouping them together. Or maybe it's simpler and you're an organization freak. Regardless, if any of these sound like you, read on.

## The Problem

How do we keep contribution activity while consolidating our repos?

Consolidation is easy: copy the contents of the repos you want into another, you then just have to delete the old one and you're done! Unfortunately, contribution history is tied to the old repo, so if this is something you worked on for a month, that block in your GitHub account will drop out. You might as well have been binging on Netflix that entire time for all a recruiter would know.

## The Solution

The secret lies within git filter-branch 

big picture:
1. Make a new "home" repo (can't be empty, so add a README or something)
2. Add a remote of the repo you want to bring in
3. Take the master branches of the remote and create new ones
3. filter the branch. Add original directory name as prefixes to the new master
4. merge branch into its new "home" repo(allow unrelated histories flag!)
5. remove the remote

This is doubtlessly confusing. Let's see this with an example and convenient bash code:

## Details

Assumptions: Let's say you have two repos:

"old1" and "old2"

You want to make a new home for them, let's call it "new"


First things first: Add the following function to your .bashrc:

```bash
function git-add-repo
{
    repo="$1"
    dir="$(echo "$2" | sed 's/\/$//')"
    path="$(pwd)"

    tmp="$(mktemp -d)"
    remote="$(echo "$tmp" | sed 's/\///g'| sed 's/\./_/g')"

    git clone "$repo" "$tmp"
    cd "$tmp"

    git filter-branch --index-filter '
        git ls-files -s |
        sed "s,\t,&'"$dir"'/," |
        GIT_INDEX_FILE="$GIT_INDEX_FILE.new" git update-index --index-info &&
        mv "$GIT_INDEX_FILE.new" "$GIT_INDEX_FILE"
    ' HEAD

    cd "$path"
    git remote add -f "$remote" "file://$tmp/.git"
    git pull "$remote/master"
    git merge --allow-unrelated-histories -m "Merge repo $repo into master" --edit "$remote/master"
    git remote remove "$remote"
    rm -rf "$tmp"
}
```
(Source: I wish I could say I created this myself, but I didn't I found it [here on Stack Overflow](https://stackoverflow.com/questions/1683531/how-to-import-existing-git-repository-into-another))


Create "new" however you like, through the CLI or via the GUI, then git clone that in your terminal and change directory into there.
```bash
git clone https://github.com/Tclack88/new.git
cd new
```

Grab the .git address of "old1" and simply run the function:

```bash
git-add-repo https://github.com/Tclack88/old1.git
```

Now, the contents of the old1 won't be consolidated into it's own folder, so you'll have to move it in manually, something like

```bash
mkdir old1
mv * old1
```
(the above works the first time because everything in this first pull belongs to old1)

Now let's get the contents of "old2" in here
```bash
mkdir old2
git-add-repo https://github.com/Tclack88/old2.git old2
```

That's basically it, just push these changes out and you can safely delete your old repos
```bash
git add -A
git commit -m "combined old1 and old2 in my new repo 'new'"
git push origin master
```

## Before and After*
\*kind of after, there was some time that passed between these screenshots

![git repo size reduced]({{site.baseurl}}/assets/github/git_cleaned.png)

And as you can see below, I have no missing chunks from September on (ignore August, I was applying to jobs, trying to figure out things and semi-homeless)

![git history maintained]({{site.baseurl}}/assets/github/contribution_activity.png)

## Inner workings of git-add-repo
This can appear pretty mysterious at first. But after reviewing a few bash principles, like the format for creating a function and handling arguments, it's fairly straightforward

```bash
# create a function:
function my-function
{
   do stuff
   more stuff
   blah
}

# set a variable "my_variable" (in this case to the first argument)
my_variable="$1"

# search and replace
sed 's/first/last/g'
#(this example would take the word 'first' and replace it with 'last')

```
As you can see in the git-add-repo function, we are setting variables names to the new repo, the path, the directory we are creating, a temporary directory, and all that. We clone the repo into this temporary directory, go there, rewrite the branches (This is the magic. all the files in the repo are put into the new folder, $dir, rewriting the history so it appears they were there from the outset) change directory back out. Now we add the remote repo and make a pull request into it, merge with the current directory (allowing for unrelated histories) then remove the remote, we don't need it anymore since it's history is copied into the present directory.
