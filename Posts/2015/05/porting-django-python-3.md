Permalink: /porting-django-apps-to-python-3-part-1/
Title: Porting Django Apps to Python 3, Part 1
Tags: projects,tech,django
Date: 2015-05-26

# Porting Django Apps to Python 3, Part 1

Hello! Welcome to the first in a series of posts about my experiences making Django apps Python 3 compatible. Through these posts I'll start with a Django app that is currently written for Python 2.7, and end up with something can be run on Python 3.4 or greater.

Some quick notes before we begin:

*   Why am I doing this? Because we have 5 years until Python 2.7 goes end-of-life, and I want to be as ready as possible for making that change in the code that I write for my job. To prep for that, I'm converting all the Django apps I can find, from side-projects and Open Source projects.
*   Why 5 years? Because that's the time outlined in [PEP-0373](https://www.python.org/dev/peps/pep-0373/), and based on [Guido's keynote at PyCon 2015](https://www.youtube.com/watch?v=G-uKNd5TSBw), that's the timeline we all should be sticking to. It's also recently been brought to my attention that further Python 2.7 releases are really the responsibility of one person, the inimitable Benjamin Peterson, and if he for any reason decides to stop making updates that 2020 timeline may get drastically shortened. It's better to be prepared now.
*   Why "Python 3 compatible"? Why not fully Python 3? Because I believe the best way forward for the next 5 years will be writing polyglot code that can be run in either Python 2.7 or Python3.4+ environments. (I'm going to start shortening those to py2 and py3 for the rest of this post.) So I won't be using 2to3, but I will be using six.

With those pieces in mind, let's begin!

I started with Cards Against Django, a Django implementation of Cards Against Humanity that I wrote with some friends a couple years ago. We didn't own Cards Against Humanity, and hilariously thought it would be easier to build it than to buy it. (We also may have just wanted the challenge of building a usable Django app from scratch). The end result was a game that could be played with an effectively unlimited number of players, each on their own device, and which was partially optimized for mobile play. To get a sense of what the code was like before I started the migration, browse the Github repo at [this commit](https://github.com/phildini/cards-against-django/tree/49f99f2268c39c37dcf9a47c9161ecc30a241310).

Now it turns out I made one assumption right at the beginning of this port that made things a bit harder, and may have distracted from the original mission. The assumption was that Django 1.5 is not py3 compatible, when in fact it was the first py3-compatible version. Had I found and read this [Python 2 to 3 porting guide for Django](https://docs.djangoproject.com/en/1.8/topics/python3/), I may have saved myself some headache. You now get the benefit of a free mini-lesson on upgrading from Django 1.5 to Django 1.8.

## Resource #1: The [Django Python 3 Porting Guide](https://docs.djangoproject.com/en/1.8/topics/python3/)

Real quick, I'm going to go through how my environment was set up at the beginning of this project, based on the starting commit listed above.

This snippet will setup a virtual environment using [mkvirtualenv](https://virtualenvwrapper.readthedocs.org/en/latest/), install the local requirements for the app, and initialize the db using the local settings.

Ok, let's upgrade to Django 1.8 `$ pip install -U Django` ..and naively try to run the dev server.

Well that's a bummer, but fairly expected that I wouldn't be able to make the jump to 1.8 easily. What's interesting about this error is that it's not _my_ code that seems to be the problem -- it looks like the problem is in django-nose.

```shell
$ pip install -U django-nose nose
```

Try runserver again...

Hmm... obviously the API for transactions changed between Django 1.5 and Django 1.8. Here I looked at the Django release notes, and noticed that 'commit-on-success' was deprecated in 1.8. Digging in to the new transaction API, it looked like 'transaction.atomic' was pretty much the behavior I wanted, so I went with that.

## Resource #2: The [Django Release Notes](https://docs.djangoproject.com/en/1.8/releases/)

Third time's the charm, yes?

Apparently not. This one was weird to me, because I didn't have South in my installed apps. Through a sense of intuition that I can't really explain, I suspected django-allauth, the authentication package this project uses. I wondered if an older version of django-allauth was trying to do South-style migrations.

```shell
$ pip install -U django-allauth
```

Sure enough, an old version of allauth was the culprit, and an upgraded version allowed the runserver to launch successfully.

So now I have the development server running, but I've got that warning about needing to run migrations. This is the part of this upgrade that I knew was coming, and I was most worried about. I already have the database initialized from Django 1.5's 'syncdb' -- what will happen when I run 'migrate'?

It turns out, not a whole lot. Running this command gave me a 'table already exists' DatabaseError. Googling for this issue left me a little stumped, so eventually I turned to the #django channel on Freenode IRC. (If you're curious how to get a persistent connection to IRC, check out [this post](/irc-all-the-way-down-znc-irccloud-quassel/).) I was able to get some great help there, and it was suggested I try the one-two punch of:

That '--fake' bit did the trick, convincing Django I had run the migrations (since the tables were already correctly created), and silencing the warning.

With the development server running on Django 1.8 (including the very limited test suite), I'm feeling confident about the migration to Python 3. Is my confidence misplaced? Find out in part 2!

If you'd like to see the totality of the work required to migrate this Django app from 1.5 to 1.8, check out [this commit](https://github.com/phildini/cards-against-django/commit/3807e5d08daf07afa3a629c0b361567a4d27293c).

_If you have feedback about what I did wrong or right, or have questions about what's here, leave a comment, and I'll respond as soon as I'm able!_
