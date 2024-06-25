Let’s designate our source and destination clusters as follows:

Source:

```
mongo-test-centos-01
mongo-test-centos-02
mongo-test-centos-03
```

Destination:

```
mongo-test-ubuntu-01
mongo-test-ubuntu-02
mongo-test-ubuntu-03
```

1. MongoDB Cluster-to-Cluster Sync Download

You can download mongosync from the link below; it is sufficient to download it to the source server.

Download MongoDB mongosync
MongoDB mongosync enables Cluster-to-Cluster Sync, allowing you to continuously sync data across any MongoDB clusters…
www.mongodb.com


```
wget https://fastdl.mongodb.org/tools/mongosync/mongosync-rhel70-x86_64-1.7.1.tgz
After the download process, you can extract the compressed file.
tar -zxvf mongosync-*.tgz
To add the mongosync binary file to your Path, you can use one of the following commands.
sudo cp /root/mongosync-rhel70-x86_64-1.7.1/bin/mongosync /usr/local/bin/
sudo ln -s /root/mongosync-rhel70-x86_64-1.7.1/bin/mongosync /usr/local/bin/mongosync
```

2. Creating a New User

Root role users must be created for both clusters. These users will facilitate communication and data transfer between the source and destination clusters.

Source:

```
db.createUser({user: "clusteradmin", pwd: "password1", roles: [{role: "root", db: "admin"}]});
```

Destination:

```
db.createUser({user: "clusteradmin", pwd: "password2", roles: [{role: "root", db: "admin"}]});
```

3. Creating the Mongosync Configuration File

We can create the configuration file under etc as mongosync.conf. While cluster0 points to the source cluster, cluster1 points to the destination cluster.

```
vi /etc/mongosync.conf
cluster0: "mongodb://clusteradmin:password1@mongo-test-centos-01:27127,mongo-test-centos-02:27127,mongo-test-centos-03:27127"
cluster1: "mongodb://clusteradmin:password2@mongo-ubuntu-test-01:27127,mongo-ubuntu-test-02:27127,mongo-ubuntu-test-03:27127"
logPath: "/var/log/mongosync"
verbosity: "WARN"
```

To check if the configuration is successful, you can run the following command.

```
cd /usr/local/bin/ 
nohup mongosync - config /etc/mongosync.conf >> mongosync1.log &
```

4. Initiating the Synchronization Process

The following command starts the synchronization, and data flow from the source cluster to the destination cluster begins. Unless manually intervened, synchronization will continue continuously. If needed, data flow can be paused and canceled.

```
curl localhost:27182/api/v1/start -XPOST \
--data '
   {
      "source": "cluster0",
      "destination": "cluster1",
      "reversible": true,
      "enableUserWriteBlocking": true
   } '

```

Note: The “enableUserWriteBlocking” parameter being set to true indicates that a user other than the clusteradmin cannot input data into the destination cluster.

Different situations, such as progress, pause, resume, and commit, are available for use when needed.

```
curl localhost:27182/api/v1/progress -XGET
curl localhost:27182/api/v1/pause -XPOST --data '{ }'
curl localhost:27182/api/v1/resume -XPOST --data '{ }'
curl localhost:27182/api/v1/commit -XPOST --data '{ }' # to disconnect
```