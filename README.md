# Synology SSH key setup
How to setup SSH key authentication for your Synology

Content from Gudbrand Olimb's now deleted https://blog.golimb.com/2020/10/03/synology-ssh-key-authentication/

<p align="center"><img src="/images/icon.jpg"></p>

There is a lot of posts throughout the web on configuring SSH key authentication on Synology NAS many with some confusing and unnecessary steps such as:
- Modifying the RSAAuthentication and PubkeyAuthentication parameters in /etc/ssh/sshd_config
- Restarting the sshd service multiple times with sudo synoservicectl --reload sshd
- Changing permissions on various folders with chmod both root folders and user folders
- Unclear creation of ~/.ssh folder ending up under root

After reading several and many great blog posts and guides on this I've tried to summarise what is actually required to make SSH key authentication work with Synology NAS assuming you are coming from a clean setup without to much changes. Hopefully this summary will help you so you dont need to search google and go through the same x number of guides.

Now through this whole guide you will be in the **context of a specific user** who is included in the **Administrator group**.
- _You will_ ***not*** _be sudo or su to root user although sudo will be used to perform some actions_.
- The reason why you need to have a user specified in the administrator group is because it is only administrators who are allowed to login through SSH by default ref below.

<p align="center"><img src="/images/image-1.png"></p>

So lets get started with the basic steps

## 1. Prerequisite - Enable SSH on your Synology NAS
As shown in the picture above to enable SSH for your Synology NAS go to Control Panel -> Terminal & SNMP -> Terminal Tab -> Check Enable SSH Service and enter a port.

- It is highly recommended to use a custom port and not standard 22 as you then will get a lot of brute force attempts from robots and attackers scanning public IPs against port 22, this is if you are exposing your Synology NAS to the internet.
