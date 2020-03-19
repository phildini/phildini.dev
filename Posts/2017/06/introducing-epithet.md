Permalink: /introducing-epithet/
Title: Introducing Epithet
Date: 2017-06-13
Tags: projects,python,open-source,git

# Introducing Epithet

There are many challenges to running an Open Source organization, but the one that I have personally felt the pain of again and again is that our tooling is awful. Github (and realistically we’re all using Github at this point) still feels in many ways like a tool designed around the idea that all the action is going to happen in one repo. This may not be entirely the fault of Github. Git itself is very tightly coupled to the idea that anything you care about for a particular action is going to happen in one, and only one, repository.   
   
When Github released Organizations, the world rejoiced, because we could now map permissions and team members in our source repository the way they were mapped in the real world. Every new feature Github adds to its Organizations product causes more rejoicing, because so many teams work across multiple repos, and the tooling around multiple repos is still awful.  
   
The awfulness of this tooling is probably a strong factor in the current trend towards “microservice, monorepo” code organization, but that’s another post.   
   
I’ve been the equivalent of a [core contributor](https://www.wordfugue.com/tips-becoming-core-contributor/) for a half dozen Github organizations, and I’ve noticed that one area where the tooling is especially lacking is around [labels](https://help.github.com/articles/applying-labels-to-issues-and-pull-requests/). I’ve seen labels used to designate team or individual ownership, indicate the status of pull requests, signal that certain issues are friendly for beginners, and even used as deploy targets for chunks of code. It’s fair to say that labels form a core tool in the infrastructure of every team I’ve seen using Github, and yet the tooling Github exposes for labels is painfully lacking.  
   
I could go on and on about this, but my goal here isn’t to necessarily make Github feel bad. I hope they’re working on better label tooling, and if they want ideas, boy am I willing to give them. But there is one label-specific wall I kept banging my head against, and that is label consistency across all the repos of an Organization.   
   
Some of you read that and feel remembered pain. I feel that pain with you, and we are here for each other. Some of you might have no idea what I’m talking about, so I’ll explain a bit more.  
   
Let’s say you want to add a “beginner-friendly” label to all the repos in your Open Source Organization, so that new contributors can find issues to start with. Right now on Github, you would need to go into every repo, click into the Issues page, click into the Labels tab, and manually create that label. There are no “Org-wide labels”, and no tool for easily creating and updating labels across all the repos of an organization.  
   
Until now.  
   
Introducing [Epithet](https://github.com/phildini/epithet), a Python-based command line tool for managing labels across an organization. You give it a Github key, organization, and label name, and it will make sure that label exists across all the repos in your org. Give it a color, and it’ll make the color of that label consistent across all repos as well. Have you decided you’re done with a particular label? Epithet can delete it from all your repos for you. Are you using Github Enterprise? Epithet supports that too.  
   
Epithet exists to fill a very particular need in open (and closed) source Github organizations, and it’s still pretty alpha. We use it for the [BeeWare](https://pybee.org/) project, and it might be used soon for syncing labels in the [Ragtag](https://ragtag.org/) organization. You can start using it today by checking out the (sadly small) documentation, and if there’s a feature missing you’d like to see, I’m happy to work with you on getting a PR submitted.  
   
Managing Open Source organizations is hard. My hope is Epithet makes it a little bit easier.

* * *

_WordFugue is independent, and we will never run traditional ads. If you like what we're doing, consider donating to [phildini's Patreon](https://www.patreon.com/phildini), or buy a book from our affiliate store. This week we're reading Patrick Rothfuss' ["The Name of the Wind"](https://www.amazon.com/gp/product/0756404746/ref=as_li_qf_sp_asin_il_tl?ie=UTF8&tag=wordfugue-20&camp=1789&creative=9325&linkCode=as2&creativeASIN=0756404746&linkId=2ed8ee6164bac01766b21d4acb39ef7f)._

_Special thanks to [Katie Cunningham](http://therealkatie.net/) and [Kenneth Love](http://gigantuan.net/) for reviewing this post._  
