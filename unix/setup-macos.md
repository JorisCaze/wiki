# Setup Mac OS

## Order files/folders everywhere

In case you want to remove the floating ordering, you have to do the following:

- Open a finder window
- Select a regular file/folder
- Press *Command-J*
- In the *Sort By drop-down* control, select *Name*. 
- Click the *Use as defaults* button at the bottom of the window.

Reference: [StackExchange question](https://apple.stackexchange.com/questions/22717/how-to-set-arrange-by-for-all-the-folders-in-the-finder)

## Fix default font for Gnuplot

If you have the following message when using Gnuplot:

```
qt.qpa.fonts: Populating font family aliases took 175 ms. Replace uses of missing font family "Sans" with one that exists to avoid this cost.
```
you can set the default font by adding this line into the *.zshrc* file:

```
export GNUTERM="qt font \"Helvetica\""
```

Reference: [StackOverflow forum](https://stackoverflow.com/questions/65457649/changing-font-when-plotting-with-gnuplot)