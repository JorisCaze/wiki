# Reset commit

Assuming that you made a *buff commit* pushed to the remote and then after that you made a local commit `Commit B`:

```console
$ git log
fbde9fd   (HEAD -> master)  Commit B
70aaed5   (origin/master)   Buff commit
ccf2779   Commit A
8c706b5   Older commit
```

Now you see that you got an useless commit `Buff commit` in your history that you want to merge to `Commit B`. 
You can either use `git rebase` or `git commit --amend` (if you did not made the `Commit B` already) but today we will use `reset`:

```console
$ git reset --soft HEAD~2
```

With this command you will reset your `HEAD` to `Commit A` and you will have in your working directory contents of `Commit B` (and before):

```console
$ git log
ccf2779   (HEAD -> master) Commit A
8c706b5   Older commit
$ git status
On branch master
Changes not staged for commit:
 (use "git add <file>..." to update what will be committed)
 (use "git checkout -- <file>..." to discard changes in working directory)

    modified: file1.cpp
    modified: file1.hpp

no changes added to commit (use "git add" and/or "git commit -a")
```

Just add all the files and commit:

```console
$ git add .
[...]
$ git commit -m "Commit B"
$ git log
734713b   (HEAD -> master)  Commit B
ccf2779   Commit A
8c706b5   Older commit
$ git push --force-with-lease
[...]
```

Note to force the push of the new history to the remote.

To sum up, with `git reset [commit]` you will undo all the commits after the specified commit and preserves the changes locally.

**Remark**:
`git reset` can also be used on a file (`git reset [file]`) to unstage the file and preserve its content.
