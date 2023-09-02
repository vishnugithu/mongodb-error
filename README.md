# mongodb-error
Replication is not created when intiate 
#!/bin/bash
cd /home/ubuntu

#Intallation steps for Mongodb
echo "Install Mongodb 5.o"
sudo apt-get install gnupg curl
curl -fsSL https://pgp.mongodb.com/server-5.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-5.0.gpg \
   --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-5.0.gpg ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/5.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-5.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org=5.0.19 mongodb-org-database=5.0.19 mongodb-org-server=5.0.19 mongodb-org-shell=5.0.19 mongodb-org-mongos=5.0.19 mongodb-org-tools=5.0.19
<< comment
when i tried to install mongodb the error below error is getting 
 The following packages have unmet dependencies:
 mongodb-org-mongos : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-server : Depends: libssl1.1 (>= 1.1.1) but it is not installable
 mongodb-org-shell : Depends: libssl1.1 (>= 1.1.1) but it is not installable
E: Unable to correct problems, you have held broken packages.

To solve this error below commands enter below commands

  echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
   sudo apt-get update
   sudo apt-get install libssl1.1 

   Then delete the focal-security list file you just created:

   sudo rm /etc/apt/sources.list.d/focal-security.list 
comment
# Create a 3 different directories for MongoDB data servers
mkdir -p server27017 server27018 server27019

# Start MongoDB instances for three different servers along with javascript disable and wiredTigercahe size 1gb
sudo systemctl enable mongod
sudo systemctl start mongod
cd /home/ubuntu/server27017
mongod --port 27017 --dbpath /home/ubuntu/server27017 --logpath /home/ubuntu/server27017/mongodb1.log --replSet rs0 --wiredTigerCacheSizeGB 1 --setParameter disableJavaScriptProtection=1 --fork
cd home/ubuntu/server27018
mongod --port 27018 --dbpath /home/ubuntu/server27018 --logpath /home/ubuntu/server27018/mongodb2.log --replSet rs0 --wiredTigerCacheSizeGB 1 --setParameter disableJavaScriptProtection=1 --fork
#Arbitary member
cd home/ubuntu/server27019
mongod --port 27019 --dbpath /home/ubuntu/server27019 --logpath /home/ubuntu/server27018/mongodb3.log --replSet rs0 --wiredTigerCacheSizeGB 1 --arbiterOnly  --setParameter disableJavaScriptProtection=1 --fork

#Initializing the replica set in local

mongo --port 27017 <<EOF
cfg={"_id":"rs0",
... "members":[{"_id":0,"host":"localhost:27017"},
... {"_id":1,"host":"localhost:27018"},
... {"_id":2,"host":"localhost:27019",arbiterOnly: true},
... ]
... }
use local
rs.initiate(cfg)
EOF

# Now check the replication status initialized or not
mongo --port 27017 --eval "rs.status()"

echo "MongoDB replica set 'rs0' is now set up with 3 members (1 primary, 1 secondary, and 1 arbiter)."


#this script in not initialised and getting below error
{
        "ok" : 0,
        "errmsg" : "not running with --replSet",
        "code" : 76,
        "codeName" : "NoReplicationEnabled"
}
