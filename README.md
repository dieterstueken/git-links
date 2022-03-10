# GIT Symlins with and without LFS under Linux and Windows

GIT _is_ able to handle [Symlinks](https://stackoverflow.com/questions/954560/how-does-git-handle-symbolic-links#answer-18791647).

There is also a popular [GIT for Windows](https://gitforwindows.org/) implementation.

Unfortunately Windows does not support UNIX like Symlinks out of the box and GIT for Windows [does not support them by default](https://github.com/git-for-windows/git/wiki/Symbolic-Links).

To handle large binary files, GIT comes with [git-lfs](https://github.com/git-lfs/git-lfs/blob/main/README.md) to store those files differently.

I accidentally tracked Symlinks as lfs-files and dod not notice it, as it simply worked fine.

Using GIT for Windows, however, I noticed some inconsistency in handling Symlinks.
Thus, I created this small sample repository with different kinds of Symlinks to track down the problem.

### observations

On my **Linux** system I created two directories containing symlinks to this README.md.
The directory `lfs-files/**` was tracked as LFS while `plain-files` contains untracked symlinks.

In addition I put a regular file `git.png` into lfs-files:

```
.
├── lfs-files
│   ├── git.png
│   ├── README.md -> ../README.md
│   └── README-TOO.md -> ../README.md
├── plain-files

│   ├── README.md -> ../README.md
│   └── README-TOO.md -> ../README.md
└── README.md

2 directories, 6 files
```

After a commit I get:

```
$ git ls-files --stage
100644 f01f3cad0f787ae23502872b5629382f703b5c82 0       .gitattributes
100644 532f6b4b9c69626deb25e91ce17d2ecd621bc524 0       README.md
120000 32d46ee883b58d6a383eed06eb98f33aa6530ded 0       lfs-files/README-TOO.md
120000 32d46ee883b58d6a383eed06eb98f33aa6530ded 0       lfs-files/README.md
100644 5020b83b0ff9d8aade82a5794a4385dc85bb145a 0       lfs-files/git.png
120000 32d46ee883b58d6a383eed06eb98f33aa6530ded 0       plain-files/README-TOO.md
120000 32d46ee883b58d6a383eed06eb98f33aa6530ded 0       plain-files/README.md
```

The binary file `git.png` was saved as an LFS-blob:

```
$ git cat-file blob 5020b83b0ff9d8aade82a5794a4385dc85bb145a
version https://git-lfs.github.com/spec/v1
oid sha256:880c0be216cc652383b1997d319b4fb31410d7303dd83bbea7603e8a3b948617
size 3783
```

All symlinks, however, are handled the same and even refer to the same blob containing the link target `../README.md`.

### GIT for Windows with core.symlinks = false

Checking out the repository with `git version 2.35.1.windows.2` does not create any symlink.
Instead plain files are created containing the target path but `git ls-files` looks exactly the same.
So, the symlinks are not really usable, but still consistent.

To reproduce the inconsitency I observerd, you may simply remove a link and restore it again:

```
$ rm lfs-files/README.md; git restore lfs-files; git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   lfs-files/README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

and:

```
$ git diff
diff --git a/lfs-files/README.md b/lfs-files/README.md
index 32d46ee..ac898e3 120000
--- a/lfs-files/README.md
+++ b/lfs-files/README.md
@@ -1 +1,3 @@
-../README.md
\ No newline at end of file
+version https://git-lfs.github.com/spec/v1
+oid sha256:dbcc210d7f4962499db6d4cfa18658e26d05ee700c962a811cac911f095e22fd
+size 12
```

Seems git expects the LFS pointer here instead of the link path. Looks like a problem with the smudge filter.

If you accidentally push this state and pull it again on Linux, you get a broken ``symlink -> version https://git-lfs.github.com/spec/v1``.

Several other strange observations can be made:

After removing **all** files and restoring them the broken symlink are sanitized again:

```
$ rm lfs-files/*; git restore lfs-files; git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

Even stranger: if you remove and restore the plain file ``git.png``, any inconsitent link gets sanitized again, too.

```
$ rm lfs-files/README.md; git restore lfs-files; git status
... broken ...
$ rm lfs-files/git.png; git restore lfs-files; git status
... fine ...
```

I don't have any clue, what is going on here. I tried with ``GIT_TRACE=1`` but that's too cryptic for me.

### GIT for Windows with core.symlinks = true

To enable symlinks, you have to 
[enable Developer Mode](https://docs.microsoft.com/en-us/windows/apps/get-started/enable-your-device-for-development)
and set ``core.symlinks = true``. 
After that, GIT for Windows behaves linke Linux GIT without problems.

### git-bash with symlinks

Using git-bash I found ``ls -s``
[does not work by default](https://www.joshkel.com/2018/01/18/symlinks-in-windows) either, 
even with Developer Mode enabled.

You have to set: ``export MSYS=winsymlinks:nativestrict`` to enable ``ls -s``.
I added this to my /etc/pofile, but I'm not convinced if this is the ideal place.
