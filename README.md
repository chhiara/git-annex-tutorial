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
## set-up git-annex repo on a local machine
First, git-annex has to be installed in a local machine. 
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
To add the files to the git-annex repository simply copy file in the directory that contains the git annex repository.
```shell
$ cd ~/data-annex
$ cp ../datasets/brainptm_data_for_optmization/brain_mask_reg_FMRIB58_FA_1mm.nii.gz .
```
Similarly to git, to track a file in the repository it is necessary to add the file,  and commit the change. Let's note that the command to add the file is "git annex add <file>", that differs from the git command "git add <file>". The commit command is instead the same. 

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
```

The files can be renamed, deleted and moved with the usual git commands. Once the modifications are committed the corresponding symlinks are updated so that the files can be retrieved without errors.
Example:
 
```shell  
git mv train/ test/
git commit -m "renamed folder"
```
## using ssh remotes
Enter on your remote machine using ssh:
```shell  
ssh user@<remote-ip>
```
  
Clone the repository from the local to the remote host. Let's assume the remote machine on which you are cloning the repository is hosted by your instituition and it ussually called UniServer. You may want to add this name in the description of the repository in the init command, to easily indentify it in the future: 
 
```shell  
git clone user@<local-ip>:trial-annex trial-annex/
git annex init "remote clusterone gitannex"
```
 
Then, it is useful to add the repository on your local machine as git remote. Let's call your local machine mydesktop
```shell  
git remote add my-desktop user@<local-ip>:trial-annex/
```
 
The same should be done in the reository on yout local machine: Clusterone has to be added as git remote.
```shell  
git remote add clusterone user@<remote-ip>:/home/user/trial-annex/
```
It is possible that you type incorrectly the path to your remotes in the previous commands. If this happens, it is necessary only to remove the remote and re-define it. To remove the remote:
```shell  
git remote remove <name-remote>
```

"Notice that these repos are set up as remotes of one another. This lets either get annexed files from the other. You'll often want to do that even when you are using git in a more centralized fashion."
 
Now in the repository in clusterone the actual files are not stored. Each file is a broken symlink that points to.. nothing. 
To get the actual contents of the repository it is necessary to synchronize the content on clusterone with the local repository. So on clusterone let's type:
```shell  
git annex sync my-desktop
```  
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







