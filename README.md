# Multi_Machine

## Setting up 2 machines
First, create a vagrant file and set as it contents:
```
Vagrant.configure("2") do |config|

  
  config.vm.synced_folder "app", "/home/vagrant/app"
  
   config.vm.define "main" do |main|
	 main.vm.box = "ubuntu/xenial64"
	 main.vm.provision "shell", path: "app/mainprovision.sh"
	 main.vm.network "private_network", ip: "192.168.10.100"
  end
  
   config.vm.define "db" do |db|
    db.vm.box = "ubuntu/xenial64"
	db.vm.provision "shell", path: "app/dbprovision.sh"
	db.vm.network "private_network", ip: "192.168.10.150"
  end
  
  end
  ```
  By doing this, we setup two virtual machines, one called `app`, another called `db` and assigne them their own IPs
  
  ## Provision Files
  In order to get the required files installed, you'll need to set up two seperate provision files to download what you need.
  in the app directory create a provision file called `mainprovision.sh` with the following contents:
  ```
#!/bin/bash
sudo apt-get update -y

sudo apt-get upgrade -y

sudo  apt-get install nginx -y

sudo systemctl enable nginx

sudo apt-get install nodejs -y

sudo apt-get install npm-y

sudo apt-get install python-software-properties

curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash

sudo apt-get install nodejs -y

sudo npm install pm2 -g

npm install
```
When you do 'vagrant up app' to launch the app VM this will be run automatically

Create another provision file called: `dbprovision.sh` with the following contents:
```
# be careful of these keys, they will go out of date
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

sudo apt-get update -y
sudo apt-get upgrade -y

# sudo apt-get install mongodb-org=3.2.20 -y
sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

# remoe the default .conf and replace with our configuration
sudo rm /etc/mongod.conf
```
When you do 'vagrant up db' to launch the db VM this will be run automatically.

Next, ssh into the db and use `cd /etc`, then use `sudo nano mongod.conf` and fill with the following contents:
```
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0


#processManagement:

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:
```
After that run the commands to bring up the db:
```
sudo systemctl restart mongod
sudo systemctl enable mongod
```

- ssh into the app and use `cd app`
- Then create an environement variable using: `DB_HOST=mongodb://192.168.10.150:27017/posts`
- After that, you need to seed the database. Use the command `cd seeds` and then run `node seed.js`
- cd back into the app directory, then run `npm start` to restart the app.js
- Finally, on a brower enter the URL: `/192.168.10.100:27017/posts` to load up the posts page

## Additional info
To make the environment variable persistant:
- SSH into the app
- Use the command: `sudo nano etc/environement`
- Paste the DB_HOST environment variable into the contents of that file
- The variable should now be persistant
