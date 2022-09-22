# Fix Spotlight huge indexing data

Spotlight allows you to easily find apps and files stored on your computer.
To perform this operation Spotlight must store information about the files location, this task is known as indexing.
Even though this tool might be useful in daily use, its use on some folders that store huge amount of files can create a big index folder and bloat the system. 
It is recommended to disable its use on directories that might contain a large amount of files for which indexing is not useful/relevant (for example, simulation files).

Directories can be excluded in *System preferences -> Spotlight -> Privacy*. 

To delete the previous index run:

```
sudo mdutil -Ea
```

The new one will be created automatically.