# Index

- [Introduction](#introduction)
- [Setup](#setup)
- [Our working methodology](#our-working-methodology)
- [Best practice or how you should do it](#best-practices---or-how-you-should-do-it)
- [Extra documentation](#extra-documentation)

## Introduction

### Version control

Version control systems are a category of software tools that help a software team manage changes to source code over time. Version control software keeps track of every modification to the code in a special kind of database. If a mistake is made, developers can turn back the clock and compare earlier versions of the code to help fix the mistake while minimizing disruption to all team members.

### Git

Git is a version control system for tracking changes in computer files and coordinating work on those files among multiple people. It is primarily used for source code management in software development, but it can be used to keep track of changes in any set of files. As a distributed revision control system it is aimed at speed, data integrity, and support for distributed, non-linear workflows. As with most other distributed version control systems, and unlike most client–server systems, every Git directory on every computer is a full-fledged repository with complete history and full version tracking abilities, independent of network access or a central server.

### Github

Github is an open-source web-based repository hosting service for version control using git. It stores your projects written in different languages, makes easier to collaborate in their development, allows you to manage them through their web-based interface and gives you a lot of extra functions with its great integration options.

### git-flow

GitFlow is a branching model for Git, created by Vincent Driessen. It has attracted a lot of attention because it is very well suited to collaboration and scaling the development team.

### Setup

In order to start working with Github and git-flow, you will just need to:

- Create a free account at [Github](https://github.com/) and navigate to our [repositories](https://github.com/BU-ISCIII).
- Install Git in your computer if you already do not have it. You can learn how to do it [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).
- [Install git-flow](https://github.com/nvie/gitflow/wiki/Installation)
- Set up your rsa ssh authentication like explained in this [link](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).
- Configure your ssh client user name <your_name> and email <your_email> as the ones used in our github account typing these lines in your terminal:

`git config user.email "<your_email>"`

`git config user.name "<your_name>"`

Finally, ask an admin to make you a collaborator of the BU-ISCIII repository and you are ready to go.

### File status

Pay attention now — here is the main thing to remember about Git if you want the rest of your learning process to go smoothly. Git has four main states that your files can reside in: committed, modified, and staged:

- Untracked means that the file is not been tracked by git.

- Unmodified means that the data is safely stored in your local database.

- Modified means that you have changed the file but have not committed it to your database yet.

- Staged means that you have marked a modified file in its current version to go into your next commit snapshot.

![git](https://git-scm.com/book/en/v2/images/lifecycle.png)

Each file in your working directory can be in one of two states: tracked or untracked. Tracked files are files that were in the last snapshot; they can be unmodified, modified, or staged. In short, tracked files are files that Git knows about.

Untracked files are everything else — any files in your working directory that were not in your last snapshot and are not in your staging area. When you first clone a repository, all of your files will be tracked and unmodified because Git just checked them out and you haven’t edited anything.

As you edit files, Git sees them as modified, because you’ve changed them since your last commit. As you work, you selectively stage these modified files and then commit all those staged changes, and the cycle repeats.

To sum up: If a particular version of a file is in the Git directory, it’s considered committed and unmodified. If it has been modified and was added to the staging area, it is staged. And if it was changed since it was checked out but has not been staged, it is modified.

### Checking the status of a file

File status can be check with `git status` and looks like this:

```
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   CONTRIBUTING.md
```

File status change when you add, modify or delete a file. In order to reverse changes to previous commit, you can simply `git checkout` the repository. When you are ready to make a new commit, stage the changes you want to include with `git add`. Not all changes have to be staged, only the ones that correspond to that particular commit.

```
$ git add CONTRIBUTING.md
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   README
    modified:   CONTRIBUTING.md
```

### Branching

Nearly every VCS has some form of branching support. Branching means you diverge from the main line of development and continue to do work without messing with that main line. In many VCS tools, this is a somewhat expensive process, often requiring you to create a new copy of your source code directory, which can take a long time for large projects.

Some people refer to Git’s branching model as its “killer feature,” and it certainly sets Git apart in the VCS community. Why is it so special? The way Git branches is incredibly lightweight, making branching operations nearly instantaneous, and switching back and forth between branches generally just as fast. Unlike many other VCSs, Git encourages workflows that branch and merge often, even multiple times in a day. Understanding and mastering this feature gives you a powerful and unique tool and can entirely change the way that you develop.

We will use git's powerful branching model for our software development. As we'll explain in nex section, our projects will contain at least one master and one develop branches, but new branches will be used for features development.

## Our working methodology

### Introduction

The objective of this guide is to make everyone capable of interacting in the same and documented way with the repository. We work on Git following the git-flow branching workflow while managing the repositories through Github web server. There are multiple other ways of achieving the same, but this one excels in making collaboration and scaling the development team as easy as possible.

In case you find yourself working in an scenario which is not considered in this guide, feel free to find your own way out of there, document the best practices and improve the methodology by adding the fix to it. By doing this, not only you and everyone else will know how to react under the same circumstances, but they will do it in the same way, making it very easy to track everyone's work.

### Understanding git-flow branching scheme

[GitFlow](http://nvie.com/posts/a-successful-git-branching-model/) is a branching model for Git, created by Vincent Driessen. We use git-flow as a wrapper around existing git commands which allow us to easily implement this scheme in our daily work. But before explaining how the wrapper works we need to understand the philosophy behind the scheme.

![Summary of how git-flow works](https://github.com/BU-ISCIII/dotfiles/blob/master/images/git-flow.png)

In the previous image you can see a summary of how a repository should be organised following this branching scheme, and how it evolves through time (y axis, from top to bottom). Each parallel vertical line represents different a branch, each bubble a new push to the main repository, and the arrows between the branches show forking (outcoming arrows) and merging (incoming arrows) events. Now let's try to explain step by step what's going on there!

![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/git-flow_1.png)

While a default just-born git repository only contains one branch (master), a git-flow repository always starts with at least two branches (master and develop). You should move straight to develop and start creating your project there: write the early documentation, create the folder structure and the basic code. NO DEVELOPMENT SHOULD EVER BE DONE IN THE MASTER BRANCH!

Once the skeleton of your project is done, it's time to forget about the develop branch and start creating features. It does not matter what you are going to do next, from adding new features to the interface to adding database support, because there should always be a way of splitting it in one or more sub-projects or tasks (feature branches). Stop one second to think what exactly you want to do and how, and then create a new feature branch called whateveryouaregoingtodohere (make sure the name self-explains to the other collaborators what is going on there) for each different thing you want to do.

![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/git-flow_2.png)

When creating a feature branch you are just forking the develop repository, coping it to a new branch where you can experiment in your idea without the risk of damaging anything in the common develop branch. You can also work in different feature branches at the same time, trying different approaches to solve a problem, and test them to finally keep only the best solution. Doing this allow us to work in a modular way and makes very easy to implement in develop each feature only when they are ready to roll, without messing with other feature's development in the meanwhile.

Once our features are completed (completely finished and tested), we can merge them back into the develop branch. After doing this, all the new functionalities implemented in our feature will be integrated in the develop branch and, as there is nothing left to do in this feature and is already a core part of the project, the feature branch can be removed.

![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/git-flow_3.png)

If we consider that our project is finally ready to be released in its actual state in the develop branch, it is time to create a release branch. This branch will pump the version number of the project and fork the develop branch into a brand new release branch. It is time to intensively test it to its limits, and discover any bugs that we could have missed before this point. NO DEVELOPMENT SHOULD BE CARRIED IN A RELEASE BRANCH, only minor bug fixes and last minute tweaks. Every change you make in a release branch will be merged back to develop.

If for any reason you discover major bugs or lack of features, it is time to merge back to develop, remove the release branch and create the corresponding feature branches where you will work on those problems. Once that you consider everything is ready to be released (again), create the release branch from the updated develop branch and test again.

![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/git-flow_4.png)

Finally, you are happy with the project at release state, so you merge the release branch into both master and develop, and remove the release branch. Good work, everyone! The first version of the project is archived as 1.0 in master branch and ready to be downloaded by anyone.

![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/git-flow_5.png)

But it will come the time when someone discovers that your released version was not bug-free at all. And this is the production version of the project, and always have to be ready to be used. And probably develop has evolved since the last time you released that version, so rolling back to that time stamp and going back to the beginning of the workflow would be a pain. No reason to do it, anyway! There is an easy way to fix this!

It is time to create a hotfix branch. These branches are forked not from develop but from master, and from master in the release version you want to. So simply move to your hotfix branch, fix the bug and merge it back to both master (with a minor release bump) and develop (so following versions do not have the bug). And remove the hotfix branch. Problem solved!

As you can see, with the git-flow branching scheme it is very easy to collaborate in the development, document and maintain projects, even if the development team suffers major changes.

### Using Github to manage our repositories

GitHub is a code hosting platform for version control and collaboration. It lets you and others work together on projects from anywhere, and allows you to graphically interact and manage your repositories. Thanks to the multiple integrative options it has, it makes pretty easy to do lots of extra functions on it, like document your projects.

Github allows us to execute lots of Git basic commands apart from exploring your repository, like create a new repo/branch, rename it, remove it, navigate through the branches, add or remove files and even do commits. But the most important function may be the Pull Request. As every public repository in Github is available to the public, anyone can review your code and collaborate to improve it, even if they are not collaborators of the project.

Unluckily, most of these previously mentioned functionalities are not well suited for our group working philosophy. Therefore, we will use it only to create, remotely allocate, document and supervise our repositories, while the development work will happen in our local machines. Faulty release branches which should be removed because of major bugs or lack of key features (in other words, that should never had been created in the first place) or abandoned feature branches can be also deleted from here by an administrator.

To document the project, you can use the Github web editor to write your README file in Mardown, and the wiki pages as this one to elaborate a bit more the usage guide of the software.

### Git in the command line

Git is an example of a distributed version control system (DVCS) commonly used for open source and commercial software development. DVCSs allow full access to every file, branch, and iteration of a project, and allows every user access to a full and self-contained history of all changes. Unlike once popular centralized version control systems, DVCSs like Git don’t need a constant connection to a central repository. Developers can work anywhere and collaborate asynchronously from any time zone.

Without version control, team members are subject to redundant tasks, slower timelines, and multiple copies of a single project. To eliminate unnecessary work, Git and other VCSs give each contributor a unified and consistent view of a project, surfacing work that’s already in progress. Seeing a transparent history of changes, who made them, and how they contribute to the development of a project helps team members stay aligned while working independently.

Of course, you are free to use all the git commands if you are already used to instead using the git-flow wrapper and Github managing tools, but please keep in mind that more people will work in the project and so, the same workflow and branching scheme must be used by everyone.

Learning how to properly use git in the command line would also be great at long term for an experimented user, as it will give the user a deeper control of what they are doing and how to fix future issues which may not be already considered in this guide.

### Combining all the previous in one workflow

Git will be our main tool which both git-flow and Github depends from. It is used in the command line, and has lots and lost of different options and functionalities. But to make it easier to get hands on work, we will use it only for creating our local copies of the repositories (git copy), change between branches when working (git checkout), create (git add ./) move (git mv <fromfile tofile>) and remove (git rm <file>) files, create commits (git commit -m "<message>") and update both the local (git pull) and remote (git push) repositories with the newest changes.

Repository creation and documentation will be handled from Github, and all the branching scheme will be operated by git-flow wrapper. For details of how to do each action see the next page of this wiki: [Best practices - or how you should do it](https://github.com/BU-ISCIII/dotfiles/wiki/Best-practices---or-how-you-should-do-it)

# Best practices - or how you should do it

## Creating a new project

If you are starting a brand new project, follow these steps to make sure it has the right internal structure so you and anyone else can work on it without problems:

1. First, navigate to [BU-ISCII](https://github.com/BU-ISCIII/) and click on "New".
![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/Create_repo_1.png)

2. Now name to your project and add a short description to it. Leave the repository public (you will need for hosting private repositories) and check the "Initialize this repository with a README" box. Click on "Create repository".
![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/Create_repo_2.png)

3. Your repository is now created, but we still need to give it our basic format. Click on "Branch: master", write "develop" and click on "Create branch".
![](https://github.com/BU-ISCIII/dotfiles/blob/master/images/Create_repo_3.png)

4. Now clone it to your local machine a transform it into a git-flow repository with

`git flow init`

Assign master to master the role and develop to the develop role, build the skeleton of your project in develop and push it back to GitHub!

## Opening an existing project

In case you want to work in an already existing (or the one you just created in the previous section):

1. Navigate to the project repository inside [BU-ISCII](https://github.com/BU-ISCIII/) and get the ssh URL.

2. Open a terminal, move to the directory where you want to create the local copy of the repository and type

`git clone <project_ssh_URL>`

where <project_ssh-URL> is the ssh URL you just got in point 1.

3. Move into the new created directory and there you are!

`cd <project_name>`

where <project_name> is the name of the project you just copied to your local machine.

## Update your local repository

`git pull`

## Checking existing branches

To see what branches are available and in which one you are working right now, type

`git branch`

## Working on an existing branch

To work in the branch <your_branch> just run

`git checkout <your_branch>`

## Uploading a new branch created locally to github

When you create a new branch locally, a normal push won't upload it to the main repository. To solve this issue, navigate to your local git branch folder and try:

```
git push -u origin `git rev-parse --abbrev-ref HEAD`
```

## Working on the develop branch

YOU SHOULD NOT WORK ON THE DEVELOP BRANCH!

The develop branch should be used to work only when there is no way to split the tasks to do in a new feature branch. One example of this situation could be when creating the project and the only task is to create the basic folder/files structures of the project and write the basic functionalities. Once this is done, basically anything can be split into a new feature branch.

If you are in one of the few cases where you should work on develop, just move to it with

`git checkout develop`

## Working on the master branch

NEVER. DO. IT! Please.

## Start a new feature

To start a new feature branch called <your_feature>, type

`git flow feature start <your_feature>`

## Start a new hotfix

To start a new hotfix branch called <your_hotfix>, type

`git flow hotfix start <your_hotfix>`

## Make Github aware of the newly created branch

```
git push -u origin `git rev-parse --abbrev-ref HEAD`

```

## Creating new files/folders

To create new files or folders in the project, just create them in your local copy of the repository and make git note them. Please, note that empty folders will not be tracked by Git. To do this, after creating them you can check what are the newly created files with

`git status`

You will see in your terminal what are the untracked changes in each branch. Now run

`git add ./`

and the files will be tracked and ready to be added in the main repository in the next push. You can check the changes typing again

`git status`

## Deleting a file

To delete a file called <your_file>, just delete it in your local copy of the repository using the command

`git rm <your_file>`

## Moving a file

In order to move files from <file_from> to <file_to>, just use this

`git mv <file_from> <file_to>`

This command can be used with regexps if you are moving multiple files, but ALWAYS test what the regexp is catching before you execute it.

## Create a commit

To make a commit of your changes, just type

`git commit -m "<whatever_you_did_this_time>"`

where <whatever_you_did_this_time> has to be a meaningful sentence explaining what you did in this commit.

### Commiting code guidelines

Write detailed commit messages in the past tense, not present tense.

- Good: “Fixed Unicode bug in RSS API.”
- Bad: “Fixes Unicode bug in RSS API.”
- Bad: “Fixing Unicode bug in RSS API.”

The commit message should be in lines of 72 chars maximum. There should be a subject line, separated by a blank line and then paragraphs of 72 char lines. The limits are soft. For the subject line, shorter is better. In the body of the commit message more detail is better than less:

```
Fixed #18307 -- Added git workflow guidelines

Refactored the Django's documentation to remove mentions of SVN
specific tasks. Added guidelines of how to use Git, GitHub, and
how to use pull request together with Trac instead.
```

- For commits to a branch, prefix the commit message with the branch name. For example: “[1.4.x] Fixed #xxxxx – Added support for mind reading.”

- Limit commits to the most granular change that makes sense. This means, use frequent small commits rather than infrequent large commits. For example, if implementing feature X requires a small change to library Y, first commit the change to library Y, then commit feature X in a separate commit. This goes a long way in helping everyone follow your changes.

- Separate bug fixes from feature changes. Bugfixes may need to be backported to the stable branch, according to Supported versions.

- If your commit closes a ticket in the ticket tracker, begin your commit message with the text “Fixed #xxxxx”, where “xxxxx” is the number of the ticket your commit fixes. Example: “Fixed #123 – Added whizbang feature.”. We’ve rigged Trac so that any commit message in that format will automatically close the referenced ticket and post a comment to it with the full commit message.

## Update changes (push)

NEVER, EVER, make a push without checking that all your files are correctly tracked or if you made any changes after your last commit. To avoid any mistakes with this, run

`git status`

to check the untracked file changes and, if you are not sure if you commited your last changes, check with

`git log -p -2`

when was your last commit and if there is any file modified after that date. Once you are sure that everything is documented and ready to update, upload the changes with

`git push`

## Close a feature

Closing a feature will remove the branch after merging it with develop, so do it only once the feature is fully functional after the last commit.

`git flow feature finish <your_feature>`

## Close a hotfix

Closing a hotfix will remove the branch after merging it  with both master and develop. Do it when the bug is fixed after the last commit.

`git flow hotfix finish <your_hotfix>`

## Release a new stable version

When releasing a final version <version> (format 'X.Y.Z') of the project, you will have to create a new branch using

`git flow release start <version>`

Do not make any changes here, THIS BRANCH IS NOT FOR DEVELOPMENT. Everything should have been already tested before releasing the stable version. Still, if you need to do any last minute changes, everything you do will be correctly merged to both master and develop at release time:

`git flow release finish <version>`

Version tags to the main repository have to be manually pushed:

`git push --tags`

## Download submodule

```Bash
git submodule init
git submodule update
git submodule add git@github.com:<user>/submodule_dir.git submodule_dir
```

## Update submodule

```Bash
cd submodule_dir
git pull
cd ..
git add submodule_dir
git commit -m "Pulled down update to submodule_dir"
```

## Check files with merge conflicts

```Bash
git diff --name-only --diff-filter=U
```

## Extra documentation

Here you can find useful links in case you want to learn more about Git, Github and git-flow. You should not need anything apart from what has been previously explained (as long as everyone stick to the plan), but extra knowledge does not harm!

- [Pro Git Book](https://git-scm.com/book/en/v2)
- [Original post by its creator](http://nvie.com/posts/a-successful-git-branching-model/)
- [Easy tutorial](https://datasift.github.io/gitflow/IntroducingGitFlow.html)
- [Even simpler tutorial](https://jeffkreeftmeijer.com/git-flow/)
- [Github official guides](https://guides.github.com/)
- [Git is fun](http://www.htzcomic.com/comics/2018-12-25-1223.png)
