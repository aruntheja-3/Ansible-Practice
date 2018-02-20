# Ansible/Linux Practice

### How to: Automate installing Wordpress on Ubuntu 14 using Ansible

### A couple disclaimers: 

* Ubuntu 14.04 is being used here. You should be able to follow along with other Linux distributions, however
* Root/sudo privileges are necessary
* This tutorial is aimed towards Linux beginners, so bear with me if you are a more advanced user

## Requirements:
* 2 Linux distributions: a build server (where you'll download Ansible) and a node (where WordPress will be downloaded) In this tutorial, both will be running Ubuntu 14.04

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

Before we download Ansible, it's important to know Ansible is built on Python (and is completely open source [if you want to check it out](https://github.com/ansible/ansible)), but you won't need to download Python as it comes preinstalled on Ubuntu. If you're on another distribution, it's probably a good idea to check if Python downloaded. Type this in to confirm:

```which python```

If `/usr/bin/python` populates, you are good to go!

So let's get started with typing this into your command line to download Ansible:

```
sudo apt-get install ansible -y
```

Now, we have Ansible installed. Woohoo! Let's check out what version it is just to start getting familiar with the syntax.

```ansible --version```

Your output should look something like: 
```
ansible 1.5.4
```

## Step 4: Understanding hosts and playbooks

**In this tutorial, we are focusing on two major components of Ansible:**
* Inventory/hosts file
* YAML playbooks

There is (obviously) way more to it than this, but for the sake of simplicity and my own limited knowledge, we will focus here, as it's all that is required to download the LAMP stack and host Wordpress. 


**Inventory**

This is your `hosts` file. Essentially, you add entries into this file for every server you want to manage with Ansible. You group them together which is how Ansible determines which servers to SSH into. Here's a quick example of what you might put in the `hosts` file:

```
[portland]
user@154.263.18.0
user.myLabServer.com
```

Don't worry, I'll explain this more in the next step; I just want to give you a quick introduction.

**Playbooks**

So what the heck is a playbook and YAML file? Well, the playbooks themselves are [YAML files](https://en.wikipedia.org/wiki/YAML), which can be a little funky at first but are at least human-readable. Here's an example of a playbook below, in which we'll be writing something similar to host WordPress ([thanks Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-create-ansible-playbooks-to-automate-system-configuration-on-ubuntu)). Same as above... don't panic. We'll work through this slowly.

```
---
- hosts: droplets
  tasks:
    - name: Installs nginx web server
      apt: pkg=nginx state=installed update_cache=true
      notify:
        - start nginx

  handlers:
    - name: start nginx
      service: name=nginx state=started
```

"[Playbooks are Ansible’s configuration, deployment, and orchestration language. They can describe a policy you want your remote systems to enforce, or a set of steps in a general IT process](http://docs.ansible.com/ansible/latest/playbooks.html)." Think of a playbook like a recipe in a cookbook. It lays out all the ingredients, details and necessary processes, and then Ansible will actually execute these.

Playbooks can be very advanced, but for us, our playbook will simply manage deployment to our lone remote machine.




## Step 5: Modifying your hosts file

Okay, now we know a little bit about both the playbooks and inventory. Let's check out the `hosts` file first in order to set up the node we'll SSH into. 

Both of these files are located at `/etc/ansible/`. Just in case you're new to Linux, /etc/ is generally where all configuration management files are located, and Ansible is no exception. 

So, let's take a look at the directory:

```
cd /etc/ansible/
```

If you type in `ls`, you'll see two files: `ansible.cfg` and `hosts`.

Take a peek into the contents of `hosts` using `cat hosts` and you'll see some examples on how to set up a collection of hosts among other things. This is a good file to reference in the future, so we're going to `mov` it in the directory for reference later and create a new `hosts` file for our own use. 

```
mov hosts hostsOriginalExps
```
Now let's create the new `hosts` file. Any text editor will work, so feel free to use eMacs or Nano instead.

```
sudo vim hosts
```

Now add in the node (basically, the IP address) you will want to use, which you can see below. [ubuntu] is the grouping name I used, but you could name it anything you want. Notice in the above example on hosts I put in [portland], so that could could be a group of servers residing in Portland. You'll use `[ubuntu]` later on to reference this specific grouping in the playbook. 

```
#this is your inventory file. comments start with #
[ubuntu]
user@IPaddress
user@154.263.18.0
```

Great! Our `hosts` file is now set up. Let's use `ping` to make sure the connection is actually established.

Type in:

```
ansible ubuntu -m ping
```

or

```
ansible all -m ping
```

You'll get 1 of these 2 responses below. If it's green, woohoo! If it's red, you probably have some SSH issues. First step should be to just trying `ssh user@54.144.223.213` to see if that works. Google is really your best bet though, as that's outside the scope of this tutorial. 

```
user@54.144.223.213 | success >> {
    "changed": false, 
    "ping": "pong"
}

OR

user@54.144.223.213 | FAILED => SSH encountered an unknown error during the connection. 
We recommend you re-run the command using -vvvv, 
which will enable SSH debugging output to help diagnose the issue
```

If you want to learn more, [here's the official documentation](http://docs.ansible.com/ansible/latest/intro_inventory.html)

## Step 6: Setting up your playbook

Hopefully you're still with me. We. Are. Almost. There.

Let's first take a look at a basic playbook which updates your cache in Ubuntu. It's equivalent to `sudo apt-get update`.



NOTE:


If you are having troubles with Apache2 not starting, there's a decent chance it's because you have Nginx installed (and possibly running), and it may be taking up your port 80. Try this on the node below, and then re-run the playbook to see if it works. Huge thanks to user [arnoldkarani](https://www.digitalocean.com/community/users/arnoldkarani) for this tip in the Digital Ocean comments.

```
sudo apt-get remove nginx nginx-common # Removes all but config files.
sudo apt-get purge nginx nginx-common # Removes everything.
sudo apt-get autoremove #After using any of the above commands, use this in order to remove dependencies used by nginx which are no longer required.
```




## Resources
This tutorial is mostly a combination of a few different Digital Ocean and Linux Academy tutorials with my own feedback added in. Basically it's a dumbed down version with my trial and error feedback during the process of learning Ansible :-) 

Here are the links:
* [Official Ansible Documentation](http://docs.ansible.com/)
* [How to set up SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
* [How to automate install WP on Ubuntu 14](https://www.digitalocean.com/community/tutorials/how-to-automate-installing-wordpress-on-ubuntu-14-04-using-ansible)
* [Ansible for DevOps GitHub Repo](https://github.com/geerlingguy/ansible-for-devops)
* [Linux Academy](https://linuxacademy.com/)
