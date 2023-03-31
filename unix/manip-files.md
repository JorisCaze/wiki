# Manipulation of files

## Delete recursively 

1. List all the files concerned

```sh
    $ find . -type f -name "*.txt" -exec ls {} \;
```

2. Delete

```sh
    $ find . -type f -name "criteria-to-find" -exec rm -f {} \;
```


## Change rights to execute recursively

```sh
    $ find . -type f -name "*.sh" -exec chmod +x {} \;
```

## Convert Windows end of line (eof) to unix one

`dos2unix` should be available with your pkg manager (*apt-get* will do the trick).

```sh
    $ dos2unix <filename>
```

Otherwise you can also use `sed`:

```sh
    $ sed -i 's/\r//g' <filename>
```

**Reference:** https://stackoverflow.com/questions/3891076/how-to-convert-windows-end-of-line-in-unix-end-of-line-cr-lf-to-lf

## Delete large directory efficiently

Removing a folder containing thousands of tiny files can take a lot of time with usual `rm` command. 
A fast and simple alternative is to use `rsync` with an empty directory on the sending side and the directory to remove on the receiving side.
The following tells `rsync` to delete extraneous files from the receiving side (ones that are not on the sending side, hence the use of an empty directory):

```sh
mkdir empty_dir
rsync -a --delete empty_dir/ yourdirectory/
```

**Reference:** https://unix.stackexchange.com/questions/37329/efficiently-delete-large-directory-containing-thousands-of-files

## Exclude a folder during a copy operation

- With `rsync`:
```sh
rsync -av --progress sourcefolder /destinationfolder --exclude thefoldertoexclude
```

- Using bash *globbing* (extended file patterns):
```sh
shopt -s extglob
echo images/!(*.jpg)
```

- Bash auto-completion:
Type `cp *`, then hit `Ctrl+X+*`

- Zsh auto-completion:
Type `cp *`, then hit `Tab`

**Reference:** https://stackoverflow.com/questions/4585929/how-to-use-cp-command-to-exclude-a-specific-directory