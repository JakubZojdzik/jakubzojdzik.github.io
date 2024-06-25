---
title: "Debug for C++"
summary: "Steal print() from Python and put it into C++"
date: 2024-01-24T23:20:51+01:00
draft: false
---

# Competitive programming

In competitive programming, participants solve coding challenges under pressure of the time. That's why many of them use various tricks to shorten the code. In my opinion, it's very helpful even if you're still a beginner player (just like me 🙂). In this article, I will present a method for such optimization in the C++ language.

# Repository

I store all of these tools in [my repository](https://github.com/JakubZojdzik/debug). Some code snippets are derived from solutions on the [Codeforces](https://codeforces.com/) platform. Unfortunately, the code is a bit dated and has been modified by me many times, making it impossible to find an author.

# Debug function

The optimization I found most useful is custom function for printing variables. If you've never heard of it, it might not sound very helpful, but you'll quickly see how powerful it is. Thanks to [templates](https://en.cppreference.com/w/cpp/language/templates), we can make the `debug()` function accept arguments of practically any type, even those from STL. This will significantly speed up the code debugging process since printing types like vector, stack, queue, tuple will only take one line and will be nicely formatted on the screen.

In the repo, it is implemented in [`debug.hpp`](https://github.com/JakubZojdzik/debug/blob/master/debug.hpp) file. Actually, it is `define` directive, because it has access to argument name which is also printed out. Take note that the function uses `cerr` output stream in order to avoid cluttering the standard output. Additionally, if the function is executed on the judging server by mistake, its output will likely be ignored.

The print format is following:

```
[DEBUG L: <line_nr>] <variables> = <values>
```

Where `<line_nr>` is number of line where function has been called.

As the declaration is in header file, we want to include it somewhere. To make it globally accessible you should place it in correct directory, where your compilator will find it. One of these, is `/usr/include` directory. Now we need `.cpp` file. That is where we can move to second feature:

# Automatic `main.cpp` file generation

Even if we want this file to be the simplest possible, it take some time to include required libraries, open `std` namespace, define main function. Especially if we want to do it again for every task. That is why I have my ready-to-use `main.cpp` file template.

## File template

Taking advantage of the fact that I don't have to write this file from scratch for each task, I don't need to worry about its length, so I added everything that might come in handy for me.

I'm using multiple `typedef` specifiers:

```cpp
typedef long long ll;
typedef unsigned long long ull;
typedef double db;
typedef pair<int, int> pii;
typedef pair<long long, long long> pll;
typedef vector<int> vi;
typedef vector<long long> vl;
typedef vector<pair<int, int>> vpii;
typedef vector<pair<long long, long long>> vpll;
```

`define` directives:

```cpp
#define fi first
#define se second
#define pb push_back
#define rep(i, x, y) for(ll i = (ll)x; i <= (ll)y; i++)
#define all(x) x.begin(), x.end()
#define sz(x) (ll)(x).size()
#define nl '\n'
```

I include my `debug` function in smart way:

```cpp
#ifdef LOCAL
#include "debug.hpp"
#else
#define debug(...)
#endif
```

The `#ifdef LOCAL` directive is true, when the `-DLOCAL` flag is present during compilation. This ensures that if I leave debug function calls in the code I submit, its declaration will be empty on the judging server, preventing compilation errors.

You may want to customize this file according to your habits and preferences. Check the full file template [here](https://github.com/JakubZojdzik/debug/blob/master/main.cpp).

## Script for pasting template

It would be annoying to manually paste this template into new files. That's why I have written a short bash script to automate it:

```bash
#!/bin/bash
cp ~/Templates/main.cpp ./
dt=`date +%d-%m-%Y`
sed -i "s/--DATE--/${dt}/" ./main.cpp
```

As you see, your template should be present in `~/Templates` directory. To understand two following lines, we have to go back to the `main.cpp` template, specifically to the comment at the very top of it:

```cpp
/*
    --Your name--
    --DATE--
*/
```

First line of this comment is for you to fill it with your name, if you want to sign your codes. Second line contains `--DATE--` string, that script will replace with current date with `sed`.

Remember to provide executable permissions to the script:

```bash
chmod +x <script_name>
```

And if you want to have an easy access to it globally, you have to place it in one of directories from your `$PATH` variable. Probably `/usr/local/bin` is good choice.

# Compilation

Last process to speed up is compilation. Online judging systems often uses some additional flags, that we don't really want to rewrite every time. We can write bash script for compiling, or add an alias in Bash. I use the second option as it is very short. Paste these lines at the end of your `.bashrc` or its equivalent in the shell you are using:

```bash
c() {
    if [[ "$1" != "" ]]
    then
        g++ -DLOCAL -O3 -static "$1" -std=c++20 -o main
    else
        g++ -DLOCAL -O3 -static main.cpp -std=c++20 -o main
    fi
}
```

Customize the compiler and its flags according to your needs. Likely, you also want to change the alias name from `c`. Just remember, that `DLOCAL` flag is required for our `debug` function.

After creating new instance of terminal, you should be able to use:

```bash
c
```

to compile `main.cpp` file into `main` binary, or:

```bash
c file.cpp
```

to compile file.cpp into `main` binary.

# Instructions

If you just want to paste some commands to make it work, here you are:

```bash
git clone https://github.com/JakubZojdzik/debug.git
cd debug
sudo mv debug.hpp /usr/include
nano main.cpp
# Replace second file with your name
# Ctrl+o to save file, Ctrl+x to exit nano
mv main.cpp ~/Templates
chmod +x newmain
sudo mv newmain /usr/local/bin
cut bashrc_alias.sh >> ~/.bashrc
cd ..
rm -R debug
```

Of course, these commands can vary a little depending on your os and software, so be sure you know what are you doing.

# Effects

If you've followed steps correctly, you should be able to perform following actions:

- Create `main.cpp` file with the initial content:

  ```bash
  newmain
  ```

- Compile it quickly with the appropriate flags

  ```bash
  c
  ```

- Run the binary and obtain the output (if you previously added something to main)
  ```bash
  ./main
  ```

# Thank you for reading

If you've found any errors in the article, there's an edit button at the top 🙂
