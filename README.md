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

## 2. Prerequisite - Creation of SSH key pair
To use SSH key authentication we will need to generate a SSH key pair (one privateKey, one publicKey). The publicKey will be shared with and stored in the Synology NAS SSH "authorized keys" while the privateKey will be used to prove our identity as it will correspond to the publicKey.

- **Windows**
   - If you are on Windows I recommend downloading puttygen to generate the keys, its very quick and user friendly, see the link below for a guide on creation of RSA key.
https://www.ssh.com/ssh/putty/windows/puttygen

- **Mac**
   - Open a terminal, navigate to a folder and run below to generate a public and private key
      - `ssh-keygen -t rsa -b 4096 -C "user@domain.com"`
   - Go here if you want to read up some more: https://www.ssh.com/ssh/keygen/

## 3. Prerequisite - Copy the publicKey
Open the created keyname.pub and copy the content to a text editor or similar. The public key should start on ssh-rsa and a lot of look like below, beware there is no new line here, it is all in one line (this is also important for later).
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSkT3A1j89RT/540ghIMHXIVwNlAEM3WtmqVG7YN/wYwtsJ8iCszg4/lXQsfLFxYmEVe8L9atgtMGCi5QdYPl4X/c+5YxFfm88Yjfx+2xEgUdOr864eaI22yaNMQ0AlyilmK+PcSyxKP4dzkf6B5Nsw8lhfB5n9F5md6GHLLjOGuBbHYlesKJKnt2cMzzS90BdRk73qW6wJ+MCUWo+cyBFZVGOzrjJGEcHewOCbVs+IJWBFSi6w1enbKGc+RY9KrnzeDKWWqzYnNofiHGVFAuMxrmZOasqlTIKiC2UK3RmLxZicWiQmPnpnjJRo7pL0oYM9r/sIWzD6i2S9szDy6aZ user@domain.com
```

## 4. SSH into your NAS
Now that we have a key pair, we have enabled SSH on the Synology NAS lets log in to configure the SSH authorized_keys (= our generated public key)

Open a terminal and ssh into the server with your admin-user, ip and custom port: 

```
ssh {admin-user}@{nas-ip-or-host} -p {specifiedCustomPort}
```

Now run `pwd` command to verify your are in the {admin-user} user directory. 
- The result should be: ***/volume1/homes/{admin-user}***

## 5. Creation of .ssh directory and authorized_keys file
Now in the {admin-user} directory create a directory named **.ssh**

```
mkdir .ssh
```

Now navigate to the .ssh folder

```
cd .ssh
```

Now run the `pwd` command to verify you are in the right location) lets create a authorized_keys file.
  - The result should be ***/volume1/homes/{admin-user}/.ssh***

Next create a authorized_keys file.

```
vi authorized_keys
```

This will take you into the vi program interface for adding content.

- Press **i** to enable inserting text.
- Paste your public key from step 3.
   - Ensure you paste your public key on one line only, no new line and remember the spaces.
- Press **esc** to enter the vi program interface.
- Press semicolon **:** key.
- Type **wq!** and press enter to save the file.

Now lets verify the file is created with the `ls` command.
  - The result should be ***authorized_keys**

Now lets verify the public key in the file with the command `more authorized_keys`

The result should look like:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDSkT3A1j89RT/540ghIMHXIVwNlAEM3WtmqVG7YN/wYwtsJ8iCszg4/lXQsfLFxYmEVe8L9atgtMGCi5QdYPl4X/c+5YxFfm88Yjfx+2xEgUdOr864eaI22yaNMQ0AlyilmK+PcSyxKP4dzkf6B5Nsw8lhfB5n9F5md6GHLLjOGuBbHYlesKJKnt2cMzzS90BdRk73qW6wJ+MCUWo+cyBFZVGOzrjJGEcHewOCbVs+IJWBFSi6w1enbKGc+RY9KrnzeDKWWqzYnNofiHGVFAuMxrmZOasqlTIKiC2UK3RmLxZicWiQmPnpnjJRo7pL0oYM9r/sIWzD6i2S9szDy6aZ user@domain.com
```

## 6. Often confused step - setting correct permissions
Now often at this point this is where a lot of confusion occurs when trying to do SSH authentication with Synology NAS. A lot of this confusion occurs because the {admin-user} home directory by default allows any access which the sshd SSH daemon considers insecure and then prevents SSH key authentication from occurring.

**Default permissions of users home folders is 777 / rwxrwxrwx**
- Users home folder = /volume1/homes/{username}
- In this case home folder = /volume1/homes/{admin-user}

<p align="center"><img src="/images/image-2.png" width="500" height="266"></p>

What we need to do is to change the permissions to below:

<p align="center"><img src="/images/image-3.png" width="500" height="266"></p>

This can be done by running:
```
sudo chmod 755 /volume1/homes/{admin-user}
```
There are some comments that changing the user home permissions might not be the best solution to resolve this due to security or the fact that a Synology update might change this later.
- The first case on security should not be a worry in itself as we are actually reducing security permissions by changing from 777 to 755 permissions
- The second case of Synology updates is something to be aware of and that you might need to set this permission again in future after an update if that update resets the permissions to 777
  - Based on the fact that there is a risk of permissions being reset outside of our control I would discourage the removal of username/pw authentication possibility in sshd_config (/etc/ssh/sshd_config) which some has suggested to do when correctly having SSH key authentication working.
 
Now if you want to be 100% sure you have the correct permissions for the user home and the .ssh directory and authorized_keys you can either
- Run these below chmod commands to set the correct permissions:
```
sudo chmod 755 /volume1/homes/{admin-user}
sudo chmod 755 /volume1/homes/{admin-user}/.ssh
sudo chmod 644 /volume1/homes/{admin-user}/.ssh/authorized_keys
```

