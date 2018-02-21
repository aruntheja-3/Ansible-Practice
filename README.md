# Ansible/Linux Practice

### How to: Automate installing LAMP stack on Ubuntu 14 using Ansible

[Step 1: Setting up an SSH RSA Key](https://github.com/lucassha/Ansible-Practice#step-1-setting-up-an-ssh-rsa-key)

[Step 2: Ansible Basics (like, REALLY basic)](https://github.com/lucassha/Ansible-Practice#step-2-ansible-basics-like-really-basic)

[Step 3: Installing Ansible](https://github.com/lucassha/Ansible-Practice#step-3-installing-ansible)

[Step 4: Understanding hosts and playbooks](https://github.com/lucassha/Ansible-Practice#step-4-understanding-hosts-and-playbooks)

[Step 5: Modifying your hosts file](https://github.com/lucassha/Ansible-Practice#step-5-modifying-your-hosts-file)

[Step 6: Setting up your first playbook](https://github.com/lucassha/Ansible-Practice#step-6-setting-up-your-first-playbook)

[Step 7: Setting up your LAMP stack playbook](https://github.com/lucassha/Ansible-Practice#step-7-setting-up-your-lamp-stack-playbook)

### A couple disclaimers: 

* Ubuntu 14.04 is being used here. You should be able to follow along with any distribution though
* Root/sudo privileges are necessary
* This tutorial is aimed towards Linux and Ansible beginners

## Requirements:
* 2 Linux distributions: a build server (where you'll download Ansible) and a node (where the LAMP stack will be downloaded) In this tutorial, both will be running Ubuntu 14.04

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

After you've entered your password to confirm you want to add the key(s), let's output the contents of `authorized_keys` file to make sure your key was added. From your home directory, type `cat ~/.ssh/authorized_keys` (or wherever your .ssh directory is located). You should check for your host's user info. It will probably be listed at the very end of about a paragraph's worth of gibberish. 

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

If `/usr/bin/python` populates, you are good to go! If not, you know what to do.

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

There is (obviously) way more to it than this, but for the sake of simplicity and my own limited knowledge, we will focus here, as it's all that is required to download the LAMP stack. 


**Inventory**

This is your `hosts` file. Essentially, you add entries into this file for every server you want to manage with Ansible. You group them together which is how Ansible determines which servers to SSH into. Here's a quick example of what you might put in the `hosts` file:

```
[portland]
user@154.263.18.0
user.myLabServer.com
```

Don't worry, I'll explain this more in the next step; I just want to give you a quick introduction.

**Playbooks**

So what the heck is a playbook and YAML file? Well, the playbooks themselves are [YAML files](https://en.wikipedia.org/wiki/YAML), which can be a little funky at first but are at least human-readable. Here's an example of a playbook below, in which we'll be writing something similar to download all the necessary packages.([thanks Digital Ocean for the sample](https://www.digitalocean.com/community/tutorials/how-to-create-ansible-playbooks-to-automate-system-configuration-on-ubuntu)). Same as above... don't panic. We'll work through this slowly.

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

Okay, now we know a little bit about both the playbooks and inventory. Let's check out the `hosts` file first in order to set up the node  we'll SSH into. 

Both of these files (`ansible.cfg` and `hosts`) are located at `/etc/ansible/`. Just in case you're new to Linux, /etc/ is generally where all configuration management files are located, and Ansible is no exception. 

So, let's take a look at the directory:

```
cd /etc/ansible/
```

If you type in `ls`, you'll see two files: `ansible.cfg` and `hosts`.

Take a peek into the contents of `hosts` using `cat hosts` and you'll see some examples on how to set up a collection of hosts among other things. This is a good file to reference in the future, so we're going to `mov` it in the directory for your reference later and create a new `hosts` file for our own use. 

```
mov hosts hostsOriginalExps
```
Now let's create the new `hosts` file. Any text editor will work, so feel free to use eMacs or Nano instead. But we all know Vim is the best :-)

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

You'll get 1 of these 2 responses below. If it's green, woohoo! If it's red, you probably have some SSH issues. First step should be to just trying `ssh user@54.144.223.213` to see if that works. If it doesn't, Google is really your best bet, as that's outside the scope of this tutorial. 

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

## Step 6: Setting up your first playbook

Hopefully you're still with me. We. Are. Almost. There.

Let's first take a look at a basic playbook which updates your cache in Ubuntu. It's equivalent to `sudo apt-get update`. Here's the code below and then we'll walk through it.

```
---  #apt-get update
- hosts: ubuntu
  user: user
  sudo: yes
  connection: ssh
  gather_facts: yes
  tasks:
  - name: apt-get update
    apt: update_cache=yes
```

A few technicalities to note before explaining:
* These YAML files are VERY sensitive to spacing. You should only use spaces. Do. Not. Use. Tabs.
* The `---` at the top is always required.

And the basics of your syntax:

* Anything that starts with a `-` is considered to be a list item. 
* Items that are formatted like `hosts: ubuntu` essentially operate as dictionaries. 

Honestly, you don't REALLY need to worry about the syntax too much. As you write playbooks, it will start to make sense. 


**Now, let's walk through the playbook**

First up, we have our dashes `---` and then anything after `#` is a comment, so we are just saying this playbook is equivalent to `sudo apt-get update`

`- hosts: ubuntu`

This is from your hosts file. As I labeled mine [ubuntu], that's what I put.

`user: user` 

On my build server, I am the user `user`. If you want to see who you are, type in `whoami`.

`sudo: yes`

We are installing files on the node, so we'll need sudo/root privileges.

`connection: ssh`

We want to connect via SSH. Ansible can also connect via Paramiko, which is something like SSHv2? (Not really sure there)

`gather_facts: yes`

Gather facts is set by default to **yes**, so it's not technically necessary to write that in. If you select no, it will also be a bit faster, but we're only working with 1 node so that doesn't matter. Here's more info on [gather_facts](http://docs.ansible.com/ansible/latest/setup_module.html).

```
tasks: 
- name: apt-get update
  apt: update_cache=yes
```

The `tasks` portion is where you're actually telling Ansible what to do. `name` can be anything, so you don't have to write `apt-get update` here. You could write `UPDATING MY REMOTE SERVER YAY` and it won't matter. 

And there you have it! Your first playbook. 

If you already haven't, copy the playbook above and open Vim/Nano/eMacs, paste it, and save it as `aptUpdate.yml`.

Now, let's run this badboy and see what happens. Type this below:

`ansible-playbook aptUpdate.yml`

It should take a few seconds as it is updating the node's cache, and then this response will populate (if successful):

```
PLAY [ubuntu] ***************************************************************** 

TASK: [apt-get update] ******************************************************** 
ok: [user@35.168.2.79]

PLAY RECAP ******************************************************************** 
user@35.168.2.79           : ok=1    changed=0    unreachable=0    failed=0   
```

If it fails, you're going to get something like this:

```
PLAY [ubuntu] ***************************************************************** 

TASK: [apt-get update] ******************************************************** 
fatal: [user@54.144.223.213] => SSH encountered an unknown error during the connection. 
We recommend you re-run the command using -vvvv, 
which will enable SSH debugging output to help diagnose the issue

FATAL: all hosts have already failed -- aborting

PLAY RECAP ******************************************************************** 
           to retry, use: --limit @/home/user/aptUpdate.retry

user@54.144.223.213        : ok=0    changed=0    unreachable=1    failed=0 
```

**NOTE**:

If you run into issues, it may be because your user doesn't have sudo privileges. If that's the case, I did the following. Type in `sudo visudo` and then at the very bottom of the file add this: `user    ALL=(ALL) NOPASSWD: ALL`, where `user` = your user name. Please note that I'm fairly certain this is a major security risk with, so it's not necessarily a great idea but a temporary fix.

## Step 7: Setting up your LAMP stack playbook

Alright, nice work if you've made it this far! We are now going to use this playbook below to download all the necessary packages to run and host our site. I'll walk you through the new parts, but you're on your own for what we have already seen.

```
---
- hosts: ubuntu
  sudo: yes
  user: user
  tasks:
  - name: install all packages
    apt: name={{item}} state=present update_cache=yes
    with_items:
    - apache2
    - mysql-server
    - php5-mysql
    - php5
    - libapache2-mod-php5
    - php5-mcrypt
    - python-mysqldb
  - name: start apache2
    service: name=apache2 state=started
```

We'll start at the top and work down:

`apt: apt: name={{item}} state=present update_cache=yes`

Each part of these is new, out side of `apt`, so let's start with `name={{item}}`. Anything with these brackets means it's a variable, and it works with `with_items`, which I'll explain in just a second. 

`state=present` 

This is the equivalent to saying 'installed'. If you put `state=absent` then all of the packages below would be uninstalled instead. [More info here](http://hakunin.com/six-ansible-practices)


`with_items` is essentially an iterative loop. It will loop through the below items and download each, based off of `apt:` above with the `{{item}}` variable. You'll see the results in the Ansible output here in just a minute.

And finally: `service: name=apache2 state=started`. This is stating that Apache2 should be started and be running. 

If you want to check, type in `http://IPaddress` and the Apache2 Ubuntu Default Page should populate. 

**NOTE**:

If you are having troubles with Apache2 not starting, there's a decent chance it's because you have Nginx installed (and possibly running), and it may be taking up your port 80. Try this on the node below, and then re-run the playbook to see if it works. Huge thanks to user [arnoldkarani](https://www.digitalocean.com/community/users/arnoldkarani) for this tip in the Digital Ocean comments because I WAS GETTING VERY FRUSTRATED HERE.

```
sudo apt-get remove nginx nginx-common # Removes all but config files.
sudo apt-get purge nginx nginx-common # Removes everything.
sudo apt-get autoremove # Use this in order to remove dependencies used by nginx which are no longer required.
```

Now let's save this file as `install.yml` and run it.

`ansible-playbook install.yml`

If successful, your output should look something like this below. Note we did not specify gather_facts: yes, and it still ran regardless. 

```
PLAY [ubuntu] ***************************************************************** 

GATHERING FACTS *************************************************************** 
ok: [user@35.168.2.79]

TASK: [install all packages] ************************************************** 
ok: [user@35.168.2.79] => (item=apache2,mysql-server,php5-mysql,php5,libapache2-mod-php5,php5-mcrypt,python-mysqldb)

TASK: [start apache2] ********************************************************* 
ok: [user@35.168.2.79]

PLAY RECAP ******************************************************************** 
user@35.168.2.79           : ok=3    changed=0    unreachable=0    failed=0  
```


### Voila! Your packages are installed.

Now you can configure them however needed. 

## Resources
This tutorial is mostly a combination of a few different Digital Ocean and Linux Academy tutorials with my own feedback added in. Basically it's a dumbed down version with my trial and error feedback during the process of learning Ansible :-) 

Here are the links:
* [Official Ansible Documentation](http://docs.ansible.com/)
* [How to set up SSH keys](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)
* [How to automate install WP on Ubuntu 14](https://www.digitalocean.com/community/tutorials/how-to-automate-installing-wordpress-on-ubuntu-14-04-using-ansible)
* [How to install LEMP stack](https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04)
* [Ansible for DevOps GitHub Repo](https://github.com/geerlingguy/ansible-for-devops)
* [Linux Academy](https://linuxacademy.com/)
