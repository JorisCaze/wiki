# How to squash commits

If you would like to squeeze some commits together to have a clean history you can `squash` commits.
Assuming you made several commits for nothing and you get the following:

```console
$ git log

fbde9fd   Add Readme
70aaed5   Update Readme
ccf2779   Update Readme again
8c706b5   Update Readme again and again
```

Now let's say you would like to concatenate all these commits into a single one called `Add Readme`, you can use `rebase` command:

```console
$ git rebase -i HEAD~3
```

Here it is requested to start sorting from third commit before the `HEAD`.
One could also speficy the `sha1` directly as:

```console
$ git rebase -i fbde9fd
```

Anyway after that we will get interactive rebase view:

```console
pick  fbde9fd   Add Readme.md
pick  70aaed5   Update Readme
pick  ccf2779   Update Readme again
pick  8c706b5   Update Readme again and again
```

Set `pick` to `squash` for commits that you want to concatenate and keep the `pick` keyword for the first one you want to keep.
This will tell to git to keep first `pick` commit and concatenate the ones with `squash` into it.
Save it and now the only thing left to do is to push the new history to the remote:

```console
$ git push origin master --force-with-lease
```

Be aware to use this trick only when working alone on your branch to avoid to messed up git history if someone else has pulled your code.

Reference: https://www.ekino.com/articles/comment-squasher-efficacement-ses-commits-avec-git