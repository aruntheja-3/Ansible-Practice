# Ansible/Linux Practice

### How to: Automate installing Wordpress on Ubuntu 14 using Ansible

### A couple disclaimers: 

* Ubuntu 14.04 is being used here. You'll run into some issues running Ubuntu 16 with the PHP installations
* Root/sudo privileges are necessary to download packages

## Requirements:
* 2 Linux distributions: a host machine and a client machine (both running Ubuntu 14)
* Basic Linux knowledge (but I'll walk you through it)

## Step 1:
First, we'll set up an [SSH RSA key](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) on your **client machine**, which is essentially an encryption algorithm allowing you to [securely communicate](https://help.ubuntu.com/community/SSH/OpenSSH/Keys) between the two machines. In your home directory, type in:
```ssh-keygen -t rsa```

Note: make sure to **not** do this as the **root** user. It will be stored in the root directory instead of your home directory (i.e. - /root/.ssh instead of /home/user/.ssh)

You'll be prompted for a storage location. If you want this in your home directory, simply press enter to continue. Otherwise, select a directory where you want it stored. It will basically look like what I have pasted below. I only hit [Enter] through the file and passphrase location, so my file is saved in my home directory for user user

```user@user-MacBookPro:~$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
3c:3b:ac:4e:b8:89:ae:5b:30:ce:da:48:79:85:a8:c9 user@user-MacBookPro
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|     . .         |
|    . . .        |
| . o . o         |
|  E o . S        |
|   o o . o       |
|. o o . +        |
|o+ o + . .       |
|*=+ o.o          |
+-----------------+
```

Now, your key is generated, and you'll want to put in the public key for the host server you will use. The command below will add the key to your host's `authorized_keys` folder, which most likely is located at: `/home/user/.ssh/authorized_keys`.  Essentially, you'll put in the IP address for the host machine. For example:

```
ssh-copy-id userName@IPaddress
```
or
```
ssh-copy-id user@54.209.33.104
```

After you've entered your password to confirm you want to add the key(s), let's output the contents of `authorized_keys` file to make sure your key was added. From your home directory, type `cat ~/.ssh/authorized_keys` (or wherever your .ssh directory is located). You should check for your host's user info. 

Now you can SSH into your host server without a password. Yay!


## Step 2: 
