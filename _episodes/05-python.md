---
title: "Making Python Not Awful with Containers"
teaching: 10
exercises: 0
questions:
objectives:
keypoints:
---
## Why can Python be Awful? ##

Python is a great language for doing all kinds of work.

![Python can get messy]({{ page.root }}/fig/python-complexity-cartoon.png)
## Building a Python Container##

This is a quick Python data science example.  We'll build a Python container, add a simple Python app, and run it.

To begin, let's clone another repo

```
cd $HOME
https://github.com/skjerven/python-demo.git
cd python-demo
```

There a few files here:

* `Dockerfile` - Outlines how we'll build our container
* `my_app.py` - A simple python app that builds a decision tree using Scikit
* `requirements.txt` - What python packages we want to install with pip


We can simply run

```
docker build -t python-demo .
```
to build our container.

After that, you can run it with

```
docker run python-demo
```