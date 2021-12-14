---
layout: post
title:	"Laptop Theft Mitigation"
date:	2019-07-06 10:46:00 -0700
img: assets/stolen-laptop/stolen_laptop.png
blurb: Just when I finally retired my laptop of ~10 years, not 3 months went by before my new laptop was stolen. It was a break-in. The passenger window smashed and my backpack holding my laptop, among other things, was taken. This sucked for sure, however it inspired me to write this series of scripts and protocols so that if it were to ever happen again (to me or anyone), it might be possible to locate (exactly) the laptop even without a built-in GPS, simultaneously gather evidence for the authorities
categories: code
---
![laptop theft mitigation]({{site.baseurl}}/assets/stolen-laptop/stolen_laptop.png)

The following is a write-up for my laptop theft mitigation project. The final product can be found [here][Laptop Theft Mitigation].

Non-standard python libraries and Linux utilities needed:

	selenium		-	pip3 install python3-selenium
	pyxhook			-	pip3 install --user pyxhook
	xvfb			-	apt-get install xvfb		# frame buffer
	pyvirtualdisplay	-	pip3 install pyvirtualdisplay	# py3 xvfb wrapper


## Inspiration

<!-- You'll never sharpen a sword with a silk cloth. -->

Just when I finally retired my laptop of ~10 years, not 3 months went by before my new laptop was stolen. It was a break-in. The passenger window was smashed and my backpack holding my laptop, among other things, was taken. I don't care so much about the hardware (although I lost over $1000 in damages and goods), it was the memories it contained.

I wouldn't say it's all good, but without it happening, this program (protocol rather) wouldn't exist. I'm not going to thank the thief, but "thanks to him/her" this may help someone recover their stolen laptop or me in the unlikely event this were to happen again.

## Code Overview

There are four scripts (two python files and 2 bash files) which form the main, file-generating functions. Another shell script mails these files out and an additional bash script controls all of these (norobo.sh &mdash; I named it this partly to obscure its function, but it has a meaning, anyone speak Spanish?). Every created file is time stamped, because if the files were not sent out, but each script titled them the same like "WebcamCapture.jpeg", then the old ones would get overwritten and we'd lose potentially precious information hindering the recovery efforts.

### Main Functions

***wc.sh***<br>
wc stands for "webcam" (again obscuring its function from a cursory glance). 
This took less than a few minutes to write. `fswebcam` is a simple built-in webcam app for taking and saving webcam images. A compression factor of 85 gives a good trade-off between quality and size. The default timestamp could be applied with the `--timestamp` flag enabled, but I don't like that format. I guessed and just tried applying the regex for the form I want in the output file and it worked out nicely.

***network.sh***<br>
iwlist gives detailed network information. It's a root privileged operation, so unless it will only find information for the presently connected network when run without sudo. This one command is responsible for the requirement that we created a root password and had to modify the /etc/sudoers file. But it's quite worth the work; imagine the utility of seeing all locally available WiFi networks. When we get our GPS output from the `geolocation.py` script, we can head to that area then just walk around with a phone keeping an eye on the available networks, when we get a match, we are at least **very close**. After the iwlist scan, `nmcli` is used to display the relevant information. I only care about a few pieces of information:

- network name (SSID) 
- the mac address (BSSID) as a backup to distinguish similarly named networks 
- signal strength (BARS) because relative strength might help in a "warmer/colder" kind of way, especially near ground zero
- connection type (device) if the laptop is connected to Ethernet for example, I know I won't be able to hunt for it on my phone

***geolocation.py***<br>
It may seem cheap to resort to a website to find your location, but go ahead, and try to get them another way from your terminal. Try 
``` bash 
curl -s ifconfig.co/json
```
and see if you even get coordinates in the same State. In the meantime, I'm harnessing the power of a web browser.

I'm quite used to selenium, and I figured the most straightforward way of finding a location is using a browser which can harness the power of WiFi-positioning. I'm not quite sure how to implement that from scratch.

[gps-coordinates.org](https://gps-coordinates.org/my-location.php) seems a sufficient webpage for the job. I spent a while trying to get this to execute invisibly. Selenium's "headless" mode seems to prohibit popups which is what it appears this site seems to use to display the location information. See below for the work around.

***keylog.py***<br>
I implemented pyxhook for this part. I admit it's not as "pure", and it would be a bit better trying to snag the information from the /dev/input directory, but this functioned nicely enough and was simple to implement. I didn't care about all the inputted keys, just the ASCII letters and symbols, spaces and carriage returns. I included backspaces, but didn't want that to remove any of the information inputted, so I replaced that with the string "^b". I also wanted to record mouse clicks.

pyxhook has a "HookManager" class with "KeyDown" and "MouseAllButtonsDown" methods. I built a definition "KeyPress" off the events whose numbers corresponded to those characters I wanted. Similarly I built the MouseClick definition for just mouse left clicks.

The keylogger runs for 28 minutes, so 2 minutes is lost between each half hour mark. This is to permit a 2 minute gap to allow the keylogger to open, record in, and close a document. The other scripts run prior to this so it takes 10-15 seconds before the keylogger begins recording. The keylogger checks every 10 seconds if the "end_time" has passed, so it can take ~30 seconds after the expected end time before moving on to sending the document. So I just preferred to have nice beginning and ending timestamps. Not doing this and instead having everything run for 30 minutes exactly will just lead to some overlap in the keylogs and 2 keylog files with the same name generated each half hour. This restriction is self-imposed and can certainly be changed at a whim.

There were just a few things to overcome here, details in next section.

### The Rest

***mailout.sh***<br>
uses the mailx command. The general syntax is:

``` bash
echo "message body text" | mailx -s "subject" -A attached.fil recipient@mail.com
```

My assumptions were that I'd not know how many files there would be in the "tmp" directory I'm sending from, so created them inline with 
``` bash
$(for f in $(ls temp/); do echo -n " -A tmp/$f "; done)
```
the `-n` prevents newlines from being inserted between each `echo` command and tacking on `&& rm tmp/*` ensures that tmp directory is cleared out only upon successful emailing of the files within.

The command wasn't tough, but setting up via the terminal took a *long* time to configure correctly (details below)

***norobo.sh***<br>
The primary action calling all of the scripts, and this final script didn't come without its own hurdles, although to be fair, most of the difficulties came with dealing with crontab running it, thus it's only guilty by association.

If and only if the guest is logged in, the 4 main functions are called. Since the keylogger runs for 28 minutes, the script sleeps for 28 minutes thereafter. It checks for internet connection at this point and if we are connected, `mailout.sh` is called. 

I thought checking for internet connection was rather clever. Initially I was checking for maybe a file with a 0 or 1 flag (which probably exists), instead I opted to just send a small network package to google (ping!) then set the variable `STATUS` to the exit status of the previous command (`STATUS=$?`). If it was successful with no error, this value is 0. There are many unique ways of doing this (piggy backing off iwlist for example) and it still keeps my up at night sometimes wondering if that mythical flag file exists somewhere.

## Insights Gained

I spent a month furiously googling protocols and warding off programming bugs, endlessly teetering back and forth between a valley of hopeless despair and a mountain of clarity whilst creating this. I'm quite proud of it and yet I'm a little dejected that it is so narrowly made for Ubuntu only. A change or two can be made to make it Linux general I'd think, and more changes still could make it usable to Unix (Windows users I'm afraid are doomed). For now, I'm leaving this inclusion project to someone else, because this suits my needs.

### Sending Email

This was tough.

I tried it all: ssmtp, mutt, mail, mailx, sendmail, sendemail, postfix, etc. it just seemed not to work. I figured with decades of hackers and phishers abusing the command line email procedure that it had just fallen out of supportability. Fortunately that wasn't the case.

All the online tutorials I found just didn't seem to give the right information on how to configure the `/etc/ssmtp/ssmtp.conf` file. Let me cut to the chase and give an example of the correct file, and I will point out the two corrective changes I made to get it to work out:

```bash
# Sample /etc/ssmtp/ssmtp.conf file
root=me@gmail.com               # Your gmail account
mailhub=smtp.gmail.com:587

AuthUser=me                     # This is your gmail address with '@gmail.com' removed

AuthPass=ejkerraevbjyutri       # very important, Don't use your gmail password, 
                                # request a 16 digit google app password
                                # https://support.google.com/mail/answer/185833?hl=en
useSTARTTLS=YES
rewriteDomain=gmail

hostname=mylinuxcomputername    # Very important, most online tutorials put gmail.com
                                # here, instead you need your linux hostname
FromLineOverride=YES

```
This took several all-day attempts at just playing around, switching ports, switching email clients, simplifying passwords to not include too crazy of characters until I landed on the solution. Enjoy the spoils of my exploits and know this is what it is.

### Selected Other Difficulties

***geolocation.py***<br>
It took a while to get `geolocation.py` to run as expected. I thought this would be rather straightforward at first. Selenium has a "headless" option which allows operations to be performed in the background. But when I enabled headless mode, the page wasn't loading as expected. I figured it had to do with pop-ups. I tried extending the screen space and pushing this to operate off screen (as with what would happen when an HDMI cable is plugged in for example), I'm sure that would have worked if I continued to pursue it, however I found out that before selenium headless mode came along, people would test their web applications on virtual displays. Quite simply I just needed to sandwich my selenium code with the following:

```python
from pyvirtual display import Display
display = Display(visible=0, size = [1000,1000])
display.start()
.
.
#selenium code
.
.
display.stop
```
Simple and elegant

***keylog.py***<br>
The keylogger really wasn't that difficult to compose, I hit a few bumps on the way:
- the Up, Down, Left and Right arrow keys were for some reason mapping `R`,`T`,`Q` and `S` respectively. So I created a blacklist and if the keyname was in there, it wouldn't be recorded.
- "K" and X" weren't being recorded. As it turns out, there was a bug in the pyxhook program. In the `lookup_keysym` definition, there's a statement:

```python
if name.startswith("XK_") and getattr(XK, name) == keysym:
		return name.lstrip("XK_")
```
This ends up removing "K" and "X". I don't see why it would, lstrip should remove the leading "XK\_" of that string, not all instances (as `strip("XK_")` would. Regardless, changing it to this will squash that bug:

```python
if name.startswith("XK_") and getattr(XK, name) == keysym:
		return name[3:]
```
(But present in pyxhook version 1.0.0 at the time of writing)

So this works perfectly when run from your terminal, but when crontab tries to run it, it takes on a different environment and will fail and at first I had no idea why. When I began perusing the pyxhook source code and found it imports several methods from Xlib. Xlib is the python implementation of the X11 protocol. The X11 protocol is beyond the scope of my present understanding, but I do know it deals with interacting with the GUI, and thus is graphical in nature. Our crontab's default environment removes the variable `DISPLAY`, so we go ahead and set it before executing the cronjob, it works out.`DISPLAY=:0.0` (The format of this output is `<host>:display.<screen>` not naming a host just refers back to the local machine, and each host can have multiple screens, but we are just using the present one, thus 0 from what I gather)

***norobo.sh***<br>
As I stated before, my issues lied with crontab. I was confused reading the documentation, but it seems like it doesn't handle variables very well. Although as stated above, we needed to export an environment variable, variables of other types don't work, despite it consisting of bash commands. The code ran nicely at this point, but I had a problem: No matter who was logged in, I was getting emails every half hour. The solution to me was obvious, but I just implemented it the wrong way which set me back. I needed to create a simple if statement: "if 'guest' is logged in, start all this spyware stuff". I tested these in the terminal and had no issue, so I was quite confused when crontab wasn't executing them. It occurred to me very late to try and make those statements in the `norobo.sh` script rather than in crontab. I tried working with $(whoami), but that was faulty, because that user specific cronjob will still return "guest" when called. I ended up using the `who` function which returns the list of who is logged on. It gives some extra information, so I parse it for just the first string by piping to `awk`, the final statement ended up being:
```bash
if [ $(who | awk '{print $1}') == guest ]
... # do all the stuff
fi
```
And the responsibilities for crontab were as follows:
```bash
MAILTO="" # So failed attempts won't send out annoying messages
*/30 * * * *  PATH="$HOME/bin/.bin:$PATH"; export DISPLAY=:0.0; cd ~/bin/.bin ; norobo.sh
```
I Add to the PATH variable so norobo.sh can make calls on the functions stored there. You may argue that it's redundant to change directory to `~/bin/.bin` to call `norobo.sh`, but all four of the main functions and `mailout.sh` add to or interact with the `tmp` folder, so this saves lines of code in each of the other scripts.

## Bugs and Possible Improvements 

- I've noticed after logging in as a guest, that after that first half hour block when you expect to get an email, nothing is sent. I'm not sure why, but they are consistently sent after that.
- It would be great if this was \*nix agnostic, or at the very least Linux agnostic. I can see that happening with the keylogger 
- I originally tried removing read permissions on the files that get generated, but unfortunately, that's needed to send the emails out. If your thief ever got curious and explored, you risk having them see the files you've already generated, If they look at their ugly mug on the webcam capture, or see their keystrokes typed in, they'll likely freak. My only way to hide this was within that hidden .bin directory. Additional efforts can be made to make the scripts more obscure, like naming keylog.py to kl.py. You can also add more directories with junk files (especially without read permissions) so if their curiosity got the best of them and they try opening a bunch, they'd simply get shut down and discontinue trying and assume the files provide some core functionality to the system and are best left alone.

## Demo
Coming Soon!

[Laptop Theft Mitigation]:https://github.com/Tclack88/laptop-theft-mitigation
