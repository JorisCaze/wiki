# Git cheatsheet

## Day to day use

### Create & clone

* Create new repo: `git init`
* Clone repo: `git clone username@host:/path/to/dir`

### Add & remove

* Add changes to *INDEX*: `git add <filename>`
* Add all changes to *INDEX*: `git add -A` for all files in the entire repo. To add all files in the current directory and sub-dir `git add .` (same effect as `git add -A` if located at the root of repo).
* Remove files from *INDEX*: `git rm <filename>`
* To see changes between current file and the one in the *HEAD*: `git diff <filename>`

### Commit & Synchronize

* Commit changes: `git commit -m "Short commit message"`
* Push changes to `remote` repository: `git push origin master`
* Connect local repo to remote repo: `git remote add origin <server>`
* Update local repo with remot changes: `git pull`
* Get last update without syncing it into your branch: `git fetch`
Thus, `git pull` = `git fetch` + `git merge`

### Branches

* Create new branch and switch to it: `git checkout -b <branch>`
* Switch to a specific `<branch>`: `git checkout <branch>` 
* List local branches: `git branch`
* List all branches (remote included): `git branch -a`
* Delete branch: `git branch -d <branch>` (`-D` to delete branch regardless of its push and merge status **be careful**)
* Push branch to remote repo: `git push origin <branch>`

### Merge

* Merge changes from **another** branch: `git merge <branch>`
* See changes between branches: `git diff <source_branch> <target_branch>`

### Workflow

* Working directory: holds actual files
* *Index*: acts as a staging area
* *HEAD*: points to the last commit

It gives:

  Working dir. -> Index -> HEAD

### Log & status

Log:
* Show all previous commit:  `git log`
* See commit of one specific author: `git log --author=bob` (thanks *Bob*)
* Compressed log with one line commit display: `git log--pretty=oneline`
* ASCII art of all branches and commits: `git log --graph --oneline --decorate --all`
* More info: `git log --help`

Status:
* Show current git state: `git status`.
Gives information on current branch, modified files in working directory, state of the staging area.



## Usefull hints

### Uncommited changes in a file are visible in all branches?

I checked out from master branch to branch feature-foo and made some uncommitted changes in a file foo.py. When I switch back to branch master, I am surprised to find that changes in foo.py are visible in master branch.

I have also thought that changes in a branch belongs to that branch, but it seems that I am terribly wrong. The truth is: until you commit your changes in a certain branch, it will be visible in other branches too.

**Reference:** [Stack Overflow](https://stackoverflow.com/questions/5531362/why-git-keeps-showing-my-changes-when-i-switch-branches-modified-added-deleted)-



## Git config

### Set your profile

```sh
$ git config --global user.name "yourNameHere"
$ git config --global user.email "yourEmail"
```

The flag `global` is used to set your global gitconfig (above system and current repo).
If you just to set it for this repo use `git config` without the flag. For the system set `--system` (usefull?...).

### Get your profile info

```sh
$ git config --get user.name
$ git config --get user.email
```

Global config is in your `$HOME` namely `~/.$ gitconfig`.

### Reset author info after commit (no push yet)
If you commited something with the wrong author info because you forgot to set your $ git config initially you can change it if you have not push yet :

```sh
$ git commit --amend --reset-author
```

### Always show log on just one line per commit
Edit the *gitconfig* with: `git config --global format.pretty oneline`

### Colorful git output
If not default add it with: `git config --global color.ui true`



## Quick fixes

### Changing git commit message after push (no pull done yet)

```sh
$ git commit --amend
$ git push --force-with-lease <repository> <branch>
```

### Erase your uncommitted changes

In case you want to restore a file to his previous *HEAD* version: `git checkout -- <filename>`.
If you want to restore all the working directory to *HEAD* state, meaning throw away any uncommitted changes: `git checkout -- .`.

### Drop local changes

If you want to drop all your local changes and commits, fetch the latest history from the server and point your local master branch at it like this:

```sh 
$ git fetch 
$ git reset --hard origin/master
```

Be aware that the use `--hard` of the flag is dangerous because all the things that you have done in that commit will be **GONE**!

### Revert latest commit

Let's say you messed up the last commit and you want to erase it from log but keep the modified content in the working directory:

```sh
$ git reset --soft HEAD^
```

Basically, it just says to go back to the previous commit *HEAD^* and the `--soft` flag allows you to keep the content as local changes.
If you don't want to keep local changes just use `--hard` flag, but again it can be **dangerous**.

**NOTE:** If you want to restore all the working directory to *HEAD* state, meaning throw away any uncommitted changes, we introduced `git checkout -- .` command but the same effect is obtained with `git reset --hard HEAD`. It means that the current state will be reversed to *HEAD* which is the previous commit. 
Be careful you will lose all files which are not committed, use it only when you did some crappy things on your working directory and you want to delete everything to go back to previous state. This command can be also use to go back to another commit state by changing `HEAD` to some commit id but it is not advised to use it because it rewrites history (dangerous if someone has pulled etc).

**References:**
* https://alexgirard.com/git-book/intermediaire/repair-reset-checkout-revert/
* https://koukia.ca/how-to-revert-latest-commits-in-git-fd86a1b2944a

### Undo a *public* change

If you already pushed some changes to remote and you'd like to undo that commit you can use `git revert`. 
Let's say you have 3 commits:

```sh 
$ git log
  d88a435cd58da6d9915cc92fc7d63d1e246f8c78 (HEAD -> master) commit 3
  218a3380bf915229462d0bfaa62c72962824c8c0 commit 2
  1ff0b47f4395e3dce29fab881b11df61d6fa9207 commit 1
```

Now you want to come back to *commit 2* because *commit 3* was really shitty, the cleanest way to do it is to keep history of *commit 3* and continue with *commit 2*. 
Alright, we use `<SHA>` of *commit 2* and the `--no-commit` flag lets git revert all the commits at once- otherwise you'll be prompted for a message for each commit in the range, littering your history with unnecessary new commits :

```sh
$ git revert --no-commit 218a3380bf915229462d0bfaa62c72962824c8c0..HEAD
$ git commit
```

Again if you check your log:

```sh
$ git log
5ad2132fdc5fc7e101cd3e61e7ba40ce0aabde14 (HEAD -> master) Revert "commit 3"
d88a435cd58da6d9915cc92fc7d63d1e246f8c78 commit 3
218a3380bf915229462d0bfaa62c72962824c8c0 commit 2
1ff0b47f4395e3dce29fab881b11df61d6fa9207 commit 1
```

It can also be used when commits are already pushed, thus this one of the safest and cleanest way to revert a commit.

**Reference:** https://stackoverflow.com/questions/4114095/how-do-i-revert-a-git-repository-to-a-previous-commit (Yarin's answer)

### Display git branch in bash prompt

Add this line to your `~/.bashrc`:

```sh
export PS1="\\w:\$(git branch 2>/dev/null | grep '^*' | colrm 1 2)\$ "
```

Sum up:
`git branch` lists your local branches, like that:

```sh
master
* branch_A
branch_B
```

Since the star indicates your current branch, let's grab it with `grep '^*'`. The `^` is used to escape the star symbol (*joker* in bash).
We only need to make the text pretty by removing character from column 1 to column 2 with `colrm 1 2`. Just remind that `colrm` keep the text after the column number specified if only one number is given otherwise if you give two numbers it will remove the content between those numbers.

**Reference:** https://gist.github.com/justintv/168835

### Full integration of git in your prompt

If you want a better integration of git into your prompt with status in it etc it might be better to use an official git tool available on the [GitHub repo](https://github.com/git/git/tree/master/contrib/completion "Git prompt support") of `git/contrib/completion`. Shawn O. Pearce developped one tool called `git-prompt.sh` wich introduces better integration of Git into your prompt and there is also one for git completion if wanted.



## gitignore file

### Most important use cases

First things first, how can we exclude every target folder created by Bob in every sub-module?

The answer is very easy: target/
This will match any directory (but not files, hence the trailing /) in any subdirectory relative to the .gitignore file. This means we don’t even need any * at all.

Here is an overview of the most relevant patterns:
| .gitignore entry | Ignores every |
| ---------------- | -------------- |
|target/   | folder (due to the trailing /) recursively |
| target   | file or folder named target recursively |
| /target  | file or folder named target in the top-most directory (due to the leading /) |
| /target/ | folder named target in the top-most directory (leading and trailing /) |
| \*.class | every file or folder ending with .class recursively |

### Advanced use cases

For more complicated use cases refer to the following table:

| .gitignore entry | Ignores every |
| ---------------- | ------------- |
|\#comment         | nothing, this is a comment (first character is a \#) |
|\#comment         | every file or folder with name #comment (\ for escaping) |
|target/logs/      | every folder named logs which is a subdirectory of a folder named target |
|target/\*/logs/   | every folder named logs two levels under a folder named target (* doesn’t include /) |
|target/\*\*/logs/ | every folder named logs somewhere under a folder named target (** includes /) |
|\*.py[co]         | file or folder ending in .pyc or .pyo. However, it doesn’t match .py! |
|!README.md        | Doesn’t ignore any README.md file even if it matches an exclude pattern, e.g. *.md. NOTE This does not work if the file is located within a ignored folder. |

**Reference:** https://labs.consol.de/development/git/2017/02/22/gitignore.html