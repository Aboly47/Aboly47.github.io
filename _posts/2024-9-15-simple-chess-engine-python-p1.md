# Writing a simple chess engine in Python - Part 1

So, you probably know about Stockfish, and LC0, torch, komodo, and other chess engines. but, have you ever thought about writing one?

Well, I'm here to teach you how to do exactly that, in Python!

## Important: Expectations

So, don't come into this thinking you'll write the next stockfish or LC0. we're going to be writing a very basic chess engine in Python. I'm calling it basic, but we will explore different evaluation functions, different search methods and tricks and optimizations, and even at some point, Neural Networks, and NNUE too. but, the chess engine we will make will still probably be bad. by bad I mean it will be slow, and it will make blunders. Why? because Python is slow. if you write the same code we will explore in part 3, but in C++, you will get around 12-15 depth in a reasonable time (at least that's what I got when I wrote it myself). but, in Python, we hardly get 6. which is laughable for a chess engine, but it's because Python is an interpreted programming language and it's slower than something like C++. This tutorial isn't trying to teach you how to write a superhuman chess engine. for that, your best bet is to choose something like C++, or even Rust. there are currently very strong chess engines being developed in Rust that you can find with a quick search. then, you probably should take a look at other engines' source codes and take notes. Be careful to not **steal** code though. stealing other people's code is frowned upon heavily in the chess engine community.

## So what is this?
a tutorial for you to get your hands down and dirty on chess engine development. you can get a sense of how things work and then continue from there. 😃

## workspace

you'll need a few things installed for the next part to work, so do it right now.

create a directory for your project and cd into it. (execute these line by line, and change `PROJECT_NAME` to suit the path where you want to create the project.)

```bash
mkdir PROJECT_NAME
cd PROJECT NAME
```

create a virtual environment (optional, but recommended.)
(the lines that start with # are comments, you don't need to execute them.)

```bash
# If you're using windows
python -m venv venv
# If you're using linux
python3 -m venv venv
```

activate the virtual environment (only if you created a virtual environment above.)

```bash
# Windows
./venv/bin/activate
# Linux
source ./venv/bin/activate
```

install the required package

```bash
# python chess is a python library used to work with chess stuff.
pip install python-chess
```

if everything goes well, you should have `python-chess` installed. if you encounter any errors, search on Google or post them on the comment section below and I'll respond.

# Note

if you're on Linux and you don't use a virtual environment, you might get this error:

```plaintext
error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.

    If you wish to install a non-Debian-packaged Python package,
    create a virtual environment using python3 -m venv path/to/venv.
    Then use path/to/venv/bin/python and path/to/venv/bin/pip. Make
    sure you have python3-full installed.

    If you wish to install a non-Debian packaged Python application,
    it may be easiest to use pipx install xyz, which will manage a
    virtual environment for you. Make sure you have pipx installed.

    See /usr/share/doc/python3.12/README.venv for more information.

note: If you believe this is a mistake, please contact your Python installation or OS distribution provider. You can override this, at the risk of breaking your Python installation or OS, by passing --break-system-packages.
hint: See PEP 668 for the detailed specification.
```

this just means you need to use a virtual environment. The only way around it is using pipx, which, well, I don't use or recommend.
