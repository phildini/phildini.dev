Permalink: /using-django-channels-email-sending-queue/
Title: Using Django Channels as an Email Sending Queue
Date: 2016-04-08
Tags: tech,django,channels

# Using Django Channels as an Email Sending Queue

[Channels](https://github.com/andrewgodwin/channels) is a project by led Andrew Godwin to bring [native asynchronous processing](https://channels.readthedocs.org/en/latest/concepts.html) to Django. Most of the tutorials for integrating Channels into a Django project focus on Channels' ability to let Django "speak WebSockets", but Channels has enormous potential as an async task runner. Channels could replace Celery or RQ for most projects, and do so in a way that feels more native.

To demonstrate this, let's use Channels to add non-blocking email sending to a Django project. We're going to add email invitations to a pre-existing project, and then send those invitations through Channels.

First, we'll need an invitation model. This isn't strictly necessary, as you could instead pass the right properties through Channels itself, but having an entry in the database provides a number of benefits, like using the Django admin to keep track of what invitations have been sent.

```python
from django.db import models
from django.contrib.auth.models import User


class Invitation(models.Model):

    email = models.EmailField()
    sent = models.DateTimeField(null=True)
    sender = models.ForeignKey(User)
    key = models.CharField(max_length=32, unique=True)

    def __str__(self):
        return "{} invited {}".format(self.sender, self.email)

We create these invitations using a ModelForm.

from django import forms
from django.utils.crypto import get_random_string

from .models import Invitation


class InvitationForm(forms.ModelForm):

    class Meta:
        model = Invitation
        fields = ['email']

    def save(self, *args, **kwargs):
        self.instance.key = get_random_string(32).lower()
        return super(InvitationForm, self).save(*args, **kwargs)
```

Connecting this form to a view is left as an exercise to the reader. What we'd like to have happen now is for the invitation to be sent in the background as soon as it's created. Which means we need to install Channels.

```shell
pip install channels
```

We're going to be using Redis as a message carrier, also called a [layer](https://channels.readthedocs.org/en/latest/backends.html) in Channels-world, between our main web process and the Channels worker processes. So we also need the appropriate Redis library.

```shell
pip install asgi-redis
```

Redis is the preferred Channels layer and the one we're going to use for our setup. (The Channels team has also provided an in-memory layer and a database layer, but use of the database layer is strongly discouraged.) If we don't have Redis installed in our development environment, we'll need instructions for installing Redis on our development OS. (This possibly means googling "install redis {OUR OS NAME}".) If we're on a Debian/Linux-based system, this will be something like:

```shell
apt-get install redis-server
```

If we're on a Mac, we're going to use [Homebrew](https://brew.sh/), then install Redis through Homebrew:

```shell
brew install redis
```

The rest of this tutorial is going to assume we have Redis installed and running in our development environment.

With Channels, redis, and asgi-redis installed, we can start adding Channels to our project. In our project's _settings.py_, add 'channels' to INSTALLED_APPS and add the channels configuration block.

```python
INSTALLED_APPS = (
    ...,
    'channels',
)

CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "asgi_redis.RedisChannelLayer",
        "CONFIG": {
            "hosts": [os.environ.get('REDIS_URL', 'redis://localhost:6379')],
        },
        "ROUTING": "myproject.routing.channel_routing",
    },
}
```

Let's look at the CHANNEL_LAYERS block. If it looks like Django's database settings, that's not an accident. Like we have a default database defined elsewhere in our settings, here we're defining a default Channels configuration. Our configuration uses the Redis backend, specifies the url of the Redis server, and points at a routing configuration. The routing configuration works like our project's `urls.py`. (We're also assuming our project is called 'myproject', you should replace that with your project's actual package name)

Since we're just using Channels to send email in the background, our `routing.py` is going to be pretty short.

```python
from channels.routing import route

from .consumers import send_invite

channel_routing = [
    route('send-invite',send_invite),
]
```

Hopefully this structure looks somewhat like how we define URLs. What we're saying here is that we have one route, 'send-invite', and what we receive on that channel should be consumed by the 'send_invite' consumer in our invitations app. The `consumers.py` file in our invitations app is similar to a `views.py` in a standard Django app, and it's where we're going to handle the actual email sending.

```python
import logging
from django.contrib.sites.models import Site
from django.core.mail import EmailMessage
from django.utils import timezone

from invitations.models import Invitation

logger = logging.getLogger('email')

def send_invite(message):
    try:
        invite = Invitation.objects.get(
            id=message.content.get('id'),
        )
    except Invitation.DoesNotExist:
        logger.error("Invitation to send not found")
        return
    
    subject = "You've been invited!"
    body = "Go to https://%s/invites/accept/%s/ to join!" % (
            Site.objects.get_current().domain,
            invite.key,
        )
    try:
        message = EmailMessage(
            subject=subject,
            body=body,
            from_email="Invites <[[email protected]](/cdn-cgi/l/email-protection)%s.com>" % Site.objects.get_current().domain,
            to=[invite.email,],
        )
        message.send()
        invite.sent = timezone.now()
        invite.save()
    except:
        logger.exception('Problem sending invite %s' % (invite.id))
```

Consumers consume messages from a given channel, and messages are wrapper objects around blocks of data. That data must reduce down to a JSON blob, so it can be stored in a Channels layer and passed around. In our case, the only data we're using is the ID of the invite to send. We fetch the invite object from the database, build an email message based on that invite object, then try to send the email. If it's successful, we set a 'sent' timestamp on the invite object. If it fails, we log an error.

The last piece to set in motion is sending a message to the 'send-invite' channel at the right time. To do this, we modify our InvitationForm

```python
from django import forms
from django.utils.crypto import get_random_string

from channels import Channel

from .models import Invitation


class InvitationForm(forms.ModelForm):

    class Meta:
        model = Invitation
        fields = ['email']

    def save(self, *args, **kwargs):
        self.instance.key = get_random_string(32).lower()
        response = super(InvitationForm, self).save(*args, **kwargs)
        notification = {
            'id': self.instance.id,
        }
        Channel('send-invite').send(notification)
        return response
```

We import Channel from the channels package, and send a data blob on the 'send-invite' channel when our invite is saved.

Now we're ready to test! Assuming we've wired the form up to a view, and set the correct email host settings in our `settings.py`, we can test sending an invite in the background of our app using Channels. The amazing thing about Channels in development is that we start our devserver normally, and, in my experience at least, It Just Works.

```shell
python manage.py runserver
```

Congratulations! We've added background tasks to our Django application, using Channels!

Now, I don't believe something is done until it's shipped, so let's talk a bit about deployment. The [Channels docs](https://channels.readthedocs.org/en/latest/deploying.html) make a great start at covering this, but I use Heroku, so I'm adapting the [excellent tutorial written by Jacob Kaplan-Moss](https://blog.heroku.com/archives/2016/3/17/in_deep_with_django_channels_the_future_of_real_time_apps_in_django) for this project.

We start by creating an `asgi.py`, which lives in the same directory as the `wsgi.py` Django created for us.

```python
import os
import channels.asgi

os.environ.setdefault("DJANGO_SETTINGS_MODULE", "myproject.settings")
channel_layer = channels.asgi.get_channel_layer()
```

(Again, remembering to replace "myproject" with the actual name of our package directory)

Then, we update our _Procfile_ to include the main Channels process, running under [Daphne](https://github.com/andrewgodwin/daphne/), and a worker process.

```shell
web: daphne myproject.asgi:channel_layer --port $PORT --bind 0.0.0.0 -v2
worker: python manage.py runworker --settings=myproject.settings -v2
```

We can use Heroku's free Redis hosting to get started, deploy our application, and enjoy sending email in the background without blocking our main app serving requests.

Hopefully this tutorial has inspired you to explore Channels' background-task functionality, and think about getting your apps ready for when Channels lands in Django core. I think we're heading towards a future where Django can do even more out-of-the-box, and I'm excited to see what we build!

* * *

_Special thanks to [Jacob Kaplan-Moss](https://jacobian.org/), Chris Clark, and Erich Blume for providing feedback and editing on this post._