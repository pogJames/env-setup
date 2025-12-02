# Environment Setup

## 1. Install WSL command
  Open PowerShell in administrator mode, and write
```bash
wsl --install
```

## 2. Set username and password
- This User Name and Password is specific to each separate Linux distribution that you install and has no bearing on your Windows user name.
- Please note that whilst entering the Password, nothing will appear on screen. This is called blind typing. You won't see what you are typing, this is completely normal.
- Once you create a User Name and Password, the account will be your default user for the distribution and automatically sign-in on launch.
- This account will be considered the Linux administrator, with the ability to run sudo (Super User Do) administrative commands.
- Each Linux distribution running on WSL has its own Linux user accounts and passwords. You will have to configure a Linux user account every time you add a distribution, reinstall, or reset.
  * To change or reset your password, open the Linux distribution and enter the command: passwd.
## 3. Update and upgrade packages
```bash
sudo apt update && sudo apt upgrade
```
