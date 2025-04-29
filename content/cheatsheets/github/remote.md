---
title: Remote
linktitle: 🌍 Remote
toc: false
type: book
date: "2019-05-05T00:00:00+01:00"
tags:
  - GitHub

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

Working with Git Remotely

<!--more-->

* Clone a remote repo:
  ```
      $ git clone <url>
  ```
* Clone a specific branch of a remote repo:
  ```
      $ git clone -b <remorebranch> <url> <localbranch>
  ```
* To add a new remote Git repository to local repo:
  ```
      $ git remote add <url-alias> <url>
  ```
* Set upstream with local and remote branch:
  ```
      $ git branch --set-upstream-to <url-alias>/<remotebranch> <localbranch>
  ```
* Fetch and merge changes from remote branch to local branch:
  ```
      $ git pull <url-alias> <remotebranch>
  ```
* Push changes to remote branch:
  ```
      $ git push <url-alias> <remotebranch>
  ```
* Push changes to remote branch. -u would create the remote branch:
  ```
      $ git push -u <url-alias> <remotebranch>
  ```
* Push local tags to remote:
	* To push all the tags to remote:
  	```
          $ git push --tags
    ```
    * To push a particular tag to remote:  
    ```
        $ git push <url-alias> <tagname>
    ```
  > Note: When you push your local repo to remote the local tags will not be pushed to remote by itself, you would need to do this explicitly.

* Show remote url-alias and url:
  ```
     $ git remote -v
  ```
* Remove remote reference with local repo:
  ```
      $ git remote rm <url-alias>
  ```
* Delete a remote branch after merging
(i.e. you missed the checkbox during the merge.              Alternatively you can use the Github/Bitbucket UI to delete the remote branch.)
  ```
      $ git push origin --delete <branchname>
  ```
* Add a Git submodule to an existing Git Project
  ```
     $ git submodule add <url> folder-to-download/
  ```
* Remove a Git submodule from an existing Git Project
  ```
      Delete the relevant section from the .gitmodules file.
      Stage the .gitmodules changes git add .gitmodules
      Delete the relevant section from .git/config.
      Run git rm --cached path_to_submodule (no trailing slash).
      Run rm -rf .git/modules/path_to_submodule (no trailing slash).
      Commit git commit -m "Removed submodule "
      Delete the now untracked submodule files rm -rf path_to_submodule
  ```
