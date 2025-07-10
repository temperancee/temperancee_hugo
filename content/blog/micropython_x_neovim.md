+++
date = '2025-07-08T22:23:51+01:00'
draft = true
title = 'Micropython X Neovim'
+++

As you may know, my editor of choice is Neovim, the finest editor in the game. This post teaches you how to get a barebones environment setup for working on Micropython projects with Neovim.

## Uploading Code

## Debugging

I am not particularly well versed in the art of debugging tools - I'm still at the stage where print statements here and there typically get the job done. As such, this section is pretty much just dedicated to allowing you to see the printed output of your programs. 

We once again make use of `rshell`. The trick here is to give your program an explicit main function. In Python, that is done via 
``` python
def main():
    # Your main function (doesn't actually have to be called main)

if __name__ == '__main__':
    main()
```

This little line makes it so that `main()` is run when you run your program. The benefit of this is that if someone imports your code, their namespace is not polluted with all of your global variables (which you can instead put inside main(), although, you needn't put all your globals in there). There may also be other benefits, but I couldn't find the PEP that outlines why it's good to do this...

Anyway, for our purposes, the reason we do this is that it allows us to copy our code to `/pyboard/main.py`, open up the `rshell` REPL, and do the following:

```
>> from main import *
>> main()
```

which will run our main function, and thus, the whole python script, directing ouputs into the REPL for us to see!
