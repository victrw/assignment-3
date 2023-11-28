# Detailed tutorial for a new digitalocean user:

### Create a new regular user

User can perform administrative tasks

User has bash as login shell

User can access the server via SSH


Prevent the root user from connecting to the server via SSH

### Nginx

Install nginx package

Configure nginx to serve a sample website



##  Step 1 - Finding your droplet's IP address

To do this, go digitalocean and locate the location of your droplet. On the right side of your droplet name, there should be an IP address. To copy, you can hover over it and left click the address. 


## Step 2 - Connecting to your debian server

Since the droplet was recently created, the root user should be the only user in your server so you will have to use **root**.

Before we connect, we have to find the absolute path to where your **private** ssh key is. 
Path e.g. (/Users/john-doe/.ssh/[ssh-keyname])

Once that is found, open up your prefered command line terminal and type the following:

```
ssh -i [path to ssh key] root@[your IP address]
```
Since it is going to be the first time you are connecting, you might see this:

Are you sure you want to continue connecting (yes/no/[fingerprint])? Make sure you type "**yes**".

You should now be logged in as root.


## Step 3 - Creating a new regular user

To add a user, we can type "**useradd**". 

In our case, we want the user to be able to have bash so we can type "**-s**" to create a shell and "**-m**" to create the home directory and required files. To do this we can type:
```
useradd -ms /bin/bash [username]
```
Replace "[username]" with whatever username you would like for the new user.

After creating a new user, it is still not secured. We need to add a password to it. To add a password to a user, we can type:
```
passwd [username]
```

If everything is good, you should have successfully created a secured user. To check you can type:
```
cat /etc/shadow
```
It should show you your new user and a encrypted version of your password.


## Step 4 - Modifying permissions of your user

Right now, your user should not have any admistrative power. To change this, we will need to add the user to the sudo group. To do this, we need to type:
```
sudo usermod -aG sudo [username]
```
**Note - The "a" is very important as without it, the user will be removed from all other groups.**

This will allow your newly created user to run commands using sudo without having to enter their password everytime.


## Step 5 - Connecting as a user via SSH

What we want to do first is to copy "**cp**" the .ssh file in root and move it to the new user's home directory. 

Since the .ssh has content inside, we need to type "**-r**". To do this, type:
```
sudo cp -r /root/.ssh /home/[username]
```
This command copies over the necessary files that allow ssh access.

To check if the files have been successfully copied over, we can type the command:
```
cd /home/[username]
```
```
ls -al
```
You should see something similar to this:

**drwx------ 2 root root 4096 Nov 28 19:03 .ssh**

We will now be utilizing "**chown**" to change the ownership of the .ssh file. **Make sure you are on root** and type the following:
```
sudo chown -R [username]:[username] /home/[username]/.ssh
```
This makes it so that .ssh and everything inside of it will be transfered over to user. To check again, type:
```
ls -al
```

To check that everything is working correctly, exit out of the root user by typing: 
```
exit
```
and try connecting back to the server with user instead of root.
```
ssh -i [path to ssh key] [username]@[your IP address]
```

If there are no issues, you would be able to successfully connect to your server with your username instead of root.


## Step 6 - Removing root from connecting via SSH

We need to remove access from the root as this account is the most valuable target for hackers. To do this we need to go to the /etc/ssh directory. 
```
cd /etc/ssh
```
We will now be editing the configuration of the file called "**sshd_config**". 

The **sshd_config** file specifies the locations of one or more host key files and the location of authorized_keys files for users.

Since we are changing a file in root, we need to use "**sudo**" and a file editor named "**vim**" 
```
sudo vim sshd_config
```
Inside of **vim**, find the line that says **/PermitRootLogin**. 

A quick way to do this would be to make sure vim is in visual mode by pressing **esc** and type:
```
/PermitRootLogin
```
Once you see **PermitRootLogin**, press enter. Currently, you can see that the value is set to yes.

To change this, you will need to press "**i**" to enter insert mode in vim and change the value "yes" to "**no**" 

Once you have entered no, click "**esc**" to get out of insert mode and type "**:wq**" to save and exit out of vim. 

If no errors popped up while trying to save and quit, you have successfully edited the sshd_config file. 

You will then need to restart the ssh.service. Type the following:
```
sudo systemctl restart ssh.service
```
This makes it so that the ssh.service restarts, and applies the changes you have made to the sshd_config file.

To check if you have successfully removed access from root, you can first exit out of the server and try to connect as root. If everything was done properly, you should see:

 **root@137.184.9.248: Permission denied (publickey)**.

 ## Step 7 - Installing nginx

Before we start installing Nginx, we need to make sure our system is up to date first. To do this, type:
```
sudo apt update
```
This will make it so that the package index files on the system are updated, which will then contain information about available upgradable packages and their versions.

Next, we will need to type:
```
sudo apt upgrade
```
What this is doing is actually **installing** and **applying** the new version available.

To install Nginx, first run the command:
```
sudo apt install nginx
```
This will install the nginx packages on your system.

To check if nginx service is running, type the command:
```
sudo systemctl status nginx
```
You should see that it is enabled and running. After that you can press "**ctrl + c**" to get back to the command line.

If for some reason the nginx.service is not running, you can type the following:
```
sudo systemctl start nginx
```
```
sudo systemctl enable nginx
```
```
sudo systemctl restart nginx
```
Systemctl **start** will start the nginx service and **enable** means that the service will start at boot. The **restart** will make sure that the latsest version of the service is running. To make sure everything is running and working, check the service again. 

If everything is enabled and running, you have successfuly installed nginx package to your system. 


## Step 8 - Creating HTML document to serve

First lets go to the Debian server's default location which is where dcumets served by nginx is.
```
cd /var/www
```
Once we are in the /www directory, we need to make another directory for your new site. You can change the name to whatever you want the name to be but for our case, we will be naming it my-site.
```
sudo mkdir [my-site]
```
This will be the directory where you will be storing your html file. Next would be to change directories to be inside of it, and creating a new index.html file.
```
cd [my-site]
```
```
sudo vim index.html
```
Inside of the editor paste in the following code:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2420</title>
    <style>
        body {
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }
        h1 {
            text-align: center;
        }
    </style>
</head>
<body>
    <h1>Hello, World</h1>
</body>
</html>

```
This code is just basic html document that will print "Hello world" in the middle of your screen. Once you have this pasted into vim, make sure to exit out and save the file with **:wq**.


## Step 9 - Configuring a server block

First, change directories to /etc/nginx with the following command:
```
cd /etc/nginx
```
In this directory, we will be needing the following 2 directories called **sites-available** and **sites-enabled**. The sites-available directory holds all your site configurations and the sites-enabled directory allows you create symbolic links to the pervious directories for the sites you wish to enable.

Now that we know what these directories do, we need to change directories to sites-available. To do this type the following:
```
cd sites-available
```
Now that we are in here, we need to create a new file. Feel free to name the file whatever you wish.
```
sudo vim my-site.conf
```
The most important part that we need to copy in is:
```
server {
	listen 80;
	listen [::]:80;
	
	root /var/www/[my-site];
	
	index index.html index.htm index.nginx-debian.html;
	
	server_name _;
	
	location / {
		# First attempt to serve request as file, then
		# as directory, then fall back to displaying a 404.
		try_files $uri $uri/ =404;
	}
}

```
The provided Nginx configuration file snippet is defining the setup for a basic web server.


To server our HTML document that we just created instead of the default document, we will need to edit the root/var/www/**[my-site]** which I have done above. You would have to change "**[my-site]**" to the name of the directory you have created in step 8. After that is done, remember to exit out of insert mode and type **:wq** to save the file and exit out of vim.

Now we will have to change directories to sites-enabled. To do this type the following:
```
cd ../sites-enabled
```

Now that we are in here, remember that sites-enabled is for creating symbolic links to the previous directories for the site you want to enable. To create a symbolic link, type the following:
```
sudo ln -s /etc/nginx/sites-available/my-site.conf /etc/nginx/sites-enabled/my-site.conf@
```
Adding @ at the end of the my-site.conf is just a way to distinguish the linked file from the original configuration file. This will then create the link in the sites-enabled folder.

To test that your code has no syntax errors, you can type the following:
```
sudo nginx -t
```
-t is just to test the configuration file and if there are any errors, output the error. 

If no errors popup, you have successfully configured a server block for your custom webpage. Make sure to restart nginx with the following command:
```
sudo systemctl restart nginx
```
We do this because we want nginx to restart and run the latest version which includes the changes that you made.


## Step 10 Testing your webpage 

Go to your browser and type in the IP address of your debian server and you should see "Hello world" in the middle of your screen. 

**Congratulations**, you followed along and finished! However, if you are still seeing the default nginx page, make sure you restarted the nginx service with the command listed above.