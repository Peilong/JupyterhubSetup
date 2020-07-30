# Install Jupyterhub as a Ubuntu Service

_Rev. 2020/08/01_

## Step 1: Install Anaconda
+ Download Anaconda from website: [link](https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh)
``` sh
wget https://repo.anaconda.com/archive/Anaconda3-2020.07-Linux-x86_64.sh
```
+ Install Anaconda3
``` sh
bash Anaconda3-2020.07-Linux-x86_64.sh
```
+ Assuming `/home/lip` is my home directory, add condo to system path. This step is not necessary if you've accepted to add conda to system path during installation.
``` sh
vim ~/.bashrc
export PATH="/home/lip/acaconda3/bin:$PATH"
```

## Step 2: Install Jupyterhub

``` sh
conda install jupyterhub
```

## Step 3: Generate Jupyterhub configuration file

``` sh
mkdir /home/lip/jupyterhub
# Use this directory as the place for config, database, cookies, etc.
cd /home/lip/jupyterhub
jupyterhub --generate-config
```
Will see a `jupyterhub_config.py` file showing up.

## Step 4: Jupyterhub configuration

+ Open jupyterhub_config.py file and edit the following:

``` sh
## Use absolute path for the database, this is critical for setting up service later on
c.JupyterHub.db_url = 'sqlite:////home/lip/jupyterhub/jupyterhub.sqlite'
## Hub listens on 54321 to avoid conflict with HDP 2.6.5
c.JupyterHub.hub_port = 54321
## Out-facing IP, should be static
c.JupyterHub.ip = 'xx.xx.xx.xx'
## The port used by web browser client
c.JupyterHub.port = 12345
## Enable admin account
c.Authenticator.admin_users = {'lip', 'root'}
```

## Step 5: Set Jupyterhub as a service [Ref](https://github.com/jupyterhub/jupyterhub/wiki/Run-jupyterhub-as-a-system-service#ubuntudebian-anaconda3-with-systemd)

+ Create a service file for system daemon
``` sh
sudo vim /etc/systemd/system/jupyterhub.service
```
+ Add the following to the service file
``` sh
[Unit]
Description=Jupyterhub
After=syslog.target network.target

[Service]
User=root
Environment="PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/home/lip/anaconda3/bin"
ExecStart=/home/lip/anaconda3/bin/jupyterhub -f /home/lip/jupyterhub/jupyterhub_config.py

[Install]
WantedBy=multi-user.target
```
+ Add in system Service
``` sh
sudo systemctl daemon-reload
sudo systemctl <start|stop|status> jupyterhub
# make jupyterhub automatically start when boot
sudo systemctl enable jupyterhub
```
