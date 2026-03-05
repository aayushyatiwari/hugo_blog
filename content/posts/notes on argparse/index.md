---
    title: "Python's argparse"
    date: 2026-03-04
---
In this article I'd like to introduce you to a rather useful python library that can be of use to you.      
It's called `argparse` and recently I have been using it as my go to for couple of things.      

I first got to know about this library when participating in a kaggle comp.         
It was pretty intimidating at first because you're not sure what's going on but after this article I am hoping you'd know how to deal with code that deals with argparse. We'll also talk about config files and how this library can be used to write config file.      

# historical context to `argparse`

Argparse is a python library that was introduced in Python 3.2     
It is a library that's used for **command-line** parsing.       
Before this, `optparse` was the official cli parsing module, but in python 3.2 they changed it to argparse.     

# what is it?
As said, it's a tool that can be used to parse command-line arguments.      

In this article we will write a very simple script to go to a website from your terminal.       

# Basics
Some basics first.      

when we run 
```bash
pythonfile.py arg1 arg2 arg3
```
all of the args are stored in `sys.argv` list.
**How `sys.argv` works:**

- `sys.argv[0]` = the script name itself
- `sys.argv[1]` = first argument
- `sys.argv[2]` = second argument
- and so on...

For example: 
I'll create a file named `helloworld.py` and it's just a template for now.
```python
import argparse

def parse():
    pass 

if __name__ == "__main__":
    parse()
```

Now to use the argparse lib, we first have to instantiate an object.

```python
import argparse

def parse():
    parser = argparse.ArgumentParser() 

if __name__ == "__main__":
    parse()
```
And now, this `parser` has methods. one of them being: `add_argument`.      

Remember [the way the arguments are stored in sys.arg?](#basics)        
You can have a look at the arguments using the below script.        

```python
import argparse
import sys

    
def parse():
    parser = argparse.ArgumentParser()
    parser.add_argument("filename")
    parser.add_argument("--count", type=int)
    args = parser.parse_args()
    print(sys.argv) # ---> the list of arguments from cli
    print(args) # ---> namespace object

if __name__ == "__main__":
    parse()
```

run the script using something like this:
```bash
python script.py hello.txt --count 5
```

If you do everything correctly, you can have a look at the `sys.argv` list and a [namespace object](#namespace).

```shell
$ python parse_example.py hello.txt --count 5
['parse_example.py', 'hello.txt', '--count', '5']
Namespace(filename='hello.txt', count=5)
```
As we can see, `sys.argv` is just a raw list of strings, and args is the `Namespace` with actual types attached. 

- the `.add_argument()` method is used to add keys to the namespace
- the `.parse_args()` used to create the Namespace object

## namespace
### **What is Namespace**
It's a simple class. When you call parse_args(), argparse creates a Namespace object and attaches your arguments as attributes to it.       

So instead of a dict, you get `args.lr` instead of `args[lr]`. Cleaner to read and write.       


# Uses

I personally use them to create config files for my model training runs.        
Say if I have to test different hyperparameters and check which of them are working better, I'd have to manually go to the code and change everything if I didn't have this library.        
With this library you can just create a python3 script with `argparse` in it and take the hyperparameters from the cli.     

```python
import argparse

def parse():
    parser = argparse.ArgumentParser()
    parser.add_argument("--lr", type=float, default=0.001)
    parser.add_argument("--epochs", type=int, default=10)
    parser.add_argument("--batch_size", type=int, default=32)
    return parser.parse_args()

def train(args):
    print(f"Training with lr={args.lr}, epochs={args.epochs}, batch_size={args.batch_size}")

args = parse()
train(args)
```

Run this like:
```shell
python train.py --lr 0.01 --epochs 50 --batch_size 64
```
And you're done. No touching the code.      

You can be more fancy by using a `config.yaml` file but that's outside of the scope of this article.        

# A toy script to go to your favourite website from the terminal

So, there's this python library called `webbrowser`.        
Using that library you can write a small script like this.      

```python
import webbrowser as net
import argparse

parser = argparse.ArgumentParser(description="opening websites from terminal")
parser.add_argument('--goto', type=str, help="openthiswebsite")

args = parser.parse_args()
if args.goto:
    net.open(f'https://{args.goto}')

```

And you're done.        
For fun, you can also write this in a bash script and make it executable so that you can run this without the python keyword.       
Can you figure out how you'd run the script?        

Hint: 
```bash
python script.py ... ...
```
What could those two places be?     


## Making it feel like a real command (no python needed!)

### Save the file as goto.py (or whatever name you like).
First, add this special line as the very first line of your script (before any import):
```python
#!/usr/bin/env python3
import webbrowser as net
import argparse

parser = argparse.ArgumentParser(description="Open websites from your terminal 🚀")
parser.add_argument('--goto', type=str, help="The website to open (example: google.com)")
args = parser.parse_args()

if args.goto:
    net.open(f'https://{args.goto}')
else:
    print("Please use --goto <website>")
```
This magic line (#!/usr/bin/env python3) tells your computer: “Hey, run me with Python 3!”      

### Now make the file executable (only needed once):
```bash
chmod +x goto.py
```     
Try it:
```bash
./goto.py --goto youtube.com
```
Nice! But typing ./goto.py every time is annoying if the file is not in your current folder.        

### Level up: run it from anywhere by adding it to your PATH

Create a personal `bin` folder if you don't have one:
```bash
mkdir -p ~/bin
```

Move your script there and drop the `.py` extension:
```bash
mv goto.py ~/bin/goto
```

Add `~/bin` to your PATH. At the end of your shell config:
```bash
export PATH="$HOME/bin:$PATH"
```
- bash → `~/.bashrc`
- zsh → `~/.zshrc`

Reload:
```bash
source ~/.bashrc   # or source ~/.zshrc
```

Now try:
```bash
goto --goto x.com
goto --help
```

There we have it.       


# References
[official python argparse doc](https://docs.python.org/3/library/argparse.html)
[sys.argv python official doc](https://docs.python.org/3/library/sys.html#sys.argv)
[howto guide official python](https://docs.python.org/3/howto/argparse.html)



Thank you for reading.       

~Aayushya Tiwari    