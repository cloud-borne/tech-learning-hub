---
title: Gitflow
linktitle: 🚦 Gitflow
toc: true
type: book
date: "2019-05-05T00:00:00+01:00"
tags:
  - GitHub

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

Gitflow for the ```Enterprise```

<!--more-->

![Gitflow](/images/uploads/Gitflow.JPG)

What I have here is a branching strategy for managing a ```Monolithic``` application being worked upon by a distributed team.
View it on [Github](https://github.com/avijitliberty/honeycomb/blob/master/static/images/uploads/Gitflow.JPG) for your 👀's

## Overview

For managing the release strategy in Git for the enterprise we can plan on using a slightly modified version of a Gitflow workflow. If you're not familiar there's a lot of info online documenting this pattern, and [Atlassian](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) has a nice write-up on their site. By applying this workflow we hope to be able to maintain stable branches to run builds and deploys from, provide useful checkpoints to serve as a natural code review, and allow easier merges that are tied more directly to the individual delivering conflicting code. For local development we would be using a collection of Git extensions to provide high-level repository operations via: [nvie/gitflow](https://github.com/nvie/gitflow)

The pattern breaks down into two main groups of branches:

1. **Long-lived** community owned branches that require pull requests to merge code into. These branches should always be expected to be buildable/deployable and the pull request provides a natural point for conducting a code review:
   - **master** -
      1. this branch will be kept in sync with production code
      2. main purpose is really record keeping so that we always have a location we could build from to replicate production code
      3. should only ever get code through pull request from a release branch
   - **release/** -
      1. we will create a release branch for each release by branching from develop on **construct/regression startdate**.
      2. will serve as the source for all release candidate builds that are promoted through uat to prod.
      3. should be feature stable and only **bugfix** development should be happening against it
      4. will be tagged on each successful CI build.
      5. should only receive code through pull requests from **bugfix** branches.
      6. As soon as we create the **release** branch, **develop** becomes available for changes for the next **release**. Only **bugfixes** for the current **release** happens on the **release** branch. Any new features go into **develop** for the next release.
      7. EC/EBF for prod is required a release branch would be created from master to support it. These use the naming convention release/**Major.Minor.Patch** where Patch is the version number which is incremented for each off-cycle release. The first off-schedule release of any **release** starts at 1.
   - **develop** -
      1. default development target
      2. all **release** branches will be cut from develop, that means any code going into this branch should be expected to be delivered to production with the next **release**
      3. will receive code from feature branch pulls, and from automated merges from release and master branches as they receive changes.
2. **Short-term** individually owned branches which are used to house active development of a feature or bugfix. These branches should be small and live no more than a few days. They should be deleted once they have been pulled into one of the stable streams. The use of the prefixes like **feature/** or **bugfix/** will help all viewers of repository history to quickly identify the source and intent of code, this makes the use of the naming pattern very important from a usability perspective:
   - **feature/**<featureName> -
      1. for new feature development which is being targeted for the monthly release that falls after the currently named release branch
      2. should never be pulled into any branch other than **develop**
   - **bugfix/**<bugfixName> -
      1. for post-construct fixes being applied to an existing feature in a release
      2. should never be pulled into any branch other than a **release**
   - **hotfix/**<hotfixName> -
      1. for post-prod fixes being applied to an existing EC/EBF branch
      2. should never be pulled into any branch other than a **release**

## Installation

For Windows users, [Git for Windows](https://gitforwindows.org/) is a good starting place for installing git. Git for Windows also includes Gitflow.
For other platforms please refer documentation [here](https://github.com/nvie/gitflow/wiki/Installation)

## Gitflow Setup

Create a new GitHub repo like: [gitflow-demo](https://github.com/avijitliberty/gitflow-demo).

  ```
  $ git clone git@github.com:avijitliberty/gitflow-demo.git
  $ cd gitflow-demo
  $ /c/Repos/gitflow-demo (master)
  $ git flow init

  Which branch should be used for bringing forth production releases?
     - master
  Branch name for production releases: [master]
  Branch name for "next release" development: [develop]

  How to name your supporting branch prefixes?
  Feature branches? [feature/]
  Bugfix branches? [bugfix/]
  Release branches? [release/]
  Hotfix branches? [hotfix/]
  Support branches? [support/]
  Version tag prefix? []
  Hooks and filters directory? [C:/Repos/gitflow-demo/.git/hooks]

  $ /c/Repos/gitflow-demo (develop)
  $ git push origin --all

  ```

> What happened:
>  - As described the branching strategy above Gitflow extension established a few enterprise standards.
>  - A new **develop** branch got created and checked out.
>  - We pushed everything back to remote

At this stage you would want to set **develop** as your **default** branch.

## Use Case: Creating Features

So let's say we would like to add a new **feature** for the next release.
Features can be created against **develop** (default Gitflow behavior) or **release** branch.
As per our branching model Features go to **develop** and no where **else**

Here's how that would work in Gitflow:

```
/c/Repos/gitflow-demo (develop)
$ git flow feature start lineFeature
Switched to a new branch 'feature/lineFeature'

Summary of actions:
- A new branch 'feature/lineFeature' was created, based on 'develop'
- You are now on branch 'feature/lineFeature'

Now, start committing on your feature. When done, use:

     git flow feature finish lineFeature

/c/Repos/gitflow-demo (feature/lineFeature)
```
Now you could make your feature changes, commit and push to remote.

```
$ vi lines.txt

# For illustration purpose we create a file like so:
# line 1.0
# line 2.0
# line 3.0
# line 4.0

/c/Repos/gitflow-demo (feature/lineFeature)
$ git commit -am "Added Line Feature"

/c/Repos/gitflow-demo (feature/lineFeature)
$ git push -u origin feature/lineFeature

```
Now you could create a Pull Request in GitHub, get a Code review done and merge to **develop**.
![Gitflow-Feature-Merge](/images/uploads/gitflow-feature-pullrequest-merge.PNG)

Lastly clear up the feature branch from local and remote.

```
/c/Repos/gitflow-demo (feature/lineFeature)
$ git flow feature finish
Switched to branch 'develop'
Updating bc762dc..8b091b4
Fast-forward
 lines.txt | 4 ++++
 1 file changed, 4 insertions(+)
 create mode 100644 lines.txt
To github.com:avijitliberty/gitflow-demo.git
 - [deleted]         feature/lineFeature
Deleted branch feature/lineFeature (was 8b091b4).

Summary of actions:
- The feature branch 'feature/lineFeature' was merged into 'develop'
- Feature branch 'feature/lineFeature' has been locally deleted; it has been remotely deleted from 'origin'
- You are now on branch 'develop'

/c/Repos/gitflow-demo (develop)
$
```

## Use Case: Creating Releases and Bugfixes

To start a **release**, use the git flow **release** command. It creates a release branch created from the **develop** branch by default.

```
/c/Repos/gitflow-demo (develop)
$ git flow release start 1.0
Switched to a new branch 'release/1.0'

Summary of actions:
- A new branch 'release/1.0' was created, based on 'develop'
- You are now on branch 'release/1.0'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish '1.0'

/c/Repos/gitflow-demo (release/1.0)
$
```
It's wise to publish the release branch after creating it to allow release commits by other developers.

```
/c/Repos/gitflow-demo (release/1.0)
$ git push origin --all
Everything up-to-date
```

Now as is the case often in real-life projects, QA finds a :bug: with "line4" :angry:

Bugfixes can be created against **develop** (default Gitflow behavior) or **release** branch.
As per our branching model Bugfixes go to **release** and no where **else**.

Here's how we would go about fixing it:

```
/c/Repos/gitflow-demo (release/1.0)
$ git flow bugfix start line4Bug release/1.0
Switched to a new branch 'bugfix/line4Bug'

Summary of actions:
- A new branch 'bugfix/line4Bug' was created, based on 'release/1.0'
- You are now on branch 'bugfix/line4Bug'

Now, start committing on your bugfix. When done, use:

     git flow bugfix finish line4Bug

/c/Repos/gitflow-demo (bugfix/line4Bug)
$ git commit -am "Fixed Line4Bug"
$ git push -u origin bugfix/line4Bug
```

Now you could create a Pull Request in GitHub, get a Code review done and merge to **release/1.0**.
![Gitflow-BugFix-Merge](/images/uploads/gitflow-bugfix-pullrequest-merge.PNG)

Lastly clear up the **bugfix/** branch from local and remote.

```
/c/Repos/gitflow-demo (bugfix/line4Bug)
$ git flow bugfix finish line4Bug
Switched to branch 'release/1.0'
Your branch is up to date with 'origin/release/1.0'.
Updating 798f10e..2dc5aeb
Fast-forward
 lines.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
To github.com:avijitliberty/gitflow-demo.git
 - [deleted]         bugfix/line4Bug
Deleted branch bugfix/line4Bug (was 2dc5aeb).

Summary of actions:
- The bugfix branch 'bugfix/line4Bug' was merged into 'release/1.0'
- bugfix branch 'bugfix/line4Bug' has been locally deleted; it has been remotely deleted from 'origin'
- You are now on branch 'release/1.0'

/c/Repos/gitflow-demo (release/1.0)
$

```
Now we could finish the release 1.0 at this point like so:

```
/c/Repos/gitflow-demo (release/1.0)
$ git flow release finish
Branches 'develop' and 'origin/develop' have diverged.
And local branch 'develop' is ahead of 'origin/develop'.
Switched to branch 'master'
Your branch is up to date with 'origin/master'.
Merge made by the 'ort' strategy.
 lines.txt | 6 +++---
 1 file changed, 3 insertions(+), 3 deletions(-)
Already on 'master'
Your branch is ahead of 'origin/master' by 9 commits.
  (use "git push" to publish your local commits)
Switched to branch 'develop'
Your branch is ahead of 'origin/develop' by 1 commit.
  (use "git push" to publish your local commits)
Merge made by the 'ort' strategy.
 lines.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
To github.com:avijitliberty/gitflow-demo.git
 - [deleted]         release/1.0
Deleted branch release/1.0 (was e1bd83d).

Summary of actions:
- Release branch 'release/1.0' has been merged into 'master'
- The release was tagged '1.0'
- Release tag '1.0' has been back-merged into 'develop'
- Release branch 'release/1.0' has been locally deleted; it has been remotely deleted from 'origin'
- You are now on branch 'develop'


/c/Repos/gitflow-demo (develop)
$ git push origin --all --follow-tags

```

{{% callout note %}}

Please make a note of what **release finish** did!

- Release branch 'release/1.0' has been merged into 'master'
- The release was tagged '1.0'
- Release tag '1.0' has been back-merged into 'develop'
- Release branch 'release/1.0' has been locally deleted; it has been remotely deleted from 'origin'
- You are now on branch 'develop'

{{% /callout %}}

## Use Case: Resolve a Merge conflict

Next we would create a merge conflict scenario and see how we would go about fixing it using Gitflow.
When a **release** branch change is conflicting with a **develop** branch change then it would cause of merge conflict because the back-merged into 'develop' would fail:

> :thought_balloon: Release tag '1.0' has been back-merged into 'develop'

Let's recreate this use-case next:

1. Start the next release:
    ```
    /c/Repos/gitflow-demo (develop)
    $ git flow release start 1.1
    Switched to a new branch 'release/1.1'

    Summary of actions:
    - A new branch 'release/1.1' was created, based on 'develop'
    - You are now on branch 'release/1.1'

    Follow-up actions:
    - Bump the version number now!
    - Start committing last-minute fixes in preparing your release
    - When done, run:

         git flow release finish '1.1'

    /c/Repos/gitflow-demo (release/1.1)

    ```

2. Now say we have a bug at line 3 this time. So we would want to fix it with a bugfix branch.

    ```
    /c/Repos/gitflow-demo (release/1.1)
    $ git flow bugfix start line3Bug release/1.1
    Switched to a new branch 'bugfix/line3Bug'

    Summary of actions:
    - A new branch 'bugfix/line3Bug' was created, based on 'release/1.1'
    - You are now on branch 'bugfix/line3Bug'

    Now, start committing on your bugfix. When done, use:

         git flow bugfix finish line3Bug

    /c/Repos/gitflow-demo (bugfix/line3Bug)

    ```

3. We fix the bug, do a **pull request/code review** as before and **finish** the bugfix

    ```
    /c/Repos/gitflow-demo (bugfix/line3Bug)
    $ vi lines.txt

    /c/Repos/gitflow-demo (bugfix/line3Bug)
    $ git commit -am "Fixed Line3Bug in release branch"
    [bugfix/line3Bug fe00b6c] Fixed Line3Bug in release branch
     1 file changed, 1 insertion(+), 1 deletion(-)

    /c/Repos/gitflow-demo (bugfix/line3Bug)
    $ git push -u origin bugfix/line3Bug
    Enumerating objects: 5, done.
    Counting objects: 100% (5/5), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (3/3), 293 bytes | 293.00 KiB/s, done.
    Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
    remote: Resolving deltas: 100% (1/1), completed with 1 local object.
    remote:
    remote: Create a pull request for 'bugfix/line3Bug' on GitHub by visiting:
    remote:      https://github.com/avijitliberty/gitflow-demo/pull/new/bugfix/line3Bug
    remote:
    To github.com:avijitliberty/gitflow-demo.git
     * [new branch]      bugfix/line3Bug -> bugfix/line3Bug
    branch 'bugfix/line3Bug' set up to track 'origin/bugfix/line3Bug'.

    /c/Repos/gitflow-demo (bugfix/line3Bug)
    $ git flow bugfix finish line3Bug
    Switched to branch 'release/1.1'
    Updating 7278ca6..fe00b6c
    Fast-forward
     lines.txt | 2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
    To github.com:avijitliberty/gitflow-demo.git
     - [deleted]         bugfix/line3Bug
    Deleted branch bugfix/line3Bug (was fe00b6c).

    Summary of actions:
    - The bugfix branch 'bugfix/line3Bug' was merged into 'release/1.1'
    - bugfix branch 'bugfix/line3Bug' has been locally deleted; it has been remotely deleted from 'origin'
    - You are now on branch 'release/1.1'

    /c/Repos/gitflow-demo (release/1.1)

    ```

4. Next we would do a feature on develop potentially changing the line3 as well.
In an enterprise that's typical since develop is open for future development for the next release and multiple distributed teams simultaneously working together.

    ```
    /c/Repos/gitflow-demo (release/1.1)
    $ git checkout develop
    Switched to branch 'develop'
    Your branch is up to date with 'origin/develop'.

    /c/Repos/gitflow-demo (develop)
    $ git flow feature start line3Feature
    Switched to a new branch 'feature/line3Feature'

    Summary of actions:
    - A new branch 'feature/line3Feature' was created, based on 'develop'
    - You are now on branch 'feature/line3Feature'

    Now, start committing on your feature. When done, use:

         git flow feature finish line3Feature

    /c/Repos/gitflow-demo (feature/line3Feature)
    $ vi lines.txt

    /c/Repos/gitflow-demo (feature/line3Feature)
    $ git commit -am "Added Line3Feature changes"
    [feature/line3Feature fdb3fe6] Added Line3Feature changes
     1 file changed, 1 insertion(+), 1 deletion(-)

    /c/Repos/gitflow-demo (feature/line3Feature)
    $ git push -u origin feature/line3Feature
    Enumerating objects: 5, done.
    Counting objects: 100% (5/5), done.
    Delta compression using up to 8 threads
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (3/3), 288 bytes | 288.00 KiB/s, done.
    Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
    remote: Resolving deltas: 100% (1/1), completed with 1 local object.
    remote:
    remote: Create a pull request for 'feature/line3Feature' on GitHub by visiting:
    remote:      https://github.com/avijitliberty/gitflow-demo/pull/new/feature/line3Feature
    remote:
    To github.com:avijitliberty/gitflow-demo.git
     * [new branch]      feature/line3Feature -> feature/line3Feature
    branch 'feature/line3Feature' set up to track 'origin/feature/line3Feature'.

    /c/Repos/gitflow-demo (feature/line3Feature)
    $ git flow feature finish line3Feature
    Switched to branch 'develop'
    Your branch is up to date with 'origin/develop'.
    Updating 7278ca6..fdb3fe6
    Fast-forward
     lines.txt | 2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
    To github.com:avijitliberty/gitflow-demo.git
     - [deleted]         feature/line3Feature
    Deleted branch feature/line3Feature (was fdb3fe6).

    Summary of actions:
    - The feature branch 'feature/line3Feature' was merged into 'develop'
    - Feature branch 'feature/line3Feature' has been locally deleted; it has been remotely deleted from 'origin'
    - You are now on branch 'develop'

    ```

5. Now let's say the release happened to production and we would want to **finish** the release.
First before we finish the release we would have to make sure locally we are in good standing in terms of **develop** and **release**

    ```
    /c/Repos/gitflow-demo (develop)
    $ git branch -l
    * develop
      master
      release/1.1

    /c/Repos/gitflow-demo (develop)
    $ git checkout release/1.1
    Switched to branch 'release/1.1'

    /c/Repos/gitflow-demo (release/1.1)
    $ git pull
    Updating fe00b6c..a543b83
    Fast-forward

    /c/Repos/gitflow-demo (release/1.1)
    $ git checkout develop
    Switched to branch 'develop'
    Your branch is behind 'origin/develop' by 1 commit, and can be fast-forwarded.
      (use "git pull" to update your local branch)

    /c/Repos/gitflow-demo (develop)
    $ git pull
    Updating fdb3fe6..2b3ef8c
    Fast-forward

    /c/Repos/gitflow-demo (develop)
    ```

6. Next we would try to **finish** the release. What would happen is the it would try to merge to **master** and then to **develop**. And the **develop** merge would fail with merge conflict like below. We can use a mergetool like KDiff3 to resolve the same.

    ```
    $ git checkout release/1.1
    Switched to branch 'release/1.1'
    Your branch is up to date with 'origin/release/1.1'.

    /c/Repos/gitflow-demo (release/1.1)
    $ git flow release finish
    Switched to branch 'master'
    Your branch is up to date with 'origin/master'.
    Merge made by the 'ort' strategy.
     lines.txt | 2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
    Already on 'master'
    Your branch is ahead of 'origin/master' by 5 commits.
      (use "git push" to publish your local commits)
    Switched to branch 'develop'
    Your branch is up to date with 'origin/develop'.
    Auto-merging lines.txt
    CONFLICT (content): Merge conflict in lines.txt
    Automatic merge failed; fix conflicts and then commit the result.
    Fatal: There were merge conflicts.

    /c/Repos/gitflow-demo (develop|MERGING)
    $ git mergetool

    /c/Repos/gitflow-demo (develop|MERGING)
    $ git merge --continue
    [develop 4f56ddf] Merge tag '1.1' into develop

    /c/Repos/gitflow-demo (develop)

    /c/Repos/gitflow-demo (develop)
    $ git checkout release/1.1
    Switched to branch 'release/1.1'
    Your branch is up to date with 'origin/release/1.1'.

    /c/Repos/gitflow-demo (release/1.1)
    $ git flow release finish
    Branches 'master' and 'origin/master' have diverged.
    And local branch 'master' is ahead of 'origin/master'.
    Branches 'develop' and 'origin/develop' have diverged.
    And local branch 'develop' is ahead of 'origin/develop'.
    Switched to branch 'master'
    Your branch is ahead of 'origin/master' by 5 commits.
      (use "git push" to publish your local commits)
    To github.com:avijitliberty/gitflow-demo.git
     - [deleted]         release/1.1
    Deleted branch release/1.1 (was a543b83).

    Summary of actions:
    - Release branch 'release/1.1' has been merged into 'master'
    - The release was tagged '1.1'
    - Release branch 'release/1.1' has been locally deleted; it has been remotely deleted from 'origin'
    - You are now on branch 'master'


    /c/Repos/gitflow-demo (master)
    $ git push origin --all --follow-tags

    ```

## Use Case: Creating EC and Hotfix

As is common place in any enterprise application production defects happen and need to be fixed with quick turn around. There comes the need for a EC/EBF and **hotfix** branches.

Here's how that would work with Gitflow:

1. Create EC branch from master:

    ```
    /c/Repos/gitflow-demo (master)
    $ git flow release start 1.2.1 master
    Switched to a new branch 'release/1.2.1'

    Summary of actions:
    - A new branch 'release/1.2.1' was created, based on 'master'
    - You are now on branch 'release/1.2.1'

    Follow-up actions:
    - Bump the version number now!
    - Start committing last-minute fixes in preparing your release
    - When done, run:

         git flow release finish '1.2.1'


    /c/Repos/gitflow-demo (release/1.2.1)
    $

    ```
2. Now create your hotfix from release/1.2.1 branch like so:

    ```
    /c/Repos/gitflow-demo (release/1.2.1)
    $ git flow hotfix start line3HotFix release/1.2.1
    Switched to a new branch 'hotfix/line3HotFix'

    Summary of actions:
    - A new branch 'hotfix/line3HotFix' was created, based on 'release/1.2.1'
    - You are now on branch 'hotfix/line3HotFix'

    Follow-up actions:
    - Start committing your hot fixes
    - Bump the version number now!
    - When done, run:

     git flow hotfix finish 'line3HotFix'

     /c/Repos/gitflow-demo (hotfix/line3HotFix)
     $ vi lines.txt

     # Fix lines.txt like so:
     # line 1.1
     # line 2.1
     # line 3.2.1
     # line 4.1

     /c/Repos/gitflow-demo (hotfix/line3HotFix)
     $ git commit -am "Fixed Line3Bug as HotFix"

     /c/Repos/gitflow-demo (hotfix/line3HotFix)
     $ git push -u origin hotfix/line3HotFix

     /c/Repos/gitflow-demo (hotfix/line3HotFix)
     $ git push origin --all

    ```
4. Now as before let the **pull request/code review** iteration happen as before.
Finally clean up the **hotfix** branch like so:

    ```
    /c/Repos/gitflow-demo (hotfix/line3HotFix)
    $ git flow hotfix finish 'line3HotFix'
    Switched to branch 'release/1.2.1'
    Merge made by the 'ort' strategy.
     lines.txt | 2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
    To github.com:avijitliberty/gitflow-demo.git
     - [deleted]         hotfix/line3HotFix
    Deleted branch hotfix/line3HotFix (was 101bc45).

    Summary of actions:
    - Hotfix branch 'hotfix/line3HotFix' has been merged into 'release/1.2.1'
    - The hotfix was tagged 'line3HotFix'
    - Hotfix branch 'hotfix/line3HotFix' has been locally deleted; it has been remotely deleted from 'origin'
    - You are now on branch 'release/1.2.1'

    /c/Repos/gitflow-demo (release/1.2.1)
    $ git push origin --all --follow-tags

    ```
5. The logical next step would be to **finish** the release like so:

    ```
    /c/Repos/gitflow-demo (develop)
    $ git checkout release/1.2.1
    Switched to branch 'release/1.2.1'
    Your branch is ahead of 'origin/release/1.2.1' by 2 commits.
      (use "git push" to publish your local commits)

    /c/Repos/gitflow-demo (release/1.2.1)
    $ git flow release finish
    Branches 'release/1.2.1' and 'origin/release/1.2.1' have diverged.
    And local branch 'release/1.2.1' is ahead of 'origin/release/1.2.1'.
    Switched to branch 'master'
    Your branch is up to date with 'origin/master'.
    Merge made by the 'ort' strategy.
     lines.txt | 2 +-
     1 file changed, 1 insertion(+), 1 deletion(-)
    Already on 'master'
    Your branch is ahead of 'origin/master' by 5 commits.
      (use "git push" to publish your local commits)
    To github.com:avijitliberty/gitflow-demo.git
     - [deleted]         release/1.2.1
    Deleted branch release/1.2.1 (was d55fd43).

    Summary of actions:
    - The release was tagged '1.2.1'
    - Release branch 'release/1.2.1' has been locally deleted; it has been remotely deleted from 'origin'
    - You are now on branch 'master'


    /c/Repos/gitflow-demo (master)
    $ git push origin --all --follow-tags

    ```

## Merging Scenarios

### Automated
Based on the Gitflow branching strategy when a release (EC or regular) goes to production the release branch it came from should be merged to master and develop with a tag added to the head commit of the release branch.

> Before creating the release branch, it's a good idea to check Github and make sure the release branch to
be merged to master isn't ahead of develop at all as this may cause conflicts when merging. It shouldn't be since construct happens after the production release, but as a sanity check it's a good idea to look👀 first.

### Manual

- If there were Bugfixes delivered to the release branch it's a good idea to manually merge it to develop. This could potentially eliminate getting merge conflicts when release is merged back to develop.

- If there were long running branches created to support larger programs like **feature/abc** or **feature/xyz** then those branches need to get manual merges from develop to remain in sync. This merging needs to be done **manually** by the respective team developers based on individual project needs.

## Enterprise considerations

- Schemantic Versioning: Semantic versioning (also referred as **SemVer**) is a versioning system to use when developing/releasing a software for most enterprises. The idea is:

{{% callout note %}}

Given a version number **Major.Minor.Patch**, increment the:
- **Major** version when you make incompatible API changes,
- **Minor** version when you add functionality in a backwards compatible manner, and
- **Patch** version when you make backwards compatible bug fixes.

![Gitflow-Semantic-Versioning](/images/uploads/gitflow-semantic-version.PNG)

{{% /callout %}}

- In most enterprises teams would setup branch protection rules to restrict direct merges to your long-running branches without a **pull request**/**code review** to maintain stability.
![Gitflow-Branch-Settings](/images/uploads/gitflow-branch-settings.PNG)

In that scenario, the **release finish** would not work since the merging on the remote would get blocked by the rules. In order to circumvent this problem, we could use a [GitHub Action]({{< ref "/cheatsheets/github/actions.md" >}} "GitHub Action") to **finish** the release.

## Further Read

Here's some interesting reads for further deep dive
- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
- [Gitflow Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)
- [git-flow-cheatsheet](http://danielkummer.github.io/git-flow-cheatsheet/)
