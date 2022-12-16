---
layout: post
title:  "Writing a CLI Testing Framework"
date:   2022-12-10 5:40:21 -0800
categories: general-dev
---
Today, I embarked on revisiting some older projects of mine. Among the mix,
there were things like [OpenMaps](https://github.com/zeplar-exe/OpenMaps),
and a little [physics engine](https://github.com/zeplar-exe/Basic-Physics-Engine).
The biggest one though, was [FsTag](https://github.com/zeplar-exe/FsTag). A project
I picked up and promptly abandoned when I felt it was done (being a way to pass time, 
after all).

So, I patched it up and am proud to say I changed absolutely nothing. Except, I
wanted to write some unit tests for it. And guess what? There's not a single
test framework (that I could find) for command line tools. There's bash-based ones,
but I hate bash. Alas, I got to working on my own, 
[ClUnit](https://github.com/zeplar-exe/ClUnit). It's nothing too big, just a big
wrapper over the command line. Which I'll tell you, was more annoying than it was
worth.

The initial idea was to have a persistent terminal as long as tests were running, or
at least a persistent terminal per session. The developer was also allowed to make
new sessions as well. This all locked me into a very ugly system, which had a 
50/50 change to hang on any given command. Thus, I scrapped it all and went for the
traditional "make a cmd process with /C arguments, and ReadToEnd()." Point is,
it was very annoying, and I'll have to come back to it eventually.

Otherwise, that's all. I'll be posting updates as I go along with this (as well as
 other projects). FsTag will also share the spotlight, alongside BakedEnv, and
my operating system, and basically everything else. You get the point.