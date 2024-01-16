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

## start and stop mongod

> sudo service mongod start
> sudo service mongod stop

> use admin;
> db.shutdownServer()


> kill  pidOfMongod
> do not use kill  -9 

if a unclean shutdown happened, start the server on a different port and the same data path, the server will do the data recovery work,, after the server on another port is started, recovery data will take some time depend on the data you have.after the data recovery, stop the server ,and restart the server use the original port


> ./bin/mongod   --dbpath ./data/ --port 40000
>

why start a new port not the original one?

+  to do the data recovery
+  prevent client to connect before the date recovered

## start/stop mongo on windows

> net start mongodb
> net stop mongodb

> taskkill /IM mongod.exe
> taskkill /F /IM mongod.exe

# configuration file

## Journaling

### journal Flush Interval

db.serverStatus().dur

> dir /s /b data1

> netsh interface ip show address

### webpage 

## Upgrading

Version number

3.0.1

Major.Minor(Odd}Even).Revision

> Even for each stable release

> Never go Odd in Prod

# Query crash course

> db.collectionName.stats()

## ObjectId(12 Bytes)

ObjectId('65830aa5629da9bcfed8b116')

 4 bytes of Unix Eponch 
 3 bytes of  MachineId 
 2 bytes of  Processid 
 3 bytes of  Counter


ObjectId('65830aa5629da9bcfed8b116').getTimestamp()

Ascending Id 

## Cursor
> x = db.collectionName.find()
>x.next()

# Data in ,Data Out

> mongodump

> mongodump --host hostname  --port port --out outdir

> mongodump --oplog only apply when running a replica set

> mongodump --db dbname

> mongodump --db dbname --collection collectionname

> admin and local db 

> mongodump --username username --password password --db dbname --collection collectionname

use bsondump to read mongodump files

## mongorestore


> mongorestore backup\dump

> mongorestore --host hostname --port 27017 backup\dump

> mongorestore --host hostname --port 27017 backup\dump --drop
> mongorestore --host hostname --port 27017 backup\dump --db dbname --collection collectionName   backup/dump/dbname/collectionname
> mongorestore --host hostname --port 27017 backup\dump --oplogReplay

## mongoimport & mongoexport

> mongoimport --type [json|csv|tsv]

> mongoimport --db dbname --collection collectionName file.json
> mongoimport --db dbname --collection collectionName file.json --upsert
> mongoimport --db dbname --collection collectionName  --upsert  --upsertFields fieldName file.json

> mongoimport --type csv --headerline --db dbname --collection collectionName  --upsert  --upsertFields fieldName file.csv

> mongoimport --type csv --fields field1,field2...fieldn --db dbname --collection collectionName  file.csv
> mongoimport --type csv --fieldFile  filedfile(one column per-line) --db dbname --collection collectionName  file.csv

> mongoexport  --db dbname --collection collectionName 

> mongoexport  --db dbname --collection collectionName  > out.json
> mongoexport  --db dbname --collection collectionName  --out out.json

> mongoexport  --db dbname --collection collectionName  --out out.json --fields field1,field2 (with id )
> mongoexport  --db dbname --collection collectionName  --out out.csv --fields field1,field2 --type=csv (without id)
> mongoexport  --db dbname --collection collectionName  --out out.csv --fields field1,field2.field3 --type=csv
> mongoexport  --db dbname --collection collectionName  --out out.csv --fields field1,field2.field3 --type=csv --query '{_id:{$gt:2}}'

> mongoexport  --skip 0 --limit 2 --sort '{_id:1}' --db dbname --collection collectionName 
> mongoexport  --skip 2 --limit 2 --sort '{_id:1}' --db dbname --collection collectionName 
>
>

# Indexes

> BTree
> db.system.indexes.find()

## create index

> db.collectionName.ensureIndex({field: direction})
> db.collectionName.dropIndex(indexName)

> db.collectionName.ensureIndex({'fields.subfield1': direction})

> db.collectionName.find().explan()

> db.collectionName.find().explan('executionStats')
> db.collectionName.find().explan(true)

> yielding number

###  covered index
> totalDocsExamined : 0 in the query explain


## sparse index

> db.collectionName.ensureIndex({'fields.subfield1': direction}, {sparse: true})
> db.system.indexes.find({name: 'prom_1'})  # sparse : true


sort fields should not be a sparse index


force to use index

> db.collectionName.find({}).sort({fields:1}).hint({field:1})

## unique index

> db.collectionName.ensureIndex({'fields': direction}, {unique: true})

## Sparse + Unique

> db.collectionName.ensureIndex({'fields': direction}, {unique: true, sparse: true})

unique index still works for array
unique index doesn't work on shards


## TTL index

> db.collectionName.ensureIndex({'fields': direction}, expireAfterSeconds: 60 * 60 * 24 * 90})

+ Removes documents after they expire
+ Define on date-time field
+ One TTL index per collection


### Build index on the background

> db.collectionName.ensureIndex({'fields': direction}, expireAfterSeconds: 60 * 60 * 24 * 90, background: true})

## reindex

> db.collectionName.reIndex()

## compact

Rebuilds collection

+ Defragments documents
+ Rebuilds all indexes
+ block operation

> db.runCommand({compace: 'mycollection'})


index key selectivity


index limitation


KeyValue Size Limit

+ 1024 bytes
+ combined value data size of all fields in index

KeyName Size limit

+ 128 bytes

max Keys per index

+ 31



Indexes Per Collection

+ 64
+(am i over-indexing)



#  ReplicaSet

## Capped Collection 

> start mongod --dbpath dirname --replSet r1 --oplogSize 1
> mongo 

> rs.initiate() # start a replicaSet
> rs.config()
> var conf = rs.config()
> rs.reconfig(conf)


> rs.status()

## Elections

## add more members to the replicaset

> start mongod --port 30001 --dbpath ./fresh/r1 --replSet r1
> start mongod --port 30002 --dbpath ./fresh/r2 --replSet r1
> start mongod --port 30003 --dbpath ./fresh/r3 --replSet r1

> rs.add('myserver:30002')
> rs.add('myserver:30003', true) # arbiter
> for(i=0;i < 10;i++) { db.demo.save({_id:i}) }

> db = connect('myserver:30002/test') # connect to other server
> db.setSlaveOk()
> db.demo.find()

## set replica member priority

> var cfg = rs.config()
> cfg.members[0].priority = 10
> rs.reconfig(cfg) 
> rs.reconfig(cfg, {force: true}) 

> db.getMongo()


### stepDown


> rs.stepDown(3 * 60)

## freeze

> rs.freeze(5 * 60)

## hidden

> cfg.members[0].priority = 0 # 0 means will never become a primary
> cfg.members[0].hidden = true


## Chaining

> rs.settings.chainingAllowed


Allow chaining lets you save on network traffic and potentially latency
disallow chaining forces all secondaries to directly pull from the primary

## Durability

Flush to Disk

Tow is One
and One is None


## Wriote concern

+ "Acknowlgeged" write concern
+ "unacknowlgeged" write concern (fastest)
+ "journaled" write concern(default)
+ majority write concern("majority of voting members")


> db.demo.insert({'x':'hh'}.{writeConcern: {w:0}})
> db.demo.insert({'x':'hh'}.{writeConcern: {j:True}}) # journaled true
> db.demo.insert({'x':'hh'}.{writeConcern: {w:1}})
> db.demo.insert({'x':'hh'}.{writeConcern: {w:2}})
> db.demo.insert({'x':'hh'}.{writeConcern: {w:2, j:true}})
> db.demo.insert({'x':'hh'}.{writeConcern: {w:'majority'}})


# Sharding


> mongod --configsvr --dbpath=fresh/config 

> mongos  --configdb myserver

> mongod --port 3001 --dbpath r1
> mongod --port 30012--dbpath r2
> 
> sh.addShard('myserver:3001')
> sh.addShard('myserver:3002')
> sh.status()
> sh.enableSharding('dbname')
> sh.shardCollection('dbname.collectionName',{'shardingKey1':1, 'shardingKey2':2}) # index will be created








































