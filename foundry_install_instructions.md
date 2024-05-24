# Install Foundryvtt on Ubuntu Server
## Prerequisites:
- Raspberry Pi running ubuntu server
- SSH Access to the server.

### Installation instructions:
SSH into your linux box.  You will need the ip of your machine and the root password.
```
ssh root@<ubuntu_ip>
```
### Enable Firewall and Open Ports for foundryvtt
```
sudo ufw enable
sudo ufw allow 30000
```

### Install Node
```
nvm -v
curl -o- https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.bashrc
nvm install 18
nvm use 18
node --version
```
### Create application and user data directories
```
cd $HOME
mkdir foundryvtt
mkdir foundrydata
```

## Install the software
You need your foundry download url.  Go to https://foundryvtt.com and login.  Navigate to Purchased Licenses.  Click on timed url (valid for 300 seconds only). Insert into the wget command below:
```
cd foundryvtt
wget -O foundryvtt.zip "<foundry-website-download-url>"
unzip foundryvtt.zip
```
### Start running the server to test it:
```
node resources/app/main.js --dataPath=$HOME/foundrydata
```
If all was successfull, your server should be up and running.  You can access by going to http://<pi_ip>:30000   

The only downside is it will shut down when you stop the process unless you set it up as a service.

### Service Script.
We can create a service to launch foundry on system startup, so you don't have to worry about it.
```
sudo nano startup.sh
```
Copy the following lines into the script.  Change your world and /home/<user> path as needed....
```
node /home/ubuntu/foundryvtt/resources/app/main.js --port=30000 --world=aeshyar --dataPath=/home/ubuntu/foundrydata
```
Press CTRL+O, enter to save.  CTRL + X to exit.  We then need to make it executable.  
```
chmod +x startup.sh
```
If you enter the command ls it should now show green and executable.  This is just a test script to fire it off and test it starts up ok before created the service.  Enter the following command to make sure it's working.
```
sudo ./startup.sh
```
You should be able to get to foundry.

<span style="color:red">Note this is all from memory about configuring the service. I think it works, but it might take a little troubleshooting.  It's been a while.</span>.

Now we need to set it up as a service.  Navigate to the services folder:
```
cd /lib/systemd/system/
sudo touch foundry.service
sudo nano foundry.service
```
Paste the following into the foundry.service file.  CTRL+O, Enter to save. CTRL+X to exit.

```
[Unit]
Description=FoundryVTT
DefaultDependenciees=no
After=network.target

[Service]
Type=forking
RemainAfterExit=yes
User=ubuntu
ExecStart=node /home/ubuntu/foundryvtt/resources/app/main.js --port=30000 --world=<worldname> --dataPath=/home/ubuntu/foundrydata
Restart=on-failure
RestartSec=10s
Type=simple

[Install]
WantedBy=multi-user.target
```

Make the service executable and enabled at boot. 
```
chmod u+x /lib/systemd/system/foundry.service
sudo systemctl start foundry.service
sudo systemctl enable foundry.service   
```

In theory the service is working and you are done.  You can stop, start and check the status at any time with the following commands.

```
sudo systemctl stop foundry.service  # to stop the service manually

sudo systemctl start foundry.service # to start it manually

sudo systemctl status foundry.service  # to get status and logs
```
