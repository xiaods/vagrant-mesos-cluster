vagrant-mesos-cluster
=====================

A vagrant configuration to set up a cluster of mesos master, slaves and zookeepers through ansible

# Usage

Clone the repository, and run:

```
vagrant up
```

This will provision a mini Mesos cluster with one master, one slave, and one
HAProxy instance.  The Mesos master server also contains Zookeeper and the
Marathon framework. The slave will come with Docker installed.


# Running applications

After provisioning the servers you can access Marathon UI here:
http://100.0.10.11:8080/ and the Mesos UI here: http://100.0.10.11:5050/

You can run the test suite of apps by executing this command in the root folder:

```
./run.sh
```

This will spawn 3 different applications:

- busybox: A container continuously looping and printing the current `date`
- http-server: A simple HTTP server listening on service port 15000, monitored by a healthcheck
- curl: A container continuously looping and executing a `curl` against the previous http-server app on port 15000

## Run custom apps

Create a [Marathon application configuration JSON file](https://mesosphere.github.io/marathon/docs/generated/api.html#v2_apps_post) and run it by executing:

```
curl -X POST -H "Content-Type: application/json" http://100.0.10.11:8080/v2/apps -d@path/to/my/app-config.json
```
