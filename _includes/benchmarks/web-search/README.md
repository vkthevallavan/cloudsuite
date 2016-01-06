# Web Search #

[![Pulls on DockerHub][dhpulls]][dhrepo]
[![Stars on DockerHub][dhstars]][dhrepo]

This repository contains the docker image for Cloudsuite's Web Search benchmark.

The Web Search benchmark relies on the [Apache Solr][apachesolr] search engine framework. The benchmark includes a client machine that simulates real-world clients that send requests to the index nodes. The index nodes contain an index of the text and fields found in a set of crawled websites.

## Using the benchmark ##

### Dockerfiles ###

Supported tags and their respective `Dockerfile` links:

- [`data`][datadocker] This builds a volume image with the benchmark's dataset, which is an index generated by crawling a set of websites.
- [`server`][serverdocker] This builds an image for the Apache Solr index nodes. You may spawn several nodes.
- [`client`][clientdocker] This builds an image with the client node. The client is used to start the benchmark and query the index nodes.

These images are automatically built using the mentioned Dockerfiles available on [`CloudSuite-EPFL/WebSearch`][repo].

### Creating a network between the server(s) and the client(s)

To facilitate the communication between the client(s) and the server(s), we build a docker network:

	$ docker network create search_network

We will attach the launched containers to this newly created docker network.

### Starting the volume images ###

The first step is to create the volume images that contain the dataset of the Web Search benchmark. First `pull` the volume images, using the following command:

	$ docker pull cloudsuite/websearch:data

The following command will start the volume images, making both the data available for other docker images on the host:

	$ docker create --name data cloudsuite/websearch:data

### Starting the server (Index Node) ###

To start the server you have to first `pull` the server image and then run it. To `pull` the server image, use the following command:

	$ docker pull cloudsuite/websearch:server

The following command will start the server and forward port 8983 to the host, so that the Apache Solr's web interface can be accessed from the web browser using the host's IP address. More information on Apache Solr's web interface can be found [here][solrui].

	$ docker run -it --volumes-from data --name server --net search_network -p 8983:8983 cloudsuite/websearch:server

### Starting the client and running the benchmark ###

To start a client you have to first `pull` the client image and then run it. To `pull` the client image, use the following command:

	$ docker pull cloudsuite/websearch:client

The following command will start the client node and run the benchmark. The `server_address` refers to the IP address, in brackets (e.g., "172.19.0.2"), of the index node that receives the client requests. The four numbers after the server address refer to: the scale, which indicates the number of concurrent clients (50); the ramp-up time in seconds (90), which refers to the time required to warm up the server; the steady-state time in seconds (60), which indicates the time the benchmark is in the steady state; and the rump-down time in seconds (60), which refers to the time to wait before ending the benchmark. Tune these parameters accordingly to stress your target system.

	$ docker run -it --volumes-from data --name client --net search_network cloudsuite/websearch:client server_address 50 90 60 60  

The output results will show on the screen after the benchmark finishes.

[datadocker]: https://github.com/CloudSuite-EPFL/WebSearch/tree/master/data/Dockerfile "Data volume Dockerfile"
[serverdocker]: https://github.com/CloudSuite-EPFL/WebSearch/tree/master/server/Dockerfile "Server Dockerfile"
[clientdocker]: https://github.com/CloudSuite-EPFL/WebSearch/tree/master/client/Dockerfile "Client Dockerfile"
[solrui]: https://cwiki.apache.org/confluence/display/solr/Overview+of+the+Solr+Admin+UI "Apache Solr UI"
[apachesolr]: https://github.com/apache/solr "Apache Solr"
[repo]: https://github.com/CloudSuite-EPFL/WebSearch "Web Search GitHub Repo"
[dhrepo]: https://hub.docker.com/r/cloudsuite/websearch/ "DockerHub Page"
[dhpulls]: https://img.shields.io/docker/pulls/cloudsuite/websearch.svg "Go to DockerHub Page"
[dhstars]: https://img.shields.io/docker/stars/cloudsuite/websearch.svg "Go to DockerHub Page"
