# git-annex Tutorial
git-annex is a software based on git's infrastructure to manage the versioning and the sharing of the files. While git is intended for small text files (typically text containing code), git-annex is designed for bigger files and, in general, for whole datasets. It eases the data sharing between different users, or between different hardware of the same user. The whole documentation is available on the [official git-annex website](https://git-annex.branchable.com/).

This brief tutorial aims at illustrating the use-of-case of decentralized data sharing, where each machine is connected via ssh.

The tutorial assumes familiarity with:
* git
* Linux shell 
* ssh

It is assumed git is already installed. It is also assumed a ssh connection is already setted between a remote and the local machine.
The shell commands reported in this tutorial were run in Ubuntu20.04, the git-annex version used is 8.20200226, while git version used was 2.25.1 

# decentralized git annex repositories setup
### set-up git-annex repo on a local machine

In all machine we want to use git-annex, the software has to be installed. 
```shell
$ sudo apt-get install git-annex
```

Then, let's create a not bare local repository named _data-annex_ in the home directory :
```shell
$ mkdir ~/data-annex/
$ cd ~/data-annex/
$ git init
Initialized empty Git repository in /home/user/data-annex/.git/
$ git annex init
init  (scanning for unlocked files...)
ok
(recording state in git...)
```

Once the repository is created, a .git folder will be created in the corresponding folder:
```shell
ls -a ~/data-annex
.  ..  .git
```
### add files
To add the files to the git-annex repository simply copy files in the directory that contains the git annex repository.
```shell
$ cd ~/data-annex
$ cp ../datasets/brainptm_data_for_optmization/brain_mask_reg_FMRIB58_FA_1mm.nii.gz .
```

Similarly to git, to track a file in the repository it is necessary to add the file,  and commit the change. Let's note that the command to add files is <code>git annex add <file> </code>, that differs from the git command <code>git add <file> </code>. The <code>git commit<file></code> command is instead the same as the one in git. 

```shell
$ git annex add brain_mask_reg_FMRIB58_FA_1mm.nii.gz 
$ git commit -m "brain mask added"
```

When you add a file to the annex and commit it, only a symlink to the content is committed to git. The content itself is stored in  the folder .git/annex/objects/
In fact:
```shell
$ ls -l
lrwxrwxrwx 1 chiara chiara 202 set  8 11:35 brain_mask_reg_FMRIB58_FA_1mm.nii.gz -> .git/annex/objects/gx/5q/SHA256E-s151748--eb27f1988a7cc914ac1fda978514b17066ef089d45a2da9464e5d42426e9f8f1.nii.gz/SHA256E-s151748--eb27f1988a7cc914ac1fda978514b17066ef089d45a2da9464e5d42426e9f8f1.nii.gz
```
git-annex allows indeed to visualize, navigate and organize the files in tree structure fashion, with directories and subdirectories. The files visualized in this way are, however, only symlink to the actual content of the files, that is stored in the internal directory .git/annex/objects/

The same procedure to add file can be actuated for folders.
```shell
cp -r ~/datasets/brainptm_single_subjs/train/ .
git annex add train/
git commit -m "added train folder"
```
### fixing symlinks to files
The files can be renamed, deleted and moved inside the repository with the usual git commands. After moving files the symlinks may be broken. You can fix them either with the command <code> git-annex fix <file> </code> , or committing the changes. In both cases symlinks are updated so that the files can be retrieved without errors.
Example:

Move the file brain_mask_reg_FMRIB58_FA_1mm.nii.gz inside the repository from /home/chiara/data-annex/ to /home/chiara/data-annex/test/case_2/ and fixing the symlinks with <code>git annex fix</code>: 
```shell  
cd /home/chiara/data-annex/
git mv brain_mask_reg_FMRIB58_FA_1mm.nii.gz  test/case_2/
git annex fix
 fix test/case_2/brain_mask_reg_FMRIB58_FA_1mm.nii.gz ok
(recording state in git...)
```
Move the file brain_mask_reg_FMRIB58_FA_1mm.nii.gz backwards  from /home/chiara/data-annex/test/case_2/ to /home/chiara/data-annex/ and fixing the symlinks committing the changes: 
```shell  
cd /home/chiara/data-annex/test/case_2/
git mv brain_mask_reg_FMRIB58_FA_1mm.nii.gz ./../../
cd ../../../
git annex add brain_mask_reg_FMRIB58_FA_1mm.nii.gz 
git commit -m "moved file "
```
 
## using ssh remotes
Now I' ll describe how to configure a clone of the repository on a remote server.
 
%For clarity, from now on the commands runned on the remote server will be in purple, while the ones runned in the local machine will be in grey. 

```diff
- text in red
+ text in green
! text in orange
# text in gray
@@ text in purple (and bold)@@
```

First, let's enter on your remote machine using ssh:

```shell  
ssh user@<remote-ip>
```
From now on you are connected on the remote machine and the following commands will be runned on this machine.
Let's clone the repository from the local to the remote server. Let's assume the remote machine on which you are cloning the repository is hosted by your University and it is ussually called UniServer. You may want to add this name in the description of the repository in the init command, to easily indentify it in the future: 

```shell  
git clone user@<local-ip>:data-annex data-annex/
git annex init "UniServer repo gitannex"
```
Then, in the UniServer repository it is useful to add the corresponding repository on your local machine as git remote. Let's call your local machine mydesktop.
 
```shell  
git remote add my-desktop user@<local-ip>:data-annex/
```

The same should be done in the reository on yout local machine: UniServer has to be added as git remote. This command is run on the local repository.
```shell  
git remote add uniserver user@<remote-ip>:/home/user/data-annex/
```
It is possible that you type incorrectly the path to your remotes in the previous commands. If this happens, you can easily remove the remote and re-add it with the previous command. To remove the remote:
```shell  
git remote remove <name-remote>
```
You can notice that these repositories are set as being remote of one another. In this way you can get annexed files from one to the other or vice versa: the data are not managed centrally by a single repository. 
 
Now in the repository UniServer the actual files are not stored. Each file is a broken symlink that points to.. nothing. 
To get the actual contents of the repository it is necessary to synchronize the content on clusterone with the local repository. So on clusterone let's type:
```shell  
git annex sync my-desktop
```  
This commands do not trasfer actual data, but only update the metadata. Each time we annex and commit new data on a repository the sync command update the metadata of the new commits. 
Then, we need to retrieve the actual data on the UniServer. To do so we can use the get command:

 
 
 After we add files in clusterone, on the contrary we may need to synchronize the content on your local machine. So it is sufficient to type inside the repo on you local machine:
```shell  
git annex sync clusterone
```  



# Examples of use-of-case:
This type of configuration may be interesting in case we need to store datasets in multiple machines. For example one could have:
* All the data stored in an office's desktop machine in a git-annex repository.
* another corresponding remote git-annex repository in a personal laptop computer. Sometimes, at home, you need to visualize some of the data you have previously generated on the office's desktop. You can easily download (get) some of the data, and visualize them on the laptop. Then you can "drop" the files to not occupy memory on your laptop. 
* a corresponding repository in a Server, containing all the metadata of the whole dataset. Let's assume this is a server with high computational power, but with limited storage capability since it is shared among different users. In some domains, as in the ones that use MRI image data, the images must follow multi-step preprocessing, before being ready for the final high-intensive computation. Hence, often not all of the dataset is needed to be uploaded to the server.  In the git-annex repository, in the server, the data can be stored totally, partially, or not at all, according to the needs. Just before the computation performed by the server is run, specific sets of data can be retrieved with "get". After the computation is ended, space can be freed again dropping the data.

  
References:
 * [https://git-annex.branchable.com/walkthrough/](https://git-annex.branchable.com/walkthrough/)
 * [ https://git-annex.branchable.com/walkthrough/using_ssh_remotes/](https://git-annex.branchable.com/walkthrough/using_ssh_remotes/)







