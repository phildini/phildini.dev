Permalink: /python-in-every-app/
Title: Python should be in every application
Tags: python
Date: 2025-10-24
Summary: The best in-app scripting engine is within reach

# Python should be in every application

In the early days of [Python](https://www.python.org/)'s life, when it was newly hatched, one of the primary selling points was Python's ability to fit into a systems programming stack, to interface fairly-cleanly with C-based programming. Even today, if you throw a brick at [https://github.com/topics/c](https://github.com/topics/c), you're likely to hit a project with some Python scripts buried inside them, to help with building or deploying or improving developer quality of life. Python as a tool is still popular with systems programmers and applications programmers of all stripes.

Which is why it's so boggling to me that Python is handily losing the embedded scripting language war. Instead of using a language that has nearly 30 years of community and knowledge around interfacing with lower-level APIs and ABIs, modern application developers reach for [Lua](https://www.lua.org/) or even JavaScript when trying to bundle powerful scripting capabilities into their applications. Video games and browsers come to mind, but I've seen Lua used as a scripting language in everything from Office Suites to Application IDEs.

What's going on here? Can we make it better?

Imagine a world where you can run one command to bundle a Jupyter notebook and all it's dependencies into a linkable library usable by Swift projects or Kotlin projects or Android projects or even Unity projects. In this world, the functions being exposed by the notebook can be called from the parent application through a fairly transparent ABI layer, and the Python child can call exposed functions from the parent.

Take this a step further. Imagine a world where the Machine Learning / AI / LLM code your team is already building in Jupyter notebooks can be dropped as a linkable library in your business application, with full access to the on-device TPU / NPU. The finely-tuned ML your team has spent hours on doesn't need to be gated behind a web service running on a nuclear powerplant's worth of GPUs, it can run on the user's device without any time spent refactoring code between languages.

The good news is: we're so much closer than you might think. We have 80% of the tools we need, the challenge is bringing them all together.

Being inspired by [Anaconda](https://anaconda.org/) and the [BeeWare](https://beeware.org/) project (disclosure, I'm a listed Core Contributor to BeeWare), we can use [Briefcase](https://beeware.org/project/briefcase/) and the set of interface layers ([rubicon](https://beeware.org/project/utilities/rubicon/), etc) that have already been written to connect Swift / Objective-C / Android / Java apps to Jupyter notebooks, or really any Python code.

This command doesn't exist today, but my idea here is a world where `briefcase bundle iOS` returns a drop-in linkable bundle instead of an application, and you can add that to your Xcode project like any external dependency. You've now given your app Python as a scripting language, exposing only the internals you want to your scripting users while not having to think too hard about scripting language semantics.

There are some challenges, here, of course. For one, Lua has a real advantage in being small. Work needs to happen on Python tree-shaking so we're not asking users to add hundreds of megabytes to their apps unnecessarily. For another, the existing challenge around Python libraries that have external system dependencies. The BeeWare team and the [Python Core Team](https://github.com/python/prebuilt-cpython) are taking this seriously, but it's not ready yet. As well, the devil is in the details, and there's going to be DevEx and education work to make this "easy" for application developers.

I think the advantages if we do this work are vast. Python is no longer locked to the server, or a commitment to be your whole app framework. You can render unto Python that which is Python's, however you define that. I for one look forward to a world where I can script mods for Warcraft in Python, and tweak my app's on-device recommendation algorithms in real time in a Jupyter notebook.

---

If this is a world you want to live in, what can you do?
- Show up for your open source projects. Fix bugs, triage issues, do what you can to give the maintainers more breathing room
- Donate to open source projects and their maintainers, especially Python and ideally BeeWare
- Check out [modulegraph](https://modulegraph.readthedocs.io/en/latest/), and get an understanding of what your code is dependent on 

---

_(Hey Anaconda: "package your Jupyter notebook for your app as a service" is a pretty compelling offering, and you should hire me to help build it ❤️)_