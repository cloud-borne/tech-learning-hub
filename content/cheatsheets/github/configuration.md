---
title: Configuration
linktitle: 🛠️ Configuration
type: book
date: "2019-05-05T00:00:00+01:00"
toc: true
tags:
  - GitHub

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

Git Configuration

<!--more-->

## Connecting to GitHub with SSH

You can connect to GitHub using the Secure Shell Protocol (SSH), which provides a secure channel over an unsecured network.

1. Checking for existing SSH keys
    ```bash
    # Lists the files in your .ssh directory, if they exist
    ls -al ~/.ssh
    ```

2. Generating a new SSH key
    ```bash
    ## -C stands for comment to help identify your ssh key
    ## -f stands for the file name where your ssh key get saved
    ssh-keygen -t ed25519 -C "your_email@example.com" -f "github-username"
    ```
    When you're prompted to *Enter a file in which to save the key*, press ```Enter```. This accepts the default file location.

    ```bash
    > Enter a file in which to save the key (/c/Users/you/.ssh/id_algorithm):[Press enter]
    At the prompt, type a secure passphrase. For more information, see "Working with SSH key passphrases."
    > Enter passphrase (empty for no passphrase): [Type a passphrase]
    > Enter same passphrase again: [Type passphrase again]
    ```

3. Ensure the ```ssh-agent``` is running. Start it manually:
  
    ```bash
    # start the ssh-agent in the background
    eval "$(ssh-agent -s)"
    ```

4. Add your SSH private key to the ````ssh-agent````. If you created your key with a different name, or if you are adding an existing key that has a different name, replace ```github-username``` in the command with the name of your private key file.

    ```bash
    ssh-add ~/.ssh/github-username
    ```

5. Add the SSH key to your account on ```GitHub```.

    * Copy the SSH public key to your clipboard.
      
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

{{% callout note %}}

- While cloning/pulling/pushing code from remote use your username and generated token when prompted for username and password respectively.
- On Windows this stores your credentials in the Windows credential store which has a Control Panel interface where you can delete or edit your stored credentials. With this store, your details are secured by your Windows login and can persist over multiple sessions.

{{% /callout %}}

## Initialize a new local repository

  ```bash
  git init
  ```

  {{% callout note %}}
  It will create the hidden .git folder which is used by git to manage the repository in your working directory
  {{% /callout %}}

## Developer Configuration

* In ```Git```, there are **three** main levels of configuration that allow you to control settings at different scopes. These configurations are:

  1. **System-level** Configuration (```--system```)
    - **Scope**: Affects all users on the machine and all repositories.
    - **Location**: The configuration file is stored in the Git system directory, typically located in:
      - Linux/macOS: /etc/gitconfig
      - Windows: C:\Program Files\Git\etc\gitconfig
    - **Usage**: This is useful for system-wide settings, but it requires **administrative** access to modify.
        ```bash
        git config --system user.name "System Admin"
        ```
  2. **Global-level** Configuration (```--global```)
    - **Scope**: Affects the current user across all repositories.
    - **Location**: Stored in the user's home directory:
      - Linux/macOS: ```~/.gitconfig``` or ```~/.config/git/config```
      - Windows: ```C:\Users\<username>\.gitconfig```
    - **Usage**: This is the most common level for personal settings, like name, email, editor, etc.
        ```bash
        git config --global user.name "Your Name"
        ```
  3. **Local-level** Configuration (```default```)
    - **Scope**: Affects only the specific repository in which the configuration is set.
    - **Location**: Stored in the ```.git/config``` file inside the repository directory.
    - **Usage**: Used for repository-specific configurations, such as different credentials, branch settings, or specific hooks.
        ```bash
        git config user.name "Repository-Specific Name"
        ```
* ```Git``` provides several options for configuring credentials for every "repo" operations, each with different ways to handle  **caching** or **storage**. 

  * The Git credential ```cache``` runs a daemon process which caches your credentials in memory for a short period (default: 15 minutes) and hands them out on demand.
    ```bash
    git config --global credential.helper 'cache --timeout=3600'
    ```

  * The Git Credential Manager (```GCM```) is a cross-platform, secure way to handle credentials for Git repositories. It integrates with the native credential storage systems on different platforms:

    * On Windows, it uses the Windows Credential Manager.
    * On macOS, it integrates with the macOS Keychain.
    * On Linux, it uses the Gnome Keyring or a similar system.

    ```bash
    git config --global credential.helper manager
    ```

* Configure the author name and email address to be used with your commits:

  ```bash
  git config --global user.name "LAST_NAME.FIRST_NAME"
  git config --global user.email "WORK_EMAIL"
  ```
* Normalize your line endings:

  ```bash
  git config --global core.autocrlf false
  ```
* Colorized output:

  ```bash
  git config --global color.ui auto
  ```
* Setup KDiff as the merge tool:

  ```bash
  git config --global --add merge.tool kdiff3
  git config --global --add mergetool.kdiff3.path "C:/Program Files/KDiff3/kdiff3.exe"
  ```
* Change the default core editor from vim to VSCode:
  ```bash
  git config --global core.editor 'code --wait'
  ```
* Setup proxies to work with Git:
    * Option1: Git Proxy without Cntlm: (Not secure as our credentials are being sent "over the wire")
      ```bash
      git config --global http.proxy http://MyUserID:MyPassword@www-proxy.company.com:80
      git config --global https.proxy http://MyUserID:MyPassword@www-proxy.company.com:80
      ```
    * Option2: Git Proxy w/ Cntlm: (Safe since our passwords are encrypted. Generic enough to be used with a host of tools like NPM, Git, AWS, Eclipse, Docker etc.)
      ```bash
      git config --global http.proxy http://localhost:3128
      git config --global https.proxy http://localhost:3128
      ```
* Display current settings

  ```bash
  git config --list
  ```

## Further Read

More info here: <http://git-scm.com/docs/git-config>
