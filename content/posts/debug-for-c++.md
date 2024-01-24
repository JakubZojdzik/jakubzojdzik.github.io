---
title: 'Debug for C++'
summary: 'Steal print() from Python and put it into C++'
date: 2024-01-24T23:20:51+01:00
draft: true
---

# Competitive programming

In competitive programming, participants solve coding challenges under pressure of the time. That's why many of them use various tricks to shorten the code. In my opinion, it's very helpful even if you're still a beginner player (just like me 🙂). In this article, I will present a method for such optimization in the C++ language.

# Repository

I store all these tools in [this repository](https://github.com/JakubZojdzik/debug). Some code snippets are derived from solutions on the [Codeforces](https://codeforces.com/) platform. Unfortunately, the code is a bit dated and has been modified by me many times, making it impossible to find an author.

# Debug function

The optimization I found most useful is custom function for printing variables. If you've never heard of it, it might not sound very helpful, but you'll quickly see how powerful it is. Thanks to [templates](https://en.cppreference.com/w/cpp/language/templates), we can make the `debug()` function accept arguments of practically any type, even those from STL. This will significantly speed up the code debugging process since printing types like vector, stack, queue, tuple will only take one line and will be nicely formatted on the screen.

in repo, this function is implemented in [`debug.hpp`](https://github.com/JakubZojdzik/debug/blob/master/debug.hpp). Actually, it is `define` directive, because it has access to argument name. Take note that the function uses cerr instead of cout to avoid cluttering the main output.

The print format is following:

```
[DEBUG L: <line_nr>] <variables> = <values>
```

Where `<line_nr>` is number of line where function has been called.

As the declaration is in header file, we want to include it somewhere. That is where we can move to second feature:
