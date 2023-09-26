---
title: "Python: venv"
date: 2023-01-07T20:22:38+02:00
draft: false
---

# Python series, part 1: venv

I'm planning a couple of posts about Python and we will start with a small and basic introduction
about venv. The reason for using venv is that we often want to use different versions of the same
Python package and venv is a built-in tool to create virtual environments for each project so that we
don't run into version conflicts. There are other tools to accomplish the same (and more), but since venv comes
with the standard Python distribution, I prefer venv. In the next post I plan to write a small C extension.
With that out of the way, let's jump right in! In your project folder (I named mine 'cextension'),
issue the following command:

```
python3 -m venv cext_tutorial
```

I named my virtual environment cext_tutorial, often people will just name it venv, but I wanted a different name
so that there will be less confusion. If we now use the ls command to show us what is inside our
previously empty project folder, we see the following:

```
cext_tutorial
```

This folder contains all the stuff related to our virtual environment, if we look into the folder, we see:

```
bin  include  lib  lib64  pyvenv.cfg
```

Before we use our virtual environment, we first have to activate it:

```
source cext_tutorial/bin/activate
```
Our virtual environment is now activated, the name should appear in your terminal. You can also use
the command 'which python' to check, your output should be '/path/to/your/project/folder/bin/python'.
If we run 'pip list' to show us all the installed packages, we only see the following:

```
pip        22.2
setuptools 59.6.0
```
As you can see, it's like a new fresh install that is seperate from your global installation. Exactly what we wanted.
If we install something inside our virtual environment, it will only be available within it and leave our global
Python installation as is. Feel free to install packages with 'pip install' if you want to experiment with it in
your project. And if you want people to replicate your virtual environment, you just run
'pip freeze > requirements.txt'. Other people then only need this requirements.txt file to reproduce
your virtual environment by running 'pip install -r requirements.txt'. You should not distribute your virtual
environment folder, only the requirements.txt file. And keep your virtual environment folder clean, by which I mean,
don't put any files in there that didn't get created automatically by venv. We can also deactivate our
virtual environment with, you guessed it, 'deactivate' and we can remove it completely by just deleting the folder:
'rm -rf cext_tutorial'. And that's it for now, this was a very basic introduction to virtual environments in Python
and the built-in venv tool, next post will be about how we can write a simple C extension so that we can speed up
cpu heavy Python code.

