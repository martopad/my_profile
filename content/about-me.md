+++
title = "About Me"
description = "Description embedded in about.md."
aliases = ["contact"]
+++

I am initially a C++ developer maintaining a very old, monolithic, and huge application for
a 3g BTS (telecommunications). As time went on, my interests went away from
understanding domain-specific codebases to general code deployment, build-system,
and toolchains type of stuff. I self-studied how this monolythic codebase is built.

It started with the simple questions:

1. What does this "configure.sh" do?
2. What are these CMakeLists files that is littered everywhere?
3. Why do people hate CMake but it became the defacto [meta] buildsystem?
4. Why do some targets require manually adding files in CMakes while others do not?

The questions got bigger:

1. How much resources does it take to build the whole binary?
2. How come we only write one piece of code for all of the architectures that we support?
3. How do we make sure that the development environment and CI nodes environment are the same?
4. When I commit to the SVN repo, what does the CI do it? Where are the knobs and levers that runs this machination?

The thing is that I started to get curious about are all "stable". The old repository is in
maintenance mode and yet my curiousity got the best of me. Little did I know that this path is
the path to "DevOps".

Modifying CMakes, Knowing how something is built, the infrastructure needed. It was a lot of work
, frankly It was outside my scope of resposiblities, but it was all to know something. 

If there is a `*Developer* <---- *DevOps* ----> *Operations*` spectrum and DevOps is in the middle,
I would be in the Middle of *Developer* and *DevOps*.

Little did I know that the questions that I asked about:
1. Multi-arch suport
2. Toolchains and SDK handling
3. Build enviroments
4. Automation

Would all be concerns about *DevOps* I never really knew. I was just curious about those things,
and also got bit by those concerns from time to time.

[BLAH] [Most likely refactor]
