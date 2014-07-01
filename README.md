We have 3 config servers ( say config1, config2 & config3 ) in production and imagine we have config db differ issue (configdb mismatch) on the config1 & config3.
Then we may have to follow the following procedure to recover it, please share your views and suggestion for any better procedure.

1)      Take Config2 Mongodump

dt_today=`date +%Y%m%d%H`
cd /opt/mongodb/bin/
./mongodump -h x.x.xx.xx --port 37010 -v -d config -o /opt/mongodb/data/configBackup/"$dt_today"_"Config1_new" >> /opt/mongodb/data/configBackup/logs/Backup_$dt_today.log

2)      SCP the dump backup to other 2 configs ( to “/opt/mongodb/” folder).

3)      Stop ConfigDB on Config1

4)      Start the configdb on different port, say 22222.

numactl --interleave=all ./mongod --configsvr --port 22222 --journal --quiet --fork --dbpath /opt/mongodb/data/conf --logpath /opt/mongodb/logs/confsvr_22222.log

5)      Restore the Config DB data

./mongorestore -h localhost --port 22222 -v -d config /opt/mongodb/20140624_Config1_new/config/

6)      Stop configdb running on port 22222

7)      Start Config db with original port

numactl --interleave=all ./mongod --configsvr --port 37010 --journal --quiet --fork --dbpath /opt/mongodb/data/conf --logpath /mongodb/logs/confsvr.log

8)      Repeat the Steps #3 to #7 for Config#3
