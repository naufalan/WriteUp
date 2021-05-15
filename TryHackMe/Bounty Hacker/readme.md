# WriteUp : Simple CTF

## Reconnaissance

- **Port & Service Scanning**

    To answer the first question :

    > Find open ports on the machine

     I need to know which service was run on this machine, so i use Nmap to perform port & service scan.

    ```bash
    nmap 10.10.98.83 -sV -T3 --top-ports=20
    ```

    Command above give me answer about which service available on this machine

    ```bash
    Nmap scan report for 10.10.98.83
    Host is up (0.22s latency).

    PORT     STATE    SERVICE       VERSION
    21/tcp   open     ftp           vsftpd 3.0.3
    22/tcp   open     ssh           OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
    23/tcp   filtered telnet
    25/tcp   filtered smtp
    53/tcp   filtered domain
    80/tcp   open     http          Apache httpd 2.4.18 ((Ubuntu))
    110/tcp  filtered pop3
    ...
    ```

- **Inspecting FTP**

    We can enumerate FTP service was running on this box, we can try to connect it with default FTP client on Kali

    ```bash
    ftp 10.10.98.83
    ```

    In a weak configuration of FTP, we can easily login with anonymous as username & password, i try on this box and that's works 

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled.png)

    After got login, we can enumerate all item was available in this box, we found a task.txt file which is likely used to answer the next question **Who wrote the task list?** to open this task.txt file first we need to download it to our machine with `get task.txt` command and open the file 

    ```bash
    cat task.txt    
    1.) Protect Vicious.
    2.) Plan for Red Eye pickup on the moon.

    -lin
    ```

    From that's file, we can answer the question, the writer of the task is **lin.** After enumerate the task.txt files, we move into locks.txt file, download and open the file and we can find it's containing something like all password combination for SSH service

    ```bash
    cat locks.txt 
    rEddrAGON
    ReDdr4g0nSynd!cat3
    Dr@gOn$yn9icat3
    ...
    ```

- **Inspecting SSH**

    To gaining shell via SSH, we must known two information, username & password. To find all username available, we can bruteforce and trying to login with wrong password, if the username is exist, the error message should notice that the password is wrong, first attempt i use the lin (the answer of previous question) as username and wrong password (123)

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%201.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%201.png)

    Wrong password, indicating the username exist

    After got the username, we can find the password by bruteforcing SSH with Hydra. In this box i use the locks.txt file as wordlist and lin as username

    ```bash
    hydra -l lin -P locks.txt 10.10.98.83 ssh -t 20
    ```

    Because the file locks.txt itself not long enough, the time to bruteforce just about 5 second and finally we can find the password for gaining shell with SSH

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%202.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%202.png)

## Exploitation

- **Get User Flag**

    After get the **username** & **password**, we can login and get shell via SSH service

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%203.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%203.png)

    Get shell with SSH

    We can enumerate current working directory with simple linux command `pwd` and list all available files or directorys in current directory

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%204.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%204.png)

    Enumerate user home directory

    `user.txt` likely containing flag which is to answer the question, so we can open it and copy the flag

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%205.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%205.png)

    Get user flag

    After getting the user flag, we move into the nex steps to get the root flag, which required additional work..

- **Get Root Flag**

    Likely the location of root flag is in root directory, but we can't see this directory due to not enough the privilege, so we must escalate our privilege from user lin to root user. I will check all the privilege of the user using command `sudo -l`  and we get the tar is likely run under root privilege 

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%206.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%206.png)

    /bin/tar run under root privilege

    Then we can check to the site [https://gtfobins.github.io/](https://gtfobins.github.io/) which list all binary that can be used to escalate privilege, and from that site we can use tar binary to escalate our privilege from lin to root with this command 

    ```bash
    sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
    ```

    After run that command, we get the root shell and list the root directory

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%207.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%207.png)

    Root directory

    we can see just one file inside root directory, which likely contain the root flag, we open the file and booomm..we get the root flag

    ![WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%208.png](WriteUp%20Simple%20CTF%20fe713a355de245218f4b3782f739e4b7/Untitled%208.png)

    Get the root flag

    That's its, the CTF is completed and we can learn much about this vulnerability of the box

## Countermeasure

We can learn about secure configuration of FTP and how to manage password of our machine account. We can summary the countermeasures to secure our local machine to prevent the attack like this :

- Never ever allow anonymous login to FTP service , although we can't store any confidential files on our FTP directory
- Our password should not stored in local machine itself, we must store it in another services like Cloud or using password manager/vault like LastPass or BitWarden
- A binary application should not run under root privilege, if we must run it under root privilege we must let the root account to run that, its much safer and separating responsibility of managing important configuration, the usual user account should not changes an importan linux settings

That's its, beside the CTF is challenging games, its also teach our about how an small vulnerabilty could be a doorway for an attacker to pwn our system
