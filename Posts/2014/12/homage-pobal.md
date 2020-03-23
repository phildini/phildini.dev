Permalink: /homage-for-the-holidays-pobal-twitter-link-aggrega/
Title: Homage for the Holidays: Pobal - Twitter Link Aggregation
Date: 2014-12-16
Tags: homage-for-the-holidays

# Homage for the Holidays: Pobal - Twitter Link Aggregation

I think this week's Homage for the Holidays solidifies my status in the Andy Baio fanclub. My card is in the mail, I'm sure.

Not too long after XOXO, I saw a link on Andy's twitter to this site called [BELONG](http://belong.io "BELONG"). I wasn't sure what I was looking at, at first. It looked like collection of interesting links pulled from twitter and aggregated. I got the impression that it was links from people Andy follows, but I've never really known anything about how it was constructed until I was researching this blog post. (The most Andy seems to have talked about it is in [this Product Hunt listing](https://www.producthunt.com/posts/belong-io).) BELONG is a collection of interesting things shared by people Andy thinks are themselves interesting.

Let's talk about what it does, or at least does for me.

BELONG shows me a set of viewpoints, lets me peek into world views that I'm not sure how else I would've been exposed to. Product announcements, interesting articles, discussions on race and class and gender and equality - BELONG mixes all of these into a half-daily-ish digest that serves me better than any 'social news' site I've seen. I've been turned off Reddit almost completely this year, the Hackernews echo chamber is wearing thin in some places, and once a week is about all I can deal with my Facebook feed. Yet I check my twitter, and BELONG, multiple times a day.

I began to wonder what my own twitter feed would look like if it were given the same treatment as BELONG, so I built one. It's called POBAL ( an Irish word for community) and you can find [mine under my pebble.ink account](http://pebble.ink/~phildini/ "POBAL").

Let's talk for a brief moment about what POBAL is, starting with some techno-babble. Feel free to skip to the next paragraph for a tl;dr. POBAL is a python script, a shell script, an html template, and a cron job. The POBAL script, triggered to run every hour by cron, pulls tweets from my twitter feed, figures out which ones have links, fetches the titles for the pages being linked to, and renders a nice list to html. The links are (currently) weighted by one-half the number of favorites plus the number of retweets. The algorithm may change as I play with it more. All of the code, minus the one line of cron, lives in [the POBAL github](https://github.com/phildini/pobal "POBAL on GitHub").

So: POBAL is a collection of interesting links from my twitter.

You're welcome to POBAL it as you see fit. If you'd like to use this but don't want to do the setup, I'm hoping to get POBAL to a point this week where others can have their own easily. You're welcome to ask me for help, and I will lovingly take feedback (my design sense is probably atrocious, and the logo was generated from a Python art program). Pull requests also welcomed.

This is in many ways the project that really inspired Homage for the Holidays, and the one that I will probably use the most. There's something indescribable about seeing what your network is sharing with you. You begin to get a sense of the caliber of people you follow, what your network cares about, and by extension what you care about. I've been using POBAL for about 24 hours, and it's already prompted me to take a good hard look at who I'm following, and the quality of what they're adding to my life. I've followed some others, and unfollowed some dead weight (mostly corporate twitter accounts).

But the other thing about seeing all the best links from your twitter listed out is that they get harder to ignore. Social media isn't really ephemeral, in that nothing ever actually dies, but the way we consume it often is. Pulling the materials being shared out of the stream-of-consciousness context forces you to look at them more critically, to evaluate what normally drifts past your eyeballs. In the best case, it exposes you to thoughts that make you and those around you better human beings.

POBAL is not BELONG. The code is different, the algorithm is different, the design is different (way worse, most likely) but the spirit of gathering news from your network how you want is there. That makes it a decent homage, I think. If I'm feeling grandiose, BELONG is a facet of the new oral tradition we call social media, of which POBAL is an imperfect mirror. If I'm being more realistic, POBAL was just fun to build and a kick to use.

Hope you enjoy.