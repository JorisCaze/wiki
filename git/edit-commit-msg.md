# Change a commit message after push

Let's say you made a typo on your last commit or you want to add something but your already pushed it.
You can use the `--amend` flag to edit last commit message locally:

```console
$ git commit --amend -m "New message"
```

Alright it was easy to edit it if the commit was not already pushed to the remote or if you work only locally.
But now, let's say the wrong commit message was already sent to the remote, you have to push again the correct commit message.
This can be done with:

```console
$ git push --force repo-name branch-name
```

However you must be aware that it can messed up your git repo if somebody has pulled this commit. Indeed, there will be two different names for the same commit. History is longer valid.
It is safer to check before pushing if there is an upstream change to the repo. If no one else has the last version, commit message will be updated and if not, push will be aborted.
This can be done with the flag `--force-with-lease`:

```console
$ git push --force-with-lease repo-name branch-name
```

Reference: https://www.educative.io/edpresso/how-to-change-a-git-commit-message-after-a-push