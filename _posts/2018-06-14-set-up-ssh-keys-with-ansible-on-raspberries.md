---
layout: post
title: WIP - Leverage Ansible to administer a cluster of Raspberry Pi from a Unix or Windows machine
---

Wanting to go ahead with getting my Infrastructure skills up-to-date, I went ahead and bought a bunch of Raspberry Pi.

The first task I found would be interesting is to administer the *Rasps* with Ansible. By administer I mean :
* Manage the inventory of the machines available
* Harmonize OS configuration
* Deploy SSH keys for multiple users, in the most dynamic way possible. What I want is a solution to use at home, but knowledge that I can use in a big corporate environment.
* Install apps/roles, depending on which Pi/cluster
* Harmonize apps configuration
* Put Continuous Delivery after Continuous Integration. Basically, push something and have Ansible handle delivery/deployment.

Here I am going to detail the two primary objectives to get started : Install ansible, and deploy SSH keys.

## Hardware
* 3 Raspberry Pi B. I believe the model is not particulary important, except for the fact that I have out-of-the-box Wi-Fi with those.
* A bunch of accessories : 3 USB cables, one A/C to USB thingy, 3 Pi cases.
* A control machine on Windows 10. It could be a Linux machine, a Mac, one of the Raspberry, or even [apparently an Android device](https://gist.github.com/hirschnase/9c2a0c6334f55bfdb373cc14dcbdf167).
The important part here is that I wanted to use my Windows machine as the Control Machine, and keep my 3 raspberries for the cluster, 2 being too few for my tests (and money not being infinite !). 
It also brought up an opportunity to have a bit more fun since Windows is not supported for Control Machine.

## Getting the SD cards ready
If you're reading this, you most likely know that you need a SD card to operate the Pis, with an OS on it. I chose to use the standard, Raspbian, but since I never like it too easy, I wanted to prepare it 'headless', meaning without plugging the Pi to a screen/keyboard/mouse/etc.
As a matter of fact, I prepared the SD cards from my tablet, in my bed.

### Getting Raspbian
Nothing really surprising or challenging here, a Google search will direct you to the same link ; all you need is to download Raspbian ISO image on the official page :
https://www.raspberrypi.org/downloads/raspbian/

Since we want it headless, not point in taking the Desktop version ; I used the Lite version.

### Preparing the SD Card
First of all, of course, insert the SD card in your card reader !
To put the image on the SD card :
* Go to https://etcher.io. Click the **arrow** next to *Download[...]* and select *Download for Windows x64 Portable*, first choice in the list.
* Choose to directly open the file. **Etcher** is gonna download and launch.
* If you're a little paranoid on your data, you can use the gear icon on the upper right corner to access the options, and uncheck *Anonymously report [...]*
* In Etcher, click the *Select image* button, and choose the `.zip` file you downloaded for Raspbian. It's an `.img` file if you unzipped it.
* Choose the drive to use, the one with the SD card, on the second button. If it's already selected, do make sure it's the right one, as all content will be erased.
* Click on the *Flash!* button !
> Note : If after that you see the drive as empty in explorer, and you can't open it, try to take out the card and put it back in.

### Going headless
Now, as we're doing everything directly on the SD card, we need to set two things up : Network connectivity, and outside access.

#### Access the network
As I'm using a model with WiFi, there is a step to setup the SSID and password. If you have a model without Wi-Fi, you can either use a Wi-Fi adapter and do the same actions, or plug a cable and skip that step.
To allow the Pi to access the home Wi-Fi, we need to create a file on the boot partition, a file called `wpa_supplicant.conf`.
> If you're on Windows, the boot partition is just the root directory you see when you open the SD card.

Open this file with your favorite text editor and add the following contents. You'll need to replace the following :
* `fr` with your country code (unless you're in France !). You can find the code [here in the *Alpha-2 code* column](https://en.wikipedia.org/wiki/ISO_3166-1#Officially_assigned_code_elements)
* `MySsid` with your SSID, the name of your Wi-Fi network
* `MyPassword` with the password to access your network. This assumes you're using WPA-PSK as the encryption method.

{% highlight conf %}
country=fr
update_config=1
ctrl_interface=/var/run/wpa_supplicant

network={
  scan_ssid=1
  ssid="MySsid"
  psk="MyPassword"
}
{% endhighlight %}

On the first boot, the Pi will connect to the Wi-Fi and that file will be moved to `/etc/wpa_supplicant/`.

#### Setup fixed IP
If you want to be able to manage your cluster easily, I find it best to use fixed IP, if possible following each other.
To do so completely headless, we're going to specify a new MAC address for the Pi, and set our router/internet box to give it the IP we want.

First, check what IP are taken on your network, and try to find 3 (or more if you have more Pis) following each other that are free. In my case I took the following
* 192.168.1.20
* 192.168.1.21
* 192.168.1.22

For each of those, let's generate a new MAC address. To do so, we can use the following website : [MAC Address Generator](https://www.miniwebtool.com/mac-address-generator/). Choose the longest format (should be chosen by default) and press the *Generate* button.

Then, open with your text editor the file `cmdline.txt` located on the boot partition. It should be some arguments separated by spaces. At the end of it, add a space, and then `smsc95xx.macaddr=XX:XX:XX:XX:XX:XX` with the XX being your generated MAC address.

Finally, go to your router/internet box, find the DHCP settings, and make the right MAC address point to the IP you want. This part depends on your device and I can't tell you more about it.

#### Opening SSH access
In order to be able to access the Pis, we need to enable SSH, since it's disabled by default.
To enable it, it's quite easy, you just create a file, empty, named `ssh` (no extension) on the boot partition.
That's it !

### Run it !
The SD cards being ready, just put them in the Raspberry Pis, power them, and you should be all set.
After a few seconds, you should be able to SSH into them, with the IP you chose.

The default user is `pi` and the default password is `raspberry`.

## Ansible Control Machine
### Using a Windows machine
Since Windows is not supported as a Control Machine for Ansible, and since I really want to use my Windows machine, I decided to take advantage that I have a recent enough Windows 10 version, and use WSL (Windows Subsystem for Linux).
If you don't know what it is, I suggest you read [the official page](https://docs.microsoft.com/en-us/windows/wsl/about) but if you want to know the essence of it, it is kind of a Linux system inside your Windows, like a virtual machine without much of the overhead.

#### Activating WSL
To enable WSL, you need Windows 10, and it has to be minimum the build 16215 (Fall Creators Update) for these steps.
Open a Powershell prompt as Admin, enter the following, and press Enter :
{% highlight powershell %}
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
{% endhighlight %}

Restart the computer and WSL will be enabled.

#### Getting a Linux distribution
Just head to the *Microsoft Store*, somewhere in your Start Menu. Then look for the Linux distro you want ; in my case I chose Debian.
Just install it like any app and then you can launch it whenever you want through the start menu. It will then open a terminal inside the Linux.

In my case I also wanted to be able to SSH into the WSL, allowing me to have all the sessions (Raspberries and the control WSL machine) in one single place ([MobaXterm](https://mobaxterm.mobatek.net) in my case). 
To enable SSH in WSL :
* First run `sudo apt-get update` to get the OS updated
* Install OpenSSH : `sudo apt-get install openssh-server`
* Edit the file `/etc/ssh/sshd_config` and put the options `UsePrivilegeSeparation no` and `PasswordAuthentication yes`. For the first one, I had to uncomment it and change its default value `sandbox`. For the second one, I only had to uncomment.
* If you want you can also change the 22 port to something else. Otherwise you might have to authorise incoming 22 connections in your firewall ; I'm not using the Windows Firewall, but another, and didn't have any problem.
* Start SSH : `sudo service ssh start`
> You will have to start SSH with the `service` command at every Windows restart. It's possible though to [automate it](https://superuser.com/a/1112022/411969), which I didn't do because I was fine with the way it is.

Voilà, you're ready to SSH into your WSL !

### Installing Ansible (and PIP and Python)
Now, we have a working Control machine, whether it's through Windows and WSL, or directly a UNIX machine.
We need to have Python 2.6/7 or 3.5+ installed on the control machine. If you already have it (you can check with the `python -V` and `python3 -V` commands) you can skip this install.

#### Installing Python
To install Python and some other useful tools, type the following commands :
{% highlight shell %}
sudo apt-get install software-properties-common
sudo apt-get install python-setuptools python-dev libffi-dev
{% endhighlight %}

We can also install some additional tools :
{% highlight shell %}
sudo apt-get install libssl-dev git sshpass
{% endhighlight %}

Libssl and sshpass will be useful for ssh connections, and git will be useful when we want to get to CI/CD.

**Warning** : You will need Python installed on all your machines, including the ones you're going to control. Now that you're into it would be a good time to do it everywhere ! I suggest also installing PIP (see below).

#### Installing PIP
PIP is a Python package manager and will be useful to install Ansible and manage releases and upgrades. Installing PIP is as easy as running this one command :
{% highlight shell %}
sudo easy_install pip
{% endhighlight %}

#### Installing Ansible
Finally, now that we're all set, installing the latest release of Ansible is also as easy as one command, using PIP :
{% highlight shell %}
sudo pip install ansible
{% endhighlight %}

Then you can check that Ansible is installed with the following :
{% highlight shell %}
ansible --version
{% endhighlight %}

This should give you the version number, and a little summary of some configuration.

### Initial Ansible configuration
#### Configuration file
If you ran the `ansible --version` command, you may have noticed a line saying `config file = None`. We're going to create the config file, which will allows us to tweak Ansible as we need.
Let's create an empty file and open it :
{% highlight shell %}
sudo nano /etc/ansible/ansible.cfg
{% endhighlight %}

To populate that file, head to [this address](https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg), copy all, and paste it in the file. Save, exit, voilà !
Everything is commented in the file, but it's a good boilerplate and it's useful to know the default values for everything.

#### Inventory
To get started, we need an inventory of machines for Ansible to administer them.
The inventory can take several forms and can be dynamically made with several files, use cloud-specific features... Here it will be the most basic form, which is a single file, since all we need to put is 3 little Pis. The usual file format used in examples is INI, but I chose to go with another officially supported format : YAML. The goal is to be coherent with the playbooks (which we'll see later) since they use YAML, and I also find YAML more complete than INI.

##### Recap of the machines
In my case I have three Pi (I put the names on little post-it on them), with consecutive addresses :

| Name    | IP           | Role        |
|---------|--------------|-------------|
| TheGood | 192.168.1.20 | Master node |
| TheBad  | 192.168.1.21 | Worker node |
| TheUgly | 192.168.1.22 | Worker node |

The 3 of them together will be referenced as **TheCluster**.

##### Creating the inventory file with the hosts
By convention the inventory is named `hosts` (INI) or as it will be the case here, `hosts.yml` (YAML). We'll place it also in the `/etc/ansible` directory.
Create it by typing :
{% highlight shell %}
sudo nano /etc/ansible/hosts.yml
{% endhighlight %}

Then in my case, the content is the following. You can find the official example also [here](https://github.com/ansible/ansible/blob/devel/examples/hosts.yaml). You should read it in any case, as it contains the guidelines and valuable information on how to constitute the file.
{% highlight yaml %}
all:
  TheCluster:
    hosts:
      192.168.1.2[0-2]:
  children:
    masters:
      hosts:
        TheGood:
          ansible_host: 192.168.1.20
          ansible_port: 22
          ansible_user: pi
          ansible_ssh_pass: REDACTED
    workers:
      hosts:
        TheBad:
          ansible_host: 192.168.1.21
          ansible_port: 22
          ansible_user: pi
          ansible_ssh_pass: REDACTED
        TheUgly:
          ansible_host: 192.168.1.22
          ansible_port: 22
          ansible_user: pi
          ansible_ssh_pass: REDACTED
{% endhighlight %}

A few precisions :
* The hosts are present both in the `TheCluster` group and in the subgroups (`masters` and `workers`)
* In the cluster definition you can see I used a range, but you need your IP to follow each other for that
* I make Ansible connect with the `pi` user, which is a sudoer
* I use, for now, password authentication and removed my passwords :)

You can also check [this example](https://github.com/confluentinc/cp-ansible/blob/master/hosts.yml), that shows a more complete file, with groups related to domain of action for the hosts.

That `hosts.yml` file is now enough for Ansible to contact the 3 machines and execute any playbook. Let's try it out, shall we ?