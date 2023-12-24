iMongoDB
 
 + Written in C++
 + Self contained
 + No external dependencies
 
 > mms (commerical distribution management platform)

> wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel80-3.6.23.tgz
> tar zxvf mongodb-linux-x86_64-rhel80-3.6.23.tgz
> mkdir data
> cd mongodb-linux-x86_64-rhel80-3.6.23
> ./bin/mongod --dbpath ./data

File system requirement

+ ext4
+ xfs
+ ext3 (not recommend)

> mount

check system file system

ensure that the operating system doesn't update last access time on files

vi /etc/fstab
> add ,noatime after the defaults

## install on ubuntu

+ install a key
+ add package repo
+ sudo apt-get update
+ sudo apt-get install -y mongodb-org( all the tools )
+ sudo apt-get install -y mongodb-org-server( just the server )

put a hold
> echo 'mongodb.org hold' | sudo dpkg --set-selections

startup configure file path

> /etc/initd/mongod.conf

> numa


## install on windows


config file 

mongod.conf


'''yaml
storeage:
    dbPath: "d:/mongoadmin/data"
systemLog:
    destination: file
    path: "d:/mongoadmin/log/mongod.log"
'''

change dir permissions

> icacls d:\mongoadmin /grant mongodb:(OI)(CI)F


install as a windows service

> mongod --install -- config d:\mongoadmin\config\mongod.conf --serviceuser mongodb --servicepassword somepassword

> sc qc mongodb

> net start mongodb

uninstall mongod service

> net stop mongodb
> mongod --remove







