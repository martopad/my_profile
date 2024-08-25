+++
title = "About Me"
description = ""
aliases = ["contact"]
+++

I was initially a C++ developer maintaining an ancient, monolithic,
and massive application for 3g BTS' (telecommunications) in Nokia Philippines.
My primary responsibility was to:

1. Implement fixes and features to meet customer needs on domain-specific code, such as:
    * How to achieve use-case A.
    * Fixing issues when the customer/tester uses case B.

As time passed, my interests went away from understanding domain-specific
codebases to general code deployment, build-system, and toolchains type of stuff.
I self-studied how to build this monolithic codebase. It started with the simple questions:

1. What does this "configure.sh" do?
2. What are these CMakeLists files that are littered everywhere?
3. Why do people hate CMake, but it became the defacto [meta] build system?
4. Why do some targets require manually adding files in CMakes while others do not?

Then the questions got bigger:

1. How much resources does it take to build the whole binary?
2. Why only write one piece of code for all the architectures we support?
3. How do we make sure that the development environment and CI nodes environment are the same?
4. When I commit my code to the SVN repo, what does the CI do with it?
Where are the knobs and levers that run this machination?

The things that I started to get curious about are all "stable."
The old repository is in maintenance mode, yet my curiosity got the best
of me. Little did I know that this path is the path to DevOps.
I only knew it was DevOps work when I relocated to Nokia Poland.
Luckily, the team that hired me is pushing DevOps practices to the whole Mobile Networks Department.

My tasks shifted to:
1. Investigating integration issues.
2. Create centralized tools or wrappers for tools.
3. As of writing, about 50+ git repositories.
4. Linux Administration for our remote developer machines.
5. Be the link between:
    * Developer/Tester Teams
    * Infrastructure Teams
    * CI Services Teams (Yes, they are different than infra)
    * Yocto Teams

I would say, if there is a spectrum and DevOps is in the middle,

    `[Developer] <---- [DevOps] ----> [Operations]`

I would be in the Middle of *Developer* and *DevOps*.

    `[Developer] <---- [DevOps] ----> [Operations]`
                    ^ Here

While my skills as a Developer were initially stronger than those in Operations,
my role in DevOps has allowed me to broaden my skill set. I have transitioned from
being a developer-only to a more versatile, 'Jack-of-all-trades' professional with
a strong focus on DevOps practices.

To make the above paragraph more tangible, let's say when a company wants to do
something for a project. The only app-specific requirement that I will ask about
is the tech stack. I will then focus on the infrastructure that will make this
possible and the deployment process:

1. The development, CI, and production environments.
    * Toolchains and SDKs
    * Supported CPU architectures
    * Cross-compilation
2. The developer experience.
    * Tooling
3. Building, Testing, and Release process.
