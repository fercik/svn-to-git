# SVN to Git

Follow this steps to move your Subversion repo to Git.

## Step 1
Initialize SVN enabled repo with GIT SVN tool.

    git svn init svn://example.com/project --stdlayout --no-metadata

* --stdlayout means that your SVN repo has standard directory structure (trunk, branches, tags in the same dir)
* --no-metadata will tell Git to not track SVN URL locations

## Step 2
Create 'authors' file to match SVN users to Git users.

#### File content should look like this:

    jdoe = John Doe <john.doe@example.com>
    gwash = George Washington <george.washington@example.com>
    theRock = Dwayne Johnson <dwayne.johnson@example.com>

#### Save file in Git config
We need to add this file to config, so Git knows how to match users.

    git config svn.authorsfile authors.txt

## Step 3
Let's start importing SVN revisions.

    git svn fetch

If during this operation you'd see information that your SVN repo has a user that you didn't specify in authors file, don't worry. Just add it to the file and start fetching again.

Whole operation might take some time, depending how big your repo is.

## Step 4
Cleaning the SVN stuff.

If you list all branches you'll probably see something like this.

    git branch -a

    * master
    remotes/tags/1.0.0
    remotes/tags/1.1.0
    remotes/tags/2.0.0
    [...]
    remotes/trunk

All SVN branches, tags etc. are now Git remotes. To easily make SVN tags git ones create little bash script and execute it.

    for t in `git branch -r | grep 'tags/' | sed s_tags/__` ; do
        git tag $t tags/$t^
        git branch -d -r tags/$t
    done

Let's remove other SVN references.

    git branch -d -r trunk
    git config --remove-section svn-remote.svn
    rm -rf .git/svn .git/{logs/,}refs/remotes/svn/

Convert remaining remote branches to local ones.

    git config remote.origin.url .
    git config --add remote.origin.fetch +refs/remotes/*:refs/heads/*
    git fetch

## Step 5
Create bare repo.

To do this we have to clone our repo.

    cd ..
    git clone --bare project

It will create a project.git directory.

## Step 6
Push repo to remote repository.

    cd project.git
    git push --mirror https://github.com/someuser/project.git

Now you can remove bare repository.

    cd ..
    rm -rf project.git

That's it! Now you can clone your repo and start working with Git :)


### Sources

    http://blokspeed.net/blog/2010/09/converting-from-subversion-to-git/

    https://help.github.com/articles/importing-a-git-repository-using-the-command-line/