## Hadoop

>[Apache Hadoop](https://hadoop.apache.org/) is a collection of open-source software utilities that facilitates using a network of many computers to solve problems involving massive amounts of data and computation. It provides a software framework for distributed storage and processing of big data using the MapReduce programming model.

> The core of Apache Hadoop consists of a storage part, known as Hadoop Distributed File System (HDFS), and a processing part which is a MapReduce programming model. Hadoop splits files into large blocks and distributes them across nodes in a cluster. It then transfers packaged code into nodes to process the data in parallel. This approach takes advantage of data locality, where nodes manipulate the data they have access to. This allows the dataset to be processed faster and more efficiently than it would be in a more conventional supercomputer architecture that relies on a parallel file system where computation and data are distributed via high-speed networking.

Source: [Wikipedia](https://en.wikipedia.org/wiki/Apache_Hadoop)

### Setup

One of the outcomes of the [Big Data Europe](http://www.big-data-europe.eu) project was a repository of Docker images of each of the building block big data tools, one of those being Apache Hadoop. These continue to be maintained.

Hadoop depends on Java 8.

Rather than install this old version of Java, use the [Docker Hadoop](https://github.com/big-data-europe/docker-hadoop) project to get started:

```
git clone https://github.com/big-data-europe/docker-hadoop.git
cd docker-hadoop
```

Create a local data directory that will be mounted on the NameNode that we can add data files to and then use to add data to HDFS later:

```
mkdir data
```

In the `volumes` key of the `namenode` service in `docker-compose.yml` add this local path, relative to the Compose file:

```
- ./data:/home/data
```

Expose the port of the DataNode web UI.

In the `ports` key of the `datanode` service in `docker-compose.yml` add this port mapping:

```
- 9864:9864
```

Give the DataNode a hostname which we can resolve to access redirects for the DataNode returned by the NameNode when retrieving files using the WebHDFS REST API.

In the `datanode` service in `docker-compose.yml` add the following:

```
hostname: datanode
```

Add a hosts entry for the DataNode to `/etc/hosts` (Linux) or `C:\Windows\System32\drivers\etc\hosts` (Windows).

```
127.0.0.1       datanode
```

Deploy the HDFS cluster:

```
docker-compose up -d
```

Browse to the web UI's where you can explore the status, logs and the file system:
- [NameNode](http://localhost:9870)
- [DataNode](http://localhost:9864)

Add some data to the local data directory to add to HDFS later:

```
echo -n 1,"test" > data/test.csv
```

Check the local data directory is mounted:

```
docker-compose exec namenode ls -alh /home/data
```

List the HDFS root directory:

```
docker-compose exec namenode hdfs dfs -ls -R /
```

Make a HDFS directory to add data to:

```
docker-compose exec namenode hdfs dfs -mkdir -p /db/data
```

Add data to HDFS:

```
docker-compose exec namenode hdfs dfs -copyFromLocal /home/data /db
```

List the HDFS directory:

```
docker-compose exec namenode hdfs dfs -ls -R /db/data
```

Use the [REST API](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html) to get the status of a file:

```
curl -i http://localhost:9870/webhdfs/v1/db/data/test.csv?op=GETFILESTATUS
```

Use the [REST API](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html) to get a file:

```
curl -i -L http://localhost:9870/webhdfs/v1/db/data/test.csv?op=OPEN
```

returns:

```
HTTP/1.1 307 Temporary Redirect
Date: Mon, 25 Jan 2021 17:20:49 GMT
Cache-Control: no-cache
Expires: Mon, 25 Jan 2021 17:20:49 GMT
Date: Mon, 25 Jan 2021 17:20:49 GMT
Pragma: no-cache
X-Content-Type-Options: nosniff
X-FRAME-OPTIONS: SAMEORIGIN
X-XSS-Protection: 1; mode=block
Location: http://datanode:9864/webhdfs/v1/db/data/test.csv?op=OPEN&namenoderpcaddress=namenode:9000&offset=0
Content-Type: application/octet-stream
Content-Length: 0

HTTP/1.1 200 OK
Access-Control-Allow-Methods: GET
Access-Control-Allow-Origin: *
Content-Type: application/octet-stream
Connection: close
Content-Length: 11

1,"test"
```

Delete the HDFS directory:

```
docker-compose exec namenode hdfs dfs -rm -r /db/data
```