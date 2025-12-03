# Environment Setup
This is a guide to install WSL, integrate it in VSCode, and other things...

## 1. Install WSL

### Install command
  Open PowerShell in administrator mode, and write
```bash
wsl --install
```

### Set username and password
- This User Name and Password is specific to each separate Linux distribution that you install
- Please note that whilst entering the Password, nothing will appear on screen. This is called blind typing
- Once you create a User Name and Password, the account will be your default user for the distribution and automatically sign-in on launch
- This account will be considered the Linux administrator, with the ability to run sudo (Super User Do) administrative commands
- Each Linux distribution running on WSL has its own Linux user accounts and passwords. You will have to configure a Linux user account every time you add a distribution, reinstall, or reset
  
*To change or reset your password, open the Linux distribution and enter the command: `passwd`*
    
### Update and upgrade packages
```bash
sudo apt update && sudo apt upgrade
```

### File storage
To open your WSL project in Windows File Explorer, enter: `explorer.exe .`

*Be sure to add the period at the end of the command to open the current directory*

## 2. Set Up WSL in VSCode
Why?
- develop in a Linux-based environment
- use Linux-specific toolchains and utilities
- run and debug your Linux-based applications from the comfort of Windows
- use the VS Code built-in terminal to run your Linux distribution of choice

nice.

### Install VSCode and GIT
...

### Install extension pack
Install the [Remote Development extension pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)

### Update Linux Distribution
```bash
sudo apt-get update
sudo apt-get install wget ca-certificates
```

### Open a WSL project in Visual Studio Code
From the command-line: Open the distribution's command line and enter: `code .`

From VS Code: Use the shortcut: `Ctrl + Shift + P` in VS Code to bring up the command palette. If you then type `WSL` you will see a list of the options available

### Extensions inside of VS Code WSL
The WSL extension splits VS Code into a “client-server” architecture, with the client (the user interface) running on your Windows machine and the server (your code, Git, plugins, etc) running "remotely" in your WSL distribution.

When running the WSL extension, selecting the 'Extensions' tab will display a list of extensions split between your local machine and your WSL distribution.

<img width="586" height="320" alt="image" src="https://github.com/user-attachments/assets/5b5c01bb-5890-4942-a5ea-517a28572327" />

## 3. Try Github
