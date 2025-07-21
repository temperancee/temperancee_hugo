+++
date = '2025-07-08T22:23:51+01:00'
draft = false
title = 'Micropython X Neovim'
+++

This post teaches you how to get a barebones environment setup for working on Micropython projects with Neovim. This won't cover setting up a proper Neovim environment, just how your can take an already well-developed environment and tailor it for Micropython development.

## Prerequistes

You should have:
- A Neovim setup, I recommend TJ DeVries' "[Advent of Neovim](https://www.youtube.com/playlist?list=PLep05UYkc6wTyBe7kPjQFWVXTlhKeQejM)" series for getting set up
- An understanding of LSPs in Neovim, again, see TJ DeVries' [video](https://youtu.be/bTWWFQZqzyI?si=ydv-dEtXykOh_cMh) on this
- An understanding of Python virtual environments, the [docs](https://docs.python.org/3/library/venv.html) are fairly accessible for this topic, or you can watch pretty much any YouTube video about venvs
- Basic familiarity with your shell (since this is a Neovim article, you probably know more than enough)

## Neovim LSP 

There are many LSPs for Python, some of which you can read about by opening Neovim and running `:h lspconfig-all`. I use Pyright, which works pretty well, and seems to be the most popular choice.

If you use Mason, you can download it with that, or if you're on Arch like me, a simple
```bash
sudo pacman -S pyright
```
will suffice. For other operating systems, just use your standard package manager.

You can also install Pyright via Pip if you'd like:
```bash
pip install pyright
```

Now we need to enable Pyright in our `init.lua`, or alternative, a separate Lua file that we `require` in `init.lua`.
```lua
vim.lsp.enable('pyright')
```
Run `:h lspconfig-all` and read the Pyright section for details on configuring the LSP - for most use cases, just enabling Pyright should suffice.

### Micropython import errors, and stubs to the rescue

Micropython is mostly identical to the standard CPython implementation, just with some extra libraries added for working with microcontrollers, e.g. `machine`.
These libraries exist on the device running Micropython, and so code that imports them runs fine.
However, when importing these libraries, we get errors in our editor, because these libraries don't exist on our PC/Laptop, so our editor cannot see them.

This is where stubs come in. Stubs are essentially files that implement all the classes, methods, functions, etc. in a library, but they leave the implementation empty. This allows your LSP to provide information on available functions and their parameters without you having to check the docs. It also gets rid of the import errors, as your system will now be able to see and recognise the Micropython libraries.

Thankfully, there exists [a large repository of stubs for various micropython compatible boards](https://github.com/Josverl/micropython-stubs), so we don't have to create these files ourselves. Getting these stubs to work involves first installing them, then configuring Pyright to see them.

#### Installation

We install the stubs with Pip, so you may have to create a virtual environment to use them (due to global Pip installs being forbidden by default in externally managed (i.e., by a package manager) environments - see [this document](https://packaging.python.org/en/latest/specifications/externally-managed-environments/#externally-managed-environments) for details).
```bash
python3 -m venv your-venv-name
source your-venv-name/bin/activate
```
These commands may differ depending on your operating system, see [the docs](https://docs.python.org/3/library/venv.html) for more details. Once the venv is created, we can either install the stubs directly, or create a `requirements-dev.txt` file. We install the stubs to a folder called `typings`.

```
#requirements-dev.txt
micropython-rp2-rpi_pico_w-stubs==1.25.0.*
```
```bash
pip install -r requirements-dev.txt --target typings
```

For more detail, see the [stubs documentation](https://micropython-stubs.readthedocs.io/en/main/11_install_stubs.html).

#### Configuring Pyright

Pyright is configured using either a `pyproject.toml` or `pyrightconfig.json` file. I use the toml file. There are various settings you can use to configure Pyright, documented [here](https://github.com/microsoft/pyright/blob/main/docs/configuration.md). Here is my `pyproject.toml`:
```toml
[tool.pyright]
stubPath = "typings"
venvPath = "."
venv = "micropython_venv"
typeshedPath = "typings"
typeCheckingMode = "basic"
reportMissingModuleSource = "none"
```


- Firstly, we set the `stubPath`, which tells Pyright where our stubs are stored.
- `venvPath` specifies a path to a directory which *contains* virtual environments. Since my venv is in the same directory as my `pyproject.toml`, I set this to the current directory.
- `venv` is used in conjunction with `venvPath` to specify which venv to use for this project.
- `typeshedPath` is used to override the standard library stubs with the micropython ones. We set it to `"typings"`, which is where our stubs are stored
- `typeCheckingMode` does what it says on the tin. `"basic"` should work for most people.
- `reportMissingModuleSource` is a warning that appears when stubs are detected but their implementation files cannot be found. Since our implementation files are all stored on the device running MicroPython, we need to disable this warning.

All paths are relative to the root of the project. The root of the project is determined based on the `root-markers` option set in your LSP configuration of Pyright. By default, this is `{ "pyproject.toml", "setup.py", "setup.cfg", "requirements.txt", "Pipfile", "pyrightconfig.json", ".git" }`. I personally have mine set to the following:
``` lua
vim.lsp.config('pyright', {
    root-markers = { "pyproject.toml", "pyrightconfig.json" }
})
vim.lsp.enable('pyright')
```
For more details on these settings and more, see both the [stub documentation](https://micropython-stubs.readthedocs.io/en/main/22_vscode.html) and the [Pyright documentation](https://github.com/microsoft/pyright/blob/main/docs/configuration.md).

And that's it! The LSP should now be up and running, and you should be able to see information about imported classes, methods, etc.

## Micropython without Thonny/VSCode

Almost every Micropython tutorial you will see will simply tell you to install Thonny to upload code - if you're lucky, they might show you how to use VSCode (although I've only seen this for RP2040 based boards). If you search for long enough, however, you may be lucky enough to learn about `rshell`.

`rshell` is a command line utility that allows you to run commands on your board running Micropython. For full details, see its [github page](https://github.com/dhylands/rshell). In this section, I'm just going to cover uploading and running files.

`rshell` is installed through `pip`, which means we'll need our venv again.
```bash
source your-venv-name/bin/activate
pip install rshell
```

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
