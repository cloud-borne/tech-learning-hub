---
title: Working with Multiple GitHub Accounts from the Same Machine
subtitle: A step by step to link multiple github accounts

# Summary for listings and search engines
summary: A step by step to link multiple github accounts

# Link this post with a project
projects: []

# Date published
date: "2024-01-16T00:00:00Z"

toc: true

# Date updated
lastmod: "2024-01-16T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption:
  focal_point: "Center"
  placement: 1
  preview_only: false

authors:
- admin

tags:
- CICD
- GitHub

categories:

---

<!--more-->

### Overview

When you need to work with different GitHub accounts—like one for personal projects and another for work—things can get tricky. How do you manage them without running into permission issues? 🤔

In this post, we’ll walk through how to configure Git on your machine so you can seamlessly switch between multiple GitHub accounts without any headaches! 🤕

Let suppose you have two github accounts, ```https://github.com/github-work``` and ```https://github.com/github-personal```. Now you want to setup your machine to easily talk to both the github accounts.

Let's dive in!🐬

{{% stepper %}}

<div class="step">

  ## Step 1: Remove Global 👌

  This step is **recommended**, but optional. I will prefer not to have a default user for all projects, but it is up to you.
  Now we are going to open the ```global``` config using the command:
  
  ```bash
  git config --edit --global
  ```
  Remove all ```[credential]``` and ```[user]``` configurations:

  ```bash
  [user]
  [core]
  [core]
  [core]
    autocrlf = false
    editor = code --wait
  [color]
    ui = auto
  [credential]
  [merge]
    tool = kdiff3
  [mergetool "kdiff3"]
    path = C:/Program Files/KDiff3/kdiff3.exe
  ...
  ``` 

</div>
<div class="step">

  ## Step 2: Generate SSH Keys🔑 for all accounts

  * First make sure your current directory is your .ssh folder.
    ```bash
    cd ~/.ssh
    ```

  * Generating unique ```ssh``` key for both accounts:
    ```bash
    ssh-keygen -t ed25519 -C "github-personal@gmail.com" -f "github-personal"
    ssh-keygen -t ed25519 -C "github-work@company.com" -f "github-work"
    ```
    here:
    - ```-C``` stands for comment to help identify your ssh key
    - ```-f``` stands for the file name where your ssh key get saved

</div>
<div class="step">
  
  ## Step 3: Add Your SSH Keys to the SSH Agent

  * Ensure the ```ssh-agent``` is running. If not start it manually:
    ```bash
    # start the ssh-agent in the background
    eval "$(ssh-agent -s)"
    ```
  * Add your SSH private key to the ```ssh-agent```.
    ```bash
    ssh-add ~/.ssh/github-personal
    ssh-add ~/.ssh/github-work
    ```

</div>
<div class="step">
  
  ## Step 4: Add SSH Keys to GitHub

  * Copy the SSH public key to your **clipboard**.
      
    ```bash
    # Copies the contents of the id_ed25519.pub file to your clipboard
    clip < ~/.ssh/github-username.pub
    ```

  * In the upper-right corner of any page, click your profile photo, then click ```Settings```
      {{< figure src="images/uploads/userbar-account-settings.png" width="250" height="400">}}

  * In the **Access** section of the sidebar, click ```SSH and GPG keys```.
      {{< figure src="images/uploads/settings-sidebar-ssh-keys.png" width="250" height="250">}}

  * Click ```New SSH key``` or ```Add SSH key```.
      {{< figure src="images/uploads/ssh-add-ssh-key.png" width="500" height="500">}}

  * In the **Title** field, add a descriptive label for the new key.
      For example, if you're using a personal laptop, call this key "Personal Laptop".💡

  * Paste your key into the **Key** field.
      {{< figure src="images/uploads/ssh-key-paste.png" width="500" height="500">}}

  * Click ```Add SSH key```.

</div>
<div class="step">
  
  ## Step 4: Create separate directories (one personal and one work)

  * We are going to define a path where our projects are going to live and create a ```.gitconfig``` file for each user profile, as many as we need.

    ```bash
    C:
    ├── Users 
          ├── <username>
                  ├── .gitconfig <-- global
    D:              
    ├── Repos
          └── Developer/
                  ├── personal/
                  │   ├── project_1/
                  │   ├── project_2/
                  │   ├── project_#/
                  │   └── .gitconfig <-- personal
                  └── work/
                      ├── project_1/
                      ├── project_2/
                      ├── project_#/
                      └── .gitconfig <-- work
    ```
  * **Personal**:
    ```bash
    # ~/Developer/github-personal/.gitconfig

    [core]
        sshCommand = ssh -i ~/.ssh/github-personal -F /dev/null
    [credential]
        username = github-personal
    [user]
        name = github-personal
        email = github-personal@gmail.com
    ```
  * **Work**:
    ```bash
    # ~/Developer/github-work/.gitconfig

    [core]
        sshCommand = ssh -i ~/.ssh/github-work -F /dev/null
    [credential]
        username = github-work
    [user]
        name = github-work
        email = github-work@company.com
    ```
</div>
<div class="step">
  
  ## Step 5: Update the Global config

  * Now we are going to open the global config using ```git config --edit --global``` command and point to the local configs:

    ```bash
    [includeIf "gitdir/i:D:/Repos/Developer/github-personal/"]
        path = D:/Repos/Developer/github-personal/.gitconfig

    [includeIf "gitdir/i:D:/Repos/Developer/github-work/"]
        path = D:/Repos/Developer/github-work/.gitconfig
    ```

</div>
<div class="step">
  
  ## Step 6: Voila 🎉 switch to personal/work folder and work

  * It will take the user configuration profile per path, and you can create or clone projects inside each profile path without dealing with manual configurations and avoid using the ```amend``` command to fix mistakes.

</div>

{{% /stepper %}}

## Wrapping It Up 🎁

Now you can work with both your personal and work GitHub accounts from the same machine without any issues. When you push or pull, Git will automatically use the correct SSH key and account based on the folder you’re working on! 
