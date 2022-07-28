# git-annex tutorial 
git-annex is a software that uses git infrastracture to manage the versioning and the sharing of thefiles. While git is intended for small text files (typically text containg code), git-annex is designed for bigger files and, in general, for whole datasets. It ease the  data sharing between different users, or between different hardwares of the same user. The whole documentation is available here (LINK).

This brief tutorial is intended as illustrate the use-of-case of decentralied data sharing, where each machine is connected via ssh. **what is decentralized data sharing**. The tutorial assumes familiarity with:
* git
* linux shell 
* ssh 

The shell commands reported in this tutorial were run in Ubuntu20.04, the git-annex version used is 8.20200226, while git version used was 2.25.1 

## decentralized git annex repositories setup

First, git-annex has to be installed. 
```shell
$ sudo apt-get install git-annex
```

Then, let's create a not bare local repository in the home directory:
```shell
$ mkdir ~/trial-annex
$ cd ~/trial-annex
$ git init
Initialized empty Git repository in /home/chiara/repo-annex/.git/
$ git annex init
init  (scanning for unlocked files...)
ok
(recording state in git...)
```

Once the repository is created, a .git folder will be created in the corresponding folder:
```shell
ls -a ~/trial-annex
.  ..  .git
```

To add the files to the git-annex repository, simply copy file in the directory that contains the git annex repository
```shell
$ cd ~/trial-annex
$ cp ../datasets/brainptm_data_for_optmization/brain_mask_reg_FMRIB58_FA_1mm.nii.gz .
```
Similarly to git, to track a file in the repository it is necessary to add the file,  and commit the change. Let's note that the command to add the file is "git annex add <file>", that differs from the git command "git annex add <file>". The commit command is instead the same. 

```shell
$ git annex add
$ git commit -a -m "FA file added"
```

When you add a file to the annex and commit it, only a symlink to the content is committed to git. The content itself is stored in .git/annex/objects/
In fact:
```shell
$ ls -l
lrwxrwxrwx  1 chiara chiara  202 lug 16 16:46 brain_mask_reg_FMRIB58_FA_1mm.nii.gz -> .git/annex/objects/gx/5q/SHA256E-s151748--eb27f1988a7cc914ac1fda978514b17066ef089d45a2da9464e5d42426e9f8f1.nii.gz/SHA256E-s151748--eb27f1988a7cc914ac1fda978514b17066ef089d45a2da9464e5d42426e9f8f1.nii.gz
drwxr
```
git-annex allows indeed to visualize, navigate and organize the files in tree structure fashion, with directories and subdirectories. The files visualized in this way are, however, only symlink to the actual content of the files, that is stored in the internal directory .git/annex/objects/

The same procedure to add file can be actuated for folders.
```shell
cp -r ~/datasets/brainptm_single_subjs/train/ .
git annex add train/
git commit -m "added train folder"
```shell



git-annex info






