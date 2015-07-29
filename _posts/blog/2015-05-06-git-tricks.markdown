---
layout: post
title:  "Git Tricks"
date:   2015-05-06
categories: blog
tags: [general,git]
author: shobhit_garg
share: true
comments: true
excerpt:
---

__When git repo is connected using ssh:__

For checking username/keys use:


{% highlight ruby %}

 ssh -T git@github.com 
 #If this is returns your user name that means you are connected with remote repo.

 ssh -vT git@github.com
 #If there are multiple ssh keys in your system then sometime you get confused which key your repo is using.So use this command to find out the ssh key your repo is using.

{% endhighlight %}

__git config files:__

Files which store the user details and branch details and some config settings like color etc.There are three config files on system:

Repository itself: <your_git_repository>/.git/config (for repo specific settings)

User home directory: ~/.gitconfig    (for a particular user)(use --global to access this)

System-wide directory: $(prefix)/etc/gitconfig  (for system wide settings)(use --system to access this)


Useful commands:

{% highlight ruby %}


#For checking config settings you can use the command:
git config --list

#If you want to change the username or email you can simplay do:
git config --global user.email 'abc@gmail.com' #for user global config file
git config  user.email 'abc@gmail.com' #for repo config file

#You can check the username and email in the same way:
git config --global user.email #for user global config file
git config --system user.name  #for system config file


{% endhighlight %}



__Default merge/Remote tracking branch:__

Under [branch "master"], try adding the following to the repo's Git config file (.git/config):


{% highlight ruby %}
[branch "master"]
    remote = origin
    merge = refs/heads/master

{% endhighlight %}

This tells Git 2 things:

When you're on the master branch, the default remote is origin.

When using git pull on the master branch, with no remote and branch specified, use the default remote (origin) and merge in the changes from the master branch.

If you don't want to edit the config file by hand, you can use the command-line tool instead:

{% highlight ruby %}
$ git config branch.master.remote origin
$ git config branch.master.merge refs/heads/master

#you can do this by checking out out the branch you want to set remote for:
git branch -u origin/branch_name
 
or

git branch --set-upstream-to origin/branch_name
{% endhighlight %}





__Setting _push default_ as current:__

You can set in git what you want to push when you do git push. There are many options like matching, nothing , current, upstream , simple . 
If always prefer to use __current__. When you use current you don't have to mention the remote branch everytime.It automatically gets mapped to the same name remote branch.Like branch name in local is branch1 then on git push it tries to push into remote branch branch1.

you can do this using:

{% highlight ruby %}
git config --global push.default current
{% endhighlight %}



__Reverting commits:__


{% highlight ruby %}
#If you want to revert a commit to a particular commit you can use:
git reset -—hard commit_hash #use commit hash


#If you want to revert some particular commits use:
git revert commit1hash commit2hash 



#reverting to previous commit:
git reset --hard HEAD


# Reverting a merge commit
git revert -m 1 <merge_commit_sha>

#reverting last two commits
git revert HEAD~2..HEAD

{% endhighlight %}



__Rebasing:__


Pull with rebase instead of merge

{% highlight ruby %}
$ git pull --rebase
# e.g. if on branch "master": performs a `git fetch origin`,
# then `git rebase origin/master'
{% endhighlight%}

Setting rebase instead of merging on pull :

{% highlight ruby %}

# make `git pull` on master always use rebase
$ git config branch.master.rebase true


# setup rebase for every tracking branch
$ git config --global branch.autosetuprebase always

{% endhighlight%}



