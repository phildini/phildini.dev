Permalink: /django-csrf-httponly/
Title: Why Doesn't the Django CSRF Cookie Default to 'httponly'?
Date: 2015-10-19
Tags: tech,django

# Why Doesn't the Django CSRF Cookie Default to 'httponly'?

Recently, some questions asked by a friend prompted me to look deeper into how Django actually handles it's CSRF protection, and something stuck out that I want to share.

As a refresher, [Cross-Site Request Forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) is a vulnerability in web applications where the server will accept state-changing requests without validating they came from the right client. If you have example.com/user/delete, where normally a user would fill out a form to delete that account, and you're not checking for CSRF, potentially any site the user visits could delete the account on your site.

Django, that marvelous framework for perfectionists with a deadline, does some things out-of-the-box to try and [defend you from CSRF attacks](https://docs.djangoproject.com/en/1.7/ref/contrib/csrf/). It comes default-configured with the CSRF middleware active in the middleware stack, and this is where most of the magic happens.

The [middleware](https://github.com/django/django/blob/master/django/middleware/csrf.py) works like so: When it gets a request, it tries to find a csrf\_token in the request's cookies (all cookies the browser knows about for a URL are sent with every request to that URL, and you can read about some interesting side-effects of that here: [Cookies Can Be Costly On CDNs](https://www.eventbrite.com/engineering/cookies-can-be-costly-how-separate-domains-can-save-big-bucks/)). If it finds a token in the cookie, and the request is a POST request, it looks for a matching token in the request's POST data. If it finds both tokens, and they match, hooray! The middleware approves the request, and the request marches forward. In all other cases, the middleware rejects the request, and an error is returned.

The CSRF middleware also modifies the response on its way out, in order to do one important thing: set the cookie with the CSRF token to read. It's here that I noticed something interesting, something that struck me as curious: The CSRF token doesn't default to 'httponly'.

When a site sets a cookie in the browser, it can choose to set an 'httponly' property on that cookie, meaning the cookie can only be read by the server, and not by anything in the browser (like, say, JavaScript). When I first read this, I thought this was weird, and possibly a mistake. Not setting the CSRF token 'httponly' means that anyone who can run JS on your pages could steal and modify the CSRF cookie, rendering its protection meaningless.

Another way to read what I just wrote would be: "If my site is vulnerable to Cross-Site Scripting (XSS) attacks, then they can break my CSRF protection!" This phrasing highlights a bit more why what I just said is funny: If your site is vulnerable to an XSS attack, that's probably game over, and worrying about the CSRF protection is akin to shutting the barn door after the horse has been stolen.

Still, if the CSRF cookie defaulted to 'httponly', and you discovered your site had an XSS, you might breathe a little easier knowing that bad state-changing requests had a harder time getting through. (Neglecting other ways the cookie could be broken in an XSS attack, like cookie jar overflow). I was talking to Asheesh Laroia about this, and he called this the "belt-and-suspenders" approach to securing this facet of your web application. He's not wrong, but I was still curious why Django, which ships with pretty incredible security out-of-the-box, didn't set the default to 'httponly'.

We don't know the answer for sure (and I would love to have someone who knows give their thoughts in the comments!), but the best answer we came up with is: AJAX requests.

The modern web is composed less-and-less of static pages. Increasingly, we're seeing rich client-side apps, built in JavaScript and HTML, with simple-yet-strong backends fielding requests from those client-side apps . In order for state-changing AJAX requests to get the same CSRF protection that forms on the page get, they need access to the CSRF token in the cookie.

It's worth noting that we're not certain about this, and the Django git history isn't super clear on an answer. There is a setting you can adjust to make your CSRF cookie 'httponly', and it's probably good to set that to 'True', if you're certain your site will never-ever need CSRF protection on AJAX requests.

Thanks for reading, let me know what you think in the comments!

_**Update (2015-10-19, 10:28 AM):** Reader [Kevin Stone](https://twitter.com/kevinastone) left a comment with one implementation of what we’re talking about:_

```javascript
$.ajaxSetup({
    headers: {
         'X-CSRFToken': $.cookie('csrftoken')
    }
}
```

_Django will also accept CSRF tokens in the header ('X-CSRFToken'), so this is a great example._ 

_Also! Check out the comment left by [Andrew Godwin](https://twitter.com/andrewgodwin) for confirmation of our guesses_.
