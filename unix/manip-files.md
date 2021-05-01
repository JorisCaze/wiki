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