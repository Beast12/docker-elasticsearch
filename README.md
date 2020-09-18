# Dockerized Elasticsearch Stack

This compose file let's you set up a elasticsearch cluster with separate Manager/Client nodes.
Also included are the logstash hosts, kibana and the beats.

## Docker file explained:
```
esmaster01docker:
  image: nexus.brusselsairport.be/elasticsearch:7.3.1
  hostname: esmaster01docker
  environment:
    - ELASTIC_PASSWORD=changeme
  deploy:
    mode: replicated
    replicas: 1
    update_config:
      parallelism: 1
      delay: 10s
    placement:
      constraints: [node.hostname == portainer01prod]
    resources:
      limits:
        cpus: '2'
        memory: 8G
      reservations:
        cpus: '2'
        memory: 8G
    restart_policy:
      condition: on-failure
      delay: 5s
      max_attempts: 3
      window: 10s
```
The first part of container (in this case esmaster01prod) will show you general configuration of the container, like the image that is used and the deploy parameters. One Important part is the initial ELASTIC_PASSWORD. This will be used to setup your cluster on the firt run.
```
secrets:
  - source: elasticsearch-docker.crt
    target: /usr/share/elasticsearch/config/elasticsearch-docker.crt
    mode: 0444
    uid: "1000"
    gid: "1000"
  - source: elasticsearch-docker.key
    target: /usr/share/elasticsearch/config/elasticsearch-docker.key
    mode: 0444
    uid: "1000"
    gid: "1000"
  - source: elkstack-ca.crt
    target: /usr/share/elasticsearch/config/ca.crt
    mode: 0444
    uid: "1000"
    gid: "1000"
  - source: elasticsearch-config
    target: /usr/share/elasticsearch/config/elasticsearch.yml
    mode: 0644
    uid: "1000"
    gid: "1000"
  - source: role-mapping-config
    target: /usr/share/elasticsearch/config/role_mapping.yml
    mode: 0644
    uid: "1000"
    gid: "1000"
  - source: elasticsearch-jvm-master
    target: /usr/share/elasticsearch/config/jvm.options
    mode: 0644
    uid: "1000"
    gid: "1000"
  - source: elasticsearch-instances
    target: /usr/share/elasticsearch/config/instances.yml
    mode: 0644
    uid: "1000"
    gid: "1000"
```
The second part are the 'Secrets'. Meaning data that is stored on the swarm and can be used in the container by calling it by it's name in the source section. You will find these secrets below in the file in the "secrets" area. It will become clear then why we do it like this..
```
volumes:
  - type: bind
    source: /etc/yum.repos.d/bac.repo
    target: /etc/yum.repos.d/CentOS-Base.repo
  - type: volume
    source: logs
    target: /var/log
  - type: volume
    source: esmaster-data1
    target: /usr/share/elasticsearch/data
  - type: volume
    source: snapshots
    target: /snapshots
```
The third part is the volumes section. Below this file you can find the creation of the volume volumes.
(https://blog.container-solutions.com/understanding-volumes-docker)
