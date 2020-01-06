# How to Deploy

## Cloud Server
We used [Digital Ocean](http://digitalocean.com/), there's also Microsoft's [Azure](https://azure.microsoft.com/en-us/) and Amazon's [AWS](https://aws.amazon.com/).

Digital Ocean refers to a server as a "droplet". When creating a droplet you can specify which operating system (I recommend Ubuntu or Debian), a plan (I recommend starting with their $5/month plan) and a region (choose whichever is closest).

When a server is created, note the server's IP address and (if you created a server with password authentication instead of with SSH keys) the root user's password. For Digital Ocean, the root user's password will be sent in an email.


## Connecting to the Server
In order to communicate with the remote server you need to connect to it. We use the `ssh` command for that.

The syntax is `ssh [username]@[ip address]`. So if my server is at `64.225.000.000` I would sign in as the root user via `ssh root@64.225.000.000`. 

You can log out of the server with `logout`.


## Updating the Server
First thing to do with a new server is update all the software. Run `apt update` and then `apt upgrade`. If you want to restart the machine after you can type `reboot`. Note that these commands usually require `sudo` if not signed in as the root user.


## Create a Non-Root User
The root user is dangerous and should only be used in special circumstances. For day-to-day operations we'll make a non-root user with the ability to `sudo` for special commands instead.

To create a user use the `adduser` command. To create a user `tyler` run the command `adduser tyler`. You'll set the user's password. Questions about the user's full name or phone number can be left blank. 

In order for the user to use `sudo` for root-level commands run `usermod -aG sudo tyler`.


## Disable Root Login
It's rarely a good idea to let someone sign in to the machine directly as root so we will disable that. Sign in to the server with the newly-created non-root user. Edit `/etc/ssh/sshd_config` with `sudo` and either `vim` or `nano` (for example, `sudo vim /etc/ssh/sshd_config`) to change the `PermitRootLogin` value to `no`.

In order for the changes to take effect we'll restart the SSH process with `sudo service sshd restart`. You should no longer be able to sign into the server with the `root` user.


## Firewall
Currently, all ports on the server are open; we can use a firewall to close all but the ones we need. We'll use a program called `ufw` to achieve that.

To install: `sudo apt install ufw`. 

We need to be very careful that we don't immediately enable the firewall without opening any ports, otherwise we will lose the ability to ssh into the server! To enable SSH open port 22 by running `sudo ufw allow 22`. 

Before enabling we need to run `sudo ufw status` to make sure it is runnign correctly. 

To enable the firewall run `sudo ufw enable`. we may need to hit `y` to  continue. So now only port 22 is open. 


## SSH Key
Rather than logging in with a username & password, it's best to use SSH keys. With Digital Ocean you can create a droplet that is automatically associated with SSH keys. Otherwise, we can set them up manually.

### Create an SSH Key
Use the `ssh-keygen` command to make an SSH key. You'll specify the name of the key and if you want it password-protected. It will create two files in your current working directory, one file is the *public* key that ends in `.pub`, and the other is a *private key* that will have no file extension. 

You should keep your SSH keys in your local `~/.ssh/` directory, so `cd` into that directory and run the `ssh-keygen` command. 

### Share the Public Key
Next you need to add the public key to the server's `~/.ssh/authorized_keys` file. Given a public key called `brock` we would run `cat ~/.ssh/brock.pub | ssh tyler@64.225.000.000 'cat >> ~/.ssh/authorized_keys'`. Note that the `~/.ssh` directory must already exist; you'll probably have to sign into the server and create one before running this command. 

### Use the Private Key to Log In
Use the `-i` flag with a path to the private key to log in with it. For example, `ssh -i ~/.ssh/brock tyler@64.255.000.000`. 


## Domain Name
I like to use [Namecheap](namecheap.com/). Buy a domain with some sort of Whois protection, otherwise your personal information will be publicly available on the internet! 

You want the domain to "point" to the IP address of your server. You'll do that by adding a *host record*, specifically an *A record*. An A record includes a *Host* for specifying the subdomain, such as `www` or `@` for a catch-all, the *Value* will be the IP address of the server. *TTL* is how long until the changes take place; you can leave this at its default value.

Once the domain is set up it will take a few minutes for the internet to catch up. You can `ping` the domain to see if it now responds with the new IP address and use SSH with the domain name instead of the IP address. For example, with an `underwater.pizza` domain I can SSH with `ssh tyler@underwater.pizza`. 
