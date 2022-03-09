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

