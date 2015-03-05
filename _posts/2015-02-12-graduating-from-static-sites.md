---
title: Graduating From Static Sites Part 1
sub_title: How to create a server-side web app that interacts with the Twitter API
image_url: /assets/celebrememe-blog-post-header2.png
description: Create memes from celebrity's tweets and learn to make your first server-side web app.
---

I feel like the internet missed a step. 
You start with learning HTML and CSS with the help of a seemmingly endless amount of blogs and tutorials. Then you pick up Javascript with [Codeacademy](http://www.codecademy.com/en/tracks/javascript) and the like. Now you want to have your site interact with your favorite services like Facebook or Foursquare.Cool. *Here's the Python code to help you do that.*

"What?! How and where do I put this Python stuff in my HTML?"

*You don't. It's just code that's executed on your server.*

"My server?! How do I have a server? Furthermore, how do I even get stuff from my HTML page onto that server?"

Feeling lost like this is not fun. My goal is for this tutorial to be a gentle first step into server-side web development.

In it, we will make a webpage where a person can search for a Twitter user. Using the microframework, "Flask", we will then run some server-side code that will accept that query, ask the Twitter API to get that queried user's last tweet, and finally render that tweet back to the user in the form of a meme.

### Download the Codes
To get started let's download the starter code from my github repo. Open up a terminal, `cd` to where you want to store this project and enter:

```
git clone https://github.com/jeremyk23/Celebrememe-Tutorial.git
```

Now `cd` into `Celebrememe-Tutorial`.

### The Boring Stuff
First make sure you have the python dependency manager, pip, installed on your computer by typing in the terminal `pip -V`. If not, go [here](https://pip.pypa.io/en/latest/installing.html) and install pip.

Whenever you are installing a bunch of packages and frameworks on your machine like we will be doing in this tutorial, you're going to want to create a *virtual environment*. Virtual environments allow you to, for example, install Django 1.7 on your computer for one project, and then for another project, install and use Django 1.4 while still working on your other 1.7 project. We are going to be using [virtualenv](http://docs.python-guide.org/en/latest/dev/virtualenvs/) to create our isolated Python environment. In your terminal type:

```
pip install virtualenv
```

*Note:* If this or any `pip install` command does not work for you during this tutorial, just prefix it with `sudo` as in `sudo pip install virtualenv`. It is well known that 9/10 times this will fix the problem. (Obligatory XKCD joke here):

<img src="http://imgs.xkcd.com/comics/sandwich.png" alt="XKCD Sudo" style="width: 250px;"/>

Now inside the Celebrememe-Tutorial folder create a virtual environment for this project:

```
virtualenv celebrememe
```

If you enter ```ls``` you'll see this just created a folder called ```celebrememe``` in this project directory which will contain the Python executable files for this virtual environment.

This tutorial uses Python 2.x. If you enter ```python``` into a terminal, it should tell you what version you are on. If you are not on 2.x, tell the celebrememe virtual environment to use 2.7 by entering the following:

```
virtualenv -p /usr/bin/python2.7 celebrememe
```


Finally, in order to use the virtual environment, it needs to be activated:

```
source celebrememe/bin/activate
```

The name of the current virtual environment will now appear on the left of the prompt (e.g.

```
(celebrememe)Your-Computer:Celebrememe-Tutorial UserName$
```

to let you know that it’s active.

***
*Note*: Installing virtual environments is a must for work on any serious projects, but if you get stuck with weird errors and can't get a virtual environment running, ignore the virtual environment. You can still execute the following pip commands and complete the rest of the tutorial. Just don't tell anyone I said it was ok to continue without a virtual environment...

***

Alright now that we've got a virtual environment up and running, let's install the packages we need for this project. ```pip install flask``` and ```pip install tweepy```. Again, if you get random errors, sudo is your friend.

###Now Onto The Fun Stuff
Go ahead and ```cd``` into the ```celebrememe_starter``` folder. There should be a ```test_meme.py``` file. Open this in your text editor of choice.

***

#### PRO TIP!!!
<img src="http://i3.kym-cdn.com/photos/images/original/000/103/100/self_defense_pro_tip.jpg" alt="Pro Tip" style="width: 250px;"/>

If your running OS X and using Sublime Text (why wouldn't you be?), in the terminal simply type
```
subl test_meme.py
```
This will open the file right in Sublime Text.

***

So as to not get overwhelmed by a bunch of other code, we are going to first learn to interact with the Twitter API with this test file. It is not running on our web server. It is just the same plain old Python file you've wrote the fibonacci numbers on several dozen times before.

However, you will notice there are some angry lines of code demanding action of you. To apease this furious code you must first create a Twitter Developer account and get some auth tokens. Like a boss.

### Registering for a Twitter Developer Account
If you already have a Twitter account simply go to https://dev.twitter.com/user/login and login with your Twitter username and password. Otherwise, hit the **Sign Up** button and create a Twitter account.

Next go to https://apps.twitter.com and **Create A New Application**. I'm going to go ahead and call this app *Celebrememe*. Note, the form will ask you to input a website. Feel free to just make something up, it doesn't need to real. Just make sure to prepend it with `https://`.

Once you successfully created the app, click on the **Keys And Access Tokens** tab. Copy the **Consumer Key** and **Consumer Secret** keys and paste them into the respective variables on `test_meme.py.`

Now scroll to the bottom of the page and click **Create my acces token**. Copy the generated **Access Token** and **Access Token Secret** strings into the respective variables on `test_meme.py`

The next two lines are kind of boring so I just filled them in for you. Basically Twitter needs to make sure you are who you say you are and that you are allowed to play around with their API. To do this, Twitter uses [OAuth](http://oauth.net/) which is a cool authorization protocol that allows for things like you posting a tweet on someone's behalf without any third party ever having to know what their password is.

###Write The Codes
Now the fun begins. Let's create an ```API``` object from the tweepy library using the auth credentials from the previous two lines:

```
api = tweepy.API(auth)
```

Now with the ```api``` variable, a tweepy API object, we have access to all of the Twitter API calls as methods. Each method will accept some parameters and return a response from Twitter.

For example let's search for Twitter users by their name. If we check out the tweepy docs, there is a function that appears to fit our needs called [search_users](http://tweepy.readthedocs.org/en/v3.2.0/api.html?highlight=search_user#API.search_users):

```API.search_users(q[, per_page][, page])```

>Run a search for users similar to Find People button on Twitter.com; the same results returned by people search on Twitter.com will be returned by using this API (about being listed in the People Search). It is only possible to retrieve the first 1000 matches from this API.

Sounds good to me!

```
users = api.search_users(q='Kim Kardashian')
```

Here we are calling the ```search_users``` method on our tweepy ```API``` object and passing in the argument of 'Kim Kardashian'. ```q``` in this case stands for query. This method will make a GET request to the Twitter API for all users that match the name 'Kim Kardashian' and return a list of users that best fit that match.

We will store that list of users in the ```users``` variable. The results are ordered by most relevant so it's safe to assume in our little test app that the first user in this array is the real Kim Kardashian.

```
user = users[0]
```

Now ```user``` contains a single tweepy ```User``` object. Unfortunately, the tweepy docs aren't really complete with all of the methods and data attributes for ```User``` but referring to this section [here](http://tweepy.readthedocs.org/en/v3.2.0/getting_started.html#models), we can see the User object has two data attributes that are particularly interesting: ```screen_name```, and ```followers_count```. Let's print these and get our first glimpse into the magic of API's.

<code>
print user.screen_name
<br />
print user.followers_count
</code>

Now if you run this in your terminal by doing ```python test_meme.py``` or if you run this right within Sublime Text like a champ with cmd + b, then you should see something like the following output.

<code>
KimKardashian
<br />
29621313
</code>


#### SUCCESS!!!
<img src="http://cattitudebehavior.com/wp-content/uploads/2014/06/success.png" alt="Success" style="width: 250px;"/>

Now that we have the user, it should be cake to get a list of their tweets. Referring to the tweepy documentation we find the useful  [user_timeline](http://tweepy.readthedocs.org/en/v3.2.0/api.html?highlight=user_timeline#API.user_timeline) method:

```API.user_timeline([id/user_id/screen_name][, since_id][, max_id][, count][, page])```

>Returns the 20 most recent statuses posted from the authenticating user or the user specified. It’s also possible to request another user’s timeline via the id parameter.

The ```user_id``` parameter seems like an easy way to get Kim's last tweets because we already have the Kim Kardashian, tweepy User model. So let's call the ```user_timeline``` method of our API object, and pass in this id as an argument. In response, we should get a list back of Kim's 20 most recent tweets.

<code>
tweets = api.user_timeline(user.id)
<br />
tweet = tweets[0]
</code>

Again, we are just going to grab the first one since this is the most recent tweet. Go ahead and print this tweet object: ```print tweet```.

There is a lot to parse through and it's mostly just incredibly detailed metadata.  However, the general structure is just a dictionary. Each key is some parameter, like ```id``` which corresponds to some value like ```573018197873463296```. Some of these keys have values which are entire objects such as the ```author``` key which has a whole ```User``` object as it's value.

But we are only concerened with one thing for our app, the text of the tweet. Sure enough there is a key ```text``` that holds the text of the tweet. If we replace that ```print tweet``` statement with

```
print tweet.text
```

We should see the following output:

<code>
KimKardashian
<br />
29622423
<br />
@miriam__rueda lol
</code>


Of course your output will differ based on whatever piece of historical record Kim has laid down at the time of you doing this tutorial.

***

Our app requires a three step process:

1. Get Twitter user that our webpage user searched for
2. Get Twitter user's most recent tweet
3. Turn most recent tweet into a meme

So far we haven't even touched the web server aspect of our app but we have completed 2 out of the 3 main functionalities of our Celebrememe app.

Now all that is left to do is transfer this code onto a Flask web server, write up a few simple HTML pages, and use [apimeme.com](www.apimeme.com) to turn the text of the tweet into a meme.

So in part 2 we will do just that. See you then!
