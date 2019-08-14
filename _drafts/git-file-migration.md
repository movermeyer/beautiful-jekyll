---
layout: post
title:  "Migrating file(s) between Git repos"
date:   2019-08-14 00:00:00 -0000
categories: git
---

In some scenarios, you might want to extract files from one Git repo and insert them into another repo.
Often this happens because you have some code that was originally part of a monolithic project and you are refactoring to pull functionality into a shared library.

You want migrate the files to a new repo, but you don't want to lose the important Git history of the files:

  * Who contributed to the files?
  * When were changes made?
  * Why were changes made?
  * What bugs did you encounter, and how did you fix them?

This guide will walks through exactly how to do this.

# Important notes

There are a lot of conflicting commands that exist on the internet for how to migrate files between Git repos.
However, using `git filter-branch` is the only way to do this without losing either:

  * some of the commits
  * some aspect of the commit (author, message, date, etc.)

This guide describes the way to do this when you want to preserve everything about the commits related to the files.

Namely it keeps:

  * Every commit that made changes to the file
  * The author, message, date, etc. of those commits

Things that it doesn't keep:

  * Information about the other files that were also modified in those commits.

# Warning!

  1. This process is **very** destructive to the git history and git index. Only perform this on a fresh clone of the repo that  you    want to extract the files from
  2. These commands were written for bash with MacOS versions of the utilities. GNU version of the utilities have different flags.

# Process

Given a source repo that you want to extract files from (`repoA`), and a repo that you want to add the files and their histories to (`repoB`), first start by getting a fresh clone of `repoA`:

```bash
git clone repoA
cd repoA
git remote rm origin # Optional: Prevents any chance of accidentally pushing the changes this is going to make
```

The next step is to run `git filter-branch` to rewrite the history of our local `repoA` clone.

(Note: None of the commands in this guide will affect `repoA` on the remote server. If something gets messed up, it's safe to simply delete the local clone and retry)

`git filter-branch` offers a simple way to reduce a repo to just a subdirectory (using the `--subdirectory-filter`). 
Unfortunately, `git filter-branch` doesn't offer a nice filter for selecting files that are not within the same directory.
But it's not too hard to hack something together for this (credit to `jkeating` and `AdrieanKhisbe` for [figuring it out](https://stackoverflow.com/a/6006679)):

```bash
git filter-branch \
    --prune-empty \
    --index-filter '
      git ls-tree -r --name-only --full-tree $GIT_COMMIT \
      | grep -v "^lib/flatten_hash.rb$" \
      | grep -v "^lib/key_based_file.rb$" \
      | grep -v "^lib/unflatten_hash.rb$" \
      | grep -v "^test/lib/flatten_hash_test.rb$" \
      | grep -v "^test/lib/key_based_file_test.rb$" \
      | grep -v "^test/lib/unflatten_hash_test.rb$" \
      | xargs git rm --cached --ignore-unmatch -r
    ' -- --all
```

What is going on here?

`git filter-branch` has an `index-filter` which allows filtering of the Git file index using a shell command.
In this case, it will run the big Bash command for each commit in the Git history.

In this case, the Bash command says:

For each commit in the Git history:

  1. Run `git ls-tree -r --name-only --full-tree $GIT_COMMIT`, which gives a list of all the files in the repo at that commit (`$GIT_COMMIT` is set by `git filter-branch`)
  2. Use `grep -v` to remove the desired files from this list. In this case, I have 6 files I wanted to extract, so I've added 6  calls to `grep -v`, each one matching one of the file names exactly.
  3. Finally, for each of the remaining filenames, run `git rm --cached --ignore-unmatch -r` to remove all record of the file from the repo at this commit.

The `--all` means that this filtering will be applied to all Git refs (~ branches) in the repo. Technically, this is overkill, as you probably only care about extracting the files from the branch that you are currently on. If that is the case, you can use `HEAD` instead of `-- --all`.

The `--prune-empty` ensures that if the commit no longer has any file changes associated with it (because we removed all the files associated with that commit), then the commit is dropped. This leaves only commits that are related to our desired files. 

This can take a long time. For example, it took ~20 minutes for the my repo.
It is deleting every file (except our desired files) in the repo at every commit, then dropping commits that don't have any files changed in them.

At the end of this, run an `ls` and you can see that only the requested files remain in the repo.

```bash
$ ls -R
lib/:
flatten_hash.rb		key_based_file.rb	unflatten_hash.rb

test/:
lib

test/lib:
flatten_hash_test.rb	key_based_file_test.rb	unflatten_hash_test.rb
```

Things are looking good so far. However, if you were to look closely at the output `git log`, you might see a number of empty merge commits still in the history.

To remove these empty merge commits we can run a second call to `git filter-branch` (credit to `raphinesse` and `James EJ ` for [figuring it out](https://stackoverflow.com/questions/9803294/prune-empty-merge-commits-from-history-in-git-repository#comment70204164_38420284)):

```bash
# Warning:
#   This command uses the MacOS versions of sed and xargs, and not the GNU versions found on many *nix distributions.
#   For the GNU version, see https://stackoverflow.com/a/38420284
git filter-branch -f \
    --prune-empty \
    --parent-filter '
      sed "s/-p //g" \
      | xargs git show-branch --independent \
      | sed "s/^/-p /g" \
      | tr "\n" " "
    ' -- --all
```

This time the command uses the `--parent-filter` of `git filter-branch`. `--parent-filter` allows you to rewrite the parent commits of each commit.

The details of this Bash command are slightly more complex than the previous command.

For each commit in the Git history, Git is supplying the Bash command with references to the parent(s) of each commit in the format:

  * empty string, for the initial commit
  * "-p parent", for a normal commit
  * "-p parent1 -p parent2 -p parent3 …​", for a merge commit

The Bash command then:

  1. Uses `sed` to remove the `-p` portion of any parents
  2. Uses `git show-branch --independent` to removes any parents that can be reached from any other reference
  3. Uses `sed` to re-add the `-p` back in to the remaining parents
  4. Uses `tr` to bring the parent references back onto the same line

With those empty merge commits also removed, it's time for some cleanup of our local Git cache. While these command aren't strictly necessary, doing them can speed things up:

```bash
git reset --hard
git gc --aggressive
git prune
```

Finally, the source repo's history pared down to exactly what is desired. It's time to add the commits to the destination repo.

The first step is to clone the destination repo:

```bash
cd ../
git clone repoB
cd repoB
```

Then add `repoA` as a remote, so that `repoB` can pull the Git history from there:

```bash
git remote add repoA ../repoA
git fetch repoA
```

Now it's time to add the commits from `repoA` to `repoB`'s history. You can get really fancy and complicated here, depending on preference and willingness to rewrite the history of `repoB`. This guide will just do a simple `git merge` though:

```bash
git merge remotes/repoA/master --allow-unrelated-histories # Assuming that you rewrote the history of the `master` branch in repoA
git remote rm repoA
```

This adds all of the commits related to the files of `repoA` to the end of the `repoB`'s history. Nothing fancy.

There you have it. The files have been imported into `repoB`, along with their entire histories. After a `git push`, everyone else will be able to see the new changes

# FAQ

### Could I get fancier with my rewriting of history?

**Absolutely!** You might want to further clean up the Git history of the files that you've extracted before you import them into the destination repo.

For example, if you are migrating files from a proprietary code base to an open source one, it is very good practice to audit the changes made in each commit individually in order to ensure that they don't contain any corporate secrets (especially security keys). While more advanced history editing is beyond the scope of this article, I have found [BFG Repo-Cleaner](https://rtyley.github.io/bfg-repo-cleaner/) to be handy for this.