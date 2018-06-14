---
layout: post
title: WIP - Leverage Ansible to administer a cluster of Raspberry Pi from a Unix or Windows machine
---

As I was looking into what skills I want to learn or enforce, I made up an app as a side project, that would make use of all these technologies in a coherent whole. Some of these things being Infrastructure related, I found better to go ahead and buy a bunch of Raspberry Pi (or is it Raspberries ?).

The first task I found would be interesting is to administer the Rasps with Ansible. By administer I mean :
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
* A control machine on Windows 10. It can be a Linux machine, a Mac, one of the Raspberry, maybe even an Android device.
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