# Linux Deploy

1. Linodes
2. windows subsystem - ssh -bash shell
    - ssh root@IP
3. apt-get update && apt-get upgrade
4. hostnamectl set-hostname django-server
5. nano etc/hosts
    - IP : django-server
6. adduser USER
7. adduser USER sudo
    - don't use root for messing anything
8. ~/ is your home directory. Typically, it's /home/USER/.
9. Directorys starting with a dot . are considered to be hidden. That means:  
    ~/somedirectory and ~/.somedirectory are different directories. That is if ~/somedirectory existed and you did mkdir ~/.somedirectory, you won't fail with a File Exists message.  
    The ls command will not show those directories starting with the .  
    The ls -a will show both directories  
    
10.  
local machine
- ssh-keygen -b 4096(Generate SSH keys)
local machine
- scp ~/.ssh/id_rsa.pub ddx@172.105.125.231:~/.ssh/authorized_keys
* linode
- sudo chmod 700 ~/.ssh/
- sudo chmod 600 ~/.ssh/*
[SSH 公開金鑰認證：不用打密碼登入 Linux 設定教學，安全又方便](https://blog.gtwang.org/linux/linux-ssh-public-key-authentication/)

# Linux Bash
* pwd  
    print working directory
* ls
    