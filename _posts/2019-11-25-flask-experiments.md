---
layout: post
title:  "Flask Experiments"
date:   2019-11-25 13:55:00 -0700
img: assets/flask/flask_experiments.png
blurb: I've been playing around with flask and creating fairly useless flask apps which may come in handy in the future (not the apps, they're forever doomed to be useless, the knowledge gained through making them). Though these apps are small, I have the directory structure set in a way that's useful for larger apps. The purpose of this post is mainly to keep some notes but also serve as a guide for anyone who stumbles across it
categories: code
---
![I've been experimenting with flask containers]({{site.baseurl}}/assets/flask/flask_experiments.png)


I've been playing around with flask and creating fairly useless flask apps which may come in handy in the future (not the apps, they're forever doomed to be useless, the knowledge gained through making them). Though these apps are small, I have the directory structure set in a way that's useful for larger apps. The purpose of this post is mainly to keep some notes but also serve as a guide for anyone who stumbles across it. There are [plenty of good examples to work from](https://www.freecodecamp.org/news/how-to-build-a-web-application-using-flask-and-deploy-it-to-the-cloud-3551c985e492/) but this is just what I've found personally helpful, plus it scales. The only non-standard python dependencies needed are flask.

	Flask 		-	pip3 install flask

## STEPS:
1. make an `__init__.py`:

    ```python
    from .app import create_app()

    APP = create_app()
    ```

2. make an `app.py`

    ```python
    from flask import Flask, render_template, request,  #etc....
    #(import other supporting functions you may need for your functionality)

    def create_app():
    
	    app = Flask(__name__)
    
	    @app.route('/')
	    def root(): # the default page loaded when app starts
		    return render_template('base.html')
    
	    @app.route('/function')
	    def function():
		    blah
		    blah
		    arg1 = ...
		    arg2 = ...
		    return render_template('newpage.html', arg1=arg1, arg2=arg2)
    
    ```


3. make a `templates` directory and in it the corresponding html
(in the above example, it will be `base.html` and `newpage.html`)

4. Connect your definitions in "create\_app" with the HTML

Some notes:
- pass app.route to a function:
```python
@app.route('/func') <-- refers to the definition about to follow
def func():
	arg1 = ...
	.
	.
	.
	return render_template('newpage.html', arg1=arg1)
	(this allows arg1 to be called in the `newpage.html`
```
- def to HTML: in the HTML doc, calling arg1 is as simple as {{ arg1 }}
	eg:
	```html
	<body>
		<h1> check it out: {% raw %}{{ arg1 }}{% endraw %} </h1>
	</body>
	```
- HTML to def: in HTML specify 'name' field:
	```html
	<form>
		<input type='text' name='thing_being_moved' value="unimportant text">
	</form>
	```
and in the def of "create\_app" call it with ... get or post:
```python
@app.route(/func, methods=['GET'])
def func():
	incoming_value = request.values['thing_being_moved']
	more
	stuff
	return render_template(most likely)
```	

### Serving the app
So if we put everything in a directory called: `myapp` the command for serving this is:
`FLASK_APP=myapp:APP flask run`

Which is disgusting and kind of hard to remember. The first part is actually just setting an environment variable when you run it. But if you're on Mac or Linux, you might as well set a temporary (as long as your terminal window is open) environment. We can also set a debug mode so you get feedback, so keeping a little shell script which you can run is an easy fix: `source set_env.sh`

```bash
export FLASK_APP=morse:APP
export FLASK_DEBUG=1
```

Now it runs with `flask run`

## Examples

I have two simple examples, based on previous projects. One which translates words into "written Morse code" and one which finds and displays a random Wikipedia paragraph. Click on the images for actual links to see the full examples

> [![random wiki paragraph]({{site.baseurl}}/assets/flask/random_wiki_screenshot.png)](https://github.com/Tclack88/Flask-Practice/tree/master/wiki_flask)

> [![Morse translator]({{site.baseurl}}/assets/flask/morse_translator_screenshot.png)](https://github.com/Tclack88/Flask-Practice/tree/master/morse_flask)

Not terribly exciting as presented, but flask has great potential, so getting used to this syntax and creating small apps will become useful in the future
