---
layout: post
title: 'Learning: Terminal-based Notebook App in Java'
categories:
- musings
tags:
- software engineering
- java
- degree
date: 2023-06-26 20:39 -0700
---
Recently, I've decided to go back to school and earn my degree. Even after seven years in the industry, it's something I've always wanted to do, and as Shia once famously said, "don't let your dreams be dreams."

An inevitable part of this journey seems to be learning Java. This was something I was not particularly looking forward to... However, (surprise, heh) I've found that the language isn't as bad as everyone says.

The final assignment in my "Intro to Java" course was to create a program to solve a basic, yet "real world" problem. To that end, I created a simple terminal-based note-taking program, available here.

Of course, writing a simple note-taking app isn't enough to understand the nuances of a language, its strengths, and weaknesses.

However, in my limited experience with Java, here are the things I've learned/noticed:

1. There doesn't seem to be the concept of default parameters in Java. This is a bit odd to me and required me to overload the constructor to achieve something similar to default parameters.
2. The way imports work is that they belong to packages. Packages are declared at the top of files and must reside within the same folder, named the same as the package, to import your classes into other files.
3. This is my opinion, but it is exceptionally verbose. I feel like I have to type a lot more to get the same amount of work done.
4. Just like in other languages, I don't like the idea of an implicit "this," but IDE tooling makes this a non-issue.
5. It's cool that the JDBC exists. It's nice to have a single interface for using so many different and seemingly disjointed databases.
6. It's interesting to see conventions in this language that are also used in others, as a consequence of legacy.

Overall, I think if you have experience with other languages, Java will be very easy to pick up. I do recommend learning it. The reason as to "why" they do things in certain ways in other languages becomes a little clearer after learning some Java.

Be excellent to each other,
- L
