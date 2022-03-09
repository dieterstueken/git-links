# GIT symlins with and without LFS under Linux and Windows

GIT _is_ able to handle [symlinks](https://stackoverflow.com/questions/954560/how-does-git-handle-symbolic-links#answer-18791647).

There is also a popular [GIT for Windows](https://gitforwindows.org/) implementation.

Unfortunately Windows does not support UNIX like symlinks out of the box and GIT for Windows [does not enable them by default](https://github.com/git-for-windows/git/wiki/Symbolic-Links).
But even with disabled symlink support GIT for Windows handles UNIX symlinks consistently (even if they are not really usable if disabled).

## lfs

To handle large binary files, GIT comes with [git-lfs](https://github.com/git-lfs/git-lfs/blob/main/README.md) to store those files differently.

I accidentally tracked symlinks as lfs-files and did not notice it, as it simply worked fine.

Using GIT for Windows, however, I noticed some inconsistency in handling symlinks.
Thus, I created this small sample repository with different kinds of symlinks to track down the problem.

## observations

On my **Linux** system I created two directories containing symlinks to this README.md.
The directory `lfs-files/**` was tracked with LFS while `plain-files` contains untracked files or symlinks.

In addition I added a regular binary file `git.png` to lfs-files to undestand the usual case.

After a commit I get:

```
$ git ls-files --stage
100644 f01f3cad0f787ae23502872b5629382f703b5c82 0       .gitattributes
100644 484cfdf3738baedadeecb73bd45aa14ebeff89a5 0       README.md
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

All symlinks (mode 120000), however, are handled the same and even refer to the same blob containing the link target `../README.md`.
