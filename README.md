# Ansible/Linux Practice

### How to: Automate installing Wordpress on Ubuntu 14 using Ansible

### A couple disclaimers: 

* Ubuntu 14.04 is being used here. You should be able to follow along with other Linux distributions, however
* Root/sudo privileges are necessary

## Requirements:
* 2 Linux distributions: a build server (where you'll download Ansible) and a node (where WordPress will be downloaded) In this tutorial, both will be running Ubuntu 14.04
* Basic Linux knowledge (but I'll walk you through it)

## Step 1: Setting up an SSH RSA Key
First, we'll set up an [SSH RSA key](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) on your **build server** (most likely just your home computer), which is essentially an encryption algorithm allowing you to [securely communicate](https://help.ubuntu.com/community/SSH/OpenSSH/Keys) between the two machines. In your home directory, type in:
```ssh-keygen -t rsa```

Note: **make sure to not do this as the root user**. It will be stored in the root directory instead of your home directory (i.e. - /root/.ssh instead of /home/user/.ssh) and unless you log into ssh using root, your key will not be found. 

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



## Step 2: Ansible Basics (like, REALLY basic)

So... what is Ansible?

Essentially, it's an open source configuration management software that [improves the scalability, consistency, and reliability of your IT environment](https://cloudacademy.com/blog/what-is-ansible/) and can probably be described in another 50 buzzwords too. It's built on Python, and it utilizes SSH, which is why we just went through all that trouble setting up an SSH RSA key!  

Courtesy of [Ansible for DevOps](https://www.ansiblefordevops.com/), Ansible aims to be:
* Clear - a simple YAML syntax
* Fast - fast and easy to set up 
* Complete - 'batteries included'
* Efficient - no extra SW needed
* Safe - uses SSH


Think of it this way: if you have 1,000 servers where you want each to be set up in an identical manner, would you rather SSH into each of these servers, or use Ansible to automate the entire process for you?


## Step 3: Installing Ansible
We're almost to the good stuff. I promise.


If you have made it this far, I assume you already know how to download and install packages through the command line, but if that isn't the case, [check out this video](https://www.youtube.com/watch?v=EKmLXiA4zaQ).

So let's get started with typing this into your command line to download Ansible:

```
sudo apt-get install ansible -y
```

Ansible is built on Python (and is completely open source [if you want to check it out](https://github.com/ansible/ansible)), so you won't need to download Python as it comes preinstalled on Ubuntu. If you're on another distribution, it's probably a good idea to check if Python downloaded. Type this in to confirm:

```which python```

If `/usr/bin/python` populates, you are good to go!

Now, we have Ansible installed. Let's check out what version it is just to get familiar with the syntax.

```ansible --version```

Your output should look something like: 
```
ansible 1.5.4
```













**In this tutorial, we are focusing on two major components of Ansible:**
* Inventory/host file
* YAML playbooks

There is (obviously) way more to it than this, but for the sake of simplicity and my own limited knowledge, we will focus here. 

**Inventory**
[Official Documentation Here](http://docs.ansible.com/ansible/latest/intro_inventory.html)



## Resources
This tutorial is mostly a combination of a few different Digital Ocean and Linux Academy tutorials with my own feedback added in. Basically it's a dumbed down version with my trial and error feedback during the process of learning Ansible :-) 

Here are the links:
* [Official Ansible Documentation](http://docs.ansible.com/)
* [How to set up SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
* [How to automate install WP on Ubuntu 14](https://www.digitalocean.com/community/tutorials/how-to-automate-installing-wordpress-on-ubuntu-14-04-using-ansible)
* [Ansible for DevOps GitHub Repo](https://github.com/geerlingguy/ansible-for-devops)
* [Linux Academy](https://linuxacademy.com/)
