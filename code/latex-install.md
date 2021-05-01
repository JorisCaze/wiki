How to install LaTeX 
====================

## Unix

### Install:

1. LaTeX editor: TeXStudio 

```sh
    $ sudo apt install texstudio
```

2. LaTeX compiler: TeXLive 

```sh
    $ sudo apt-get install texlive-full
```

Alternatively, you can install `MiKTeX` but the best support is for `TeXLive` see [StackExchange](https://tex.stackexchange.com/questions/164262/installation-of-texstudio-and-miktex-in-ubuntu-12-04-lte).

### Use of packages for encoding and language (fr-version)

#### Input encoding : 
Modern computer systems allow you to input letters of national alphabets directly from the keyboard

```latex
    \usepackage[utf8]{inputenc} 
```

#### Font encoding :
To proper LaTeX document generation you must also choose a font encoding which has to support specific characters for French language

```latex
    \usepackage[T1]{fontenc}
```

#### Language specific pkg and commands :
To extended the default LaTeX capabilities, for proper hyphenation and translating the names of the document elements (such as *table of contents* --> *bibliography*), import the babel package for the French language.

```latex
    \usepackage[francais]{babel}
```

You can also only import babel pkg without option and add a global option *french* to the document class: 

```latex
    \documentclass[french]{article}
     
    %encoding
    %--------------------------------------
    \usepackage[utf8]{inputenc}
    \usepackage[T1]{fontenc}
    %--------------------------------------
     
    %French-specific commands
    %--------------------------------------
    \usepackage{babel}
```

Reference: https://fr.overleaf.com/learn/latex/French



## Windows

### LaTeX Compiler

Again you have the choice between [TeX Live](https://www.tug.org/texlive/) and [MiKTeX](https://miktex.org/).

There is also an easy-to-install distribution for Windows, based on MiKTeX, called [proTeXt](http://www.tug.org/protext/). 
proTeXt is a tool which allows you to install easily *MiKTeX*, *Ghostscript*, *GSview* and *TeX Studio* (even though the version of TeXStudio is old). 
Last time on Windows I installed succesfully *MiKTeX* with this tool and *TeXStudio* separately (from official website).

**Note:** it can be useful to check if you LaTeX install is in your `PATH` or `echo %PATH%` (to have a nice display of your paths use `echo %path:;=&echo.%`), hit *ctrl+r* and type *cmd.exe* to open command line. Once done, just type `path` and check if your TeX install is in it. 

### LaTeX editor

#### With TeXStudio editor

With `Chocolatey`: 

```console
    choco install texstudio
```

Otherwise from official website: [TeX Studio](https://www.texstudio.org/).

#### Sublime Text editor

1. Install the LaTeXTools sublime text package

Now, you should install the LaTeXTools package, which provides a lot of functionalities for writing and compiling LaTeX files. If you have installed package control, you can install this package using package control.

**Note:** 

* Install package control
    * Open the command palette with `ctrl+shift+p` or go to *Tools -> Command palette*
    * Type Install Package Control, press enter

* Install a package:
    * Open the command palette with `ctrl+shift+p` or go to *Tools -> Command palette*
    * Type *Package Control: Install Package*, press enter
    * Type the name of your package

2. Install Sumatra PDF reader

To preview the produced PDF files, I recommend using the well-known light-weight [Sumatra PDF](https://www.sumatrapdfreader.org/free-pdf-reader.html) reader (`chocoley` install also possible). After install, we should also set its path properly.

3. Check your system

After all these installations and settings, we can use the built-in function of LaTexTools to check our system. 
To check if there are any issues, use *Check System* of LaTeXTools package: *Preferences -> Package settings -> LaTeXTools -> Check system*.

4. Write and compile LaTeX source files

Use the shortcut key *Ctrl+B* to compile the file. You may be prompted to choose a builder. Just choose the basic builder.

5. Other packages to install

There are a few packages designed for use with LaTeXTools, for example, [LaTex-cwl](https://github.com/LaTeXing/LaTeX-cwl) and [LaTeXYZ](https://github.com/randy3k/LaTeXYZ). You should install them to enhance your experience of writing LaTeX in Sublime Text.

6. Configure inverse search for PDF files

Inverse search is a handy feature. First, compile any tex files and open it with Sumatra PDF reader. Then go to Settings -> Options. In the inverse search part, use the following setting:

    "C:\Program Files\Sublime Text 3\sublime_text.exe" "%f:%l"

In the above setting, change the directory of Sublime Text executable to its actual location on your system.

**Reference:** beautifully copy-pasted from https://jdhao.github.io/2018/03/10/sublime-text-latextools-setup/