log-pilot
=========

[![CircleCI](https://circleci.com/gh/AliyunContainerService/log-pilot.svg?style=svg)](https://circleci.com/gh/AliyunContainerService/log-pilot)
[![Go Report Card](https://goreportcard.com/badge/github.com/AliyunContainerService/log-pilot)](https://goreportcard.com/report/github.com/AliyunContainerService/log-pilot)

`log-pilot` is an awesome docker log tool. With `log-pilot` you can collect logs from docker hosts and send them to your centralized log system such as elasticsearch, graylog2, awsog and etc. `log-pilot` can collect not only docker stdout but also log file that inside docker containers.

Try it
======

Prerequisites:

- docker-compose >= 1.6
- Docker Engine >= 1.10

```
# download log-pilot project
git clone git@github.com:AliyunContainerService/log-pilot.git
# build log-pilot image
cd log-pilot/ && ./build-image.sh
# quick start
cd quickstart/ && ./run
```

Then access kibana under the tips. You will find that tomcat's has been collected and sended to kibana.

Create index:
![kibana](quickstart/Kibana.png)

Query the logs:
![kibana](quickstart/Kibana2.png)

Quickstart
==========

### Docker build
```
docker build -t log-pilot:latest -f Dockerfile.filebeat .
```

### Run pilot

```
docker run -d --rm -it \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v /etc/localtime:/etc/localtime \
    -v /:/host:ro \
    -e PILOT_TYPE=filebeat \
    -e LOGGING_OUTPUT=elasticsearch \
    -e ELASTICSEARCH_HOSTS=172.19.13.222:9200 \
    -e ELASTICSEARCH_BULK_MAX_SIZE=1000 \
    -e ELASTICSEARCH_WORKER=3 \
    -e FILEBEAT_MAX_PROCS=4 \
    --cap-add SYS_ADMIN \
    log-pilot:latest
```

### Run applications whose logs need to be collected

Open a new terminal, run the application. With tomcat for example:

```
docker run -it --rm  -p 10080:8080 \
    -v /usr/local/tomcat/logs \
    --label aliyun.logs.catalina=stdout \
    --label aliyun.logs.access=/usr/local/tomcat/logs/localhost_access_log.*.txt \
    tomcat
```

Now watch the output of log-pilot. You will find that log-pilot get all tomcat's startup logs. If you access tomcat with your broswer, access logs in `/usr/local/tomcat/logs/localhost_access_log.\*.txt` will also be displayed in log-pilot's output.

More Info: [Fluentd Plugin](docs/fluentd/docs.md) and [Filebeat Plugin](docs/filebeat/docs.md)

Feature
========

- Support both [fluentd plugin](docs/fluentd/docs.md) and [filebeat plugin](docs/filebeat/docs.md). You don't need to create new fluentd or filebeat process for every docker container.
- Support both stdout and log files. Either docker log driver or logspout can only collect stdout.
- Declarative configuration. You need do nothing but declare the logs you want to collect.
- Support many log management: elastichsearch, graylog2, awslogs and more.
- Tags. You could add tags on the logs collected, and later filter by tags in log management.

Build log-pilot
===================

Prerequisites:

- Go >= 1.6

```
go get github.com/AliyunContainerService/log-pilot
cd $GOPATH/github.com/AliyunContainerService/log-pilot
# This will create a new docker image named log-pilot:latest
./build-image.sh
```

Contribute
==========

You are welcome to make new issues and pull reuqests.

