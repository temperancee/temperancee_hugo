+++
date = '2025-07-08T22:23:51+01:00'
draft = false
title = 'Micropython X Neovim'
+++

As you may know, my editor of choice is Neovim, the finest editor in the game. This post teaches you how to get a barebones environment setup for working on Micropython projects with Neovim. This won't cover setting up a proper Neovim environment, just how your can take an already well-developed environment and tailor it for Micropython development.

## Prerequistes

You should have:
- A Neovim setup
- An understanding of LSPs in Neovim
- An understanding of Python virtual environments
- Basic familiarity with your shell (since this is a Neovim article, you probably know more than enough)

## Neovim LSP 

There are many LSPs for Python, some of which you can read about by opening Neovim and running `:h lspconfig-all`. I use Pyright, which works pretty well. 


If you use Mason, you can download it with that, or if you're on Arch like me, a simple
```bash
sudo pacman -S pyright
```
will suffice.

### A sad note on Micropython import errors

Micropython is mostly identical to the standard CPython implementation, just with some extra libraries added for working with microcontrollers, e.g. `machine`.
These libraries exist on the device running Micropython, and so code that imports them runs fine.
However, when importing these libraries, we get errors in our editor, because these libraries don't exist on our PC/Laptop, so our editor cannot see them.

As far as I'm aware (backed up by [this post](https://stackoverflow.com/questions/62548091/why-cant-vscode-load-micropython-machine)) that the only way to get around this would be to create libraries that implement all the classes, methods, etc. of the Micropython libraries on our local machine. As referenced in that post, you can see an example of this idea in this [Github repository for the ESP32](https://github.com/tflander/esp32-machine-emulator). I haven't used this repository myself, but it seems like the project was unfortunately abandoned

My current """solution""" to this issue is simply ignoring the errors. While the code runs fine, this is less than ideal. Hopefully I'll find a way around this some day.

## Micropython without Thonny/VSCode

Almost every Micropython tutorial you will see will simply tell you to install Thonny to upload code - if you're lucky, they might show you how to use VSCode (although I've only seen this for RP2040 based boards). If you search for long enough, however, you may be lucky enough to learn about `rshell`.

`rshell` is a command line utility that allows you to run commands on your board running Micropython. For full details, see its [github page](https://github.com/dhylands/rshell). In this section, I'm just going to cover uploading and running files.

`rshell` is installed through `pip`, which means you'll likely have to set up a `venv` to use it:
```bash
python3 -m venv your-venv-name
source your-venv-name/bin/activate
pip install rshell
```
The second and third commands may differ depending on your operating system. See [the docs](https://docs.python.org/3/library/venv.html) for more details.

### Uploading Code

Micropython only runs programs on start-up when you copy them onto the board and name them `main.py`. To accomplish this, open a terminal, make sure `rshell` is installed (and, if necessary, make sure your `venv` is active), then run
```bash
rshell -p /dev/ttyACM0
```
which will put you into `rshell`. Your board's serial port might differ from `/dev/ttyACM0`, in which case, change it accordingly.

From here, we need to copy over our program to the board. Files on the board are stored under `/pyboard/`, and you can see files on your system with `ls`, just like in a normal shell. Bringing this together:
```bash
cp my_program.py /pyboard/main.py
```
And just like that, your program is on your board! Reset your board and it'll start running.

### Using the REPL for debugging

I am not particularly well versed in the art of debugging tools - I'm still at the stage where print statements here and there typically get the job done. As such, this section is pretty much just dedicated to allowing you to see the printed output of your programs. 

We once again make use of `rshell`. The trick here is to give your program an explicit main function. In Python, that is done via 
``` python
def main():
    # Your main function (doesn't actually have to be called main)

if __name__ == '__main__':
    main()
```

This little snippet makes it so that `main()` is run when you run your program. The benefit of this is that if someone imports your code, their namespace is not polluted with all of your global variables (which you can instead put inside main(), although, you needn't put all your globals in there). There may also be other benefits, but I couldn't find the PEP that outlines why it's good to do this...

Anyway, for our purposes, the reason we do this is that it allows us to copy our code to `/pyboard/main.py`, open up the `rshell` REPL, and do the following:

```
>> from main import *
>> main()
```

which will run our main function, and thus, the whole python script, directing ouputs into the REPL for us to see!

You can also use this to run individual functions in your program, and see their output, which can similarly be helpful for debugging.
