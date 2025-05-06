# Deploy Wazuh Docker in single node configuration

This deployment is defined in the `docker-compose.yml` file with one Wazuh manager containers, one Wazuh indexer containers, and one Wazuh dashboard container. It can be deployed by following these steps: 

1) Increase max_map_count on your host (Linux). This command must be run with root permissions:
```
$ sysctl -w vm.max_map_count=262144
```
2) Run the certificate creation script:
```
$ docker-compose -f generate-indexer-certs.yml run --rm generator
```
3) Start the environment with docker-compose:

- In the foregroud:
```
$ docker-compose up
```
- In the background:
```
$ docker-compose up -d
```

The environment takes about 1 minute to get up (depending on your Docker host) for the first time since Wazuh Indexer must be started for the first time and the indexes and index patterns must be generated.

## Configuration for syslog

### 1. Change configuraation Wazuh Manager

```
$ vim /var/ossec/etc/ossec.conf
```

```
  <global>
    <jsonout_output>yes</jsonout_output>
    <alerts_log>yes</alerts_log>
    <logall>no</logall>
    <logall_json>yes</logall_json>
    <email_notification>no</email_notification>
    <smtp_server>smtp.example.wazuh.com</smtp_server>
    <email_from>wazuh@example.wazuh.com</email_from>
    <email_to>recipient@example.wazuh.com</email_to>
    <email_maxperhour>12</email_maxperhour>
    <email_log_source>alerts.log</email_log_source>
    <agents_disconnection_time>10m</agents_disconnection_time>
    <agents_disconnection_alert_time>0</agents_disconnection_alert_time>
  </global>
```

```
  <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>tcp</protocol>
    <allowed-ips>0.0.0.0/0</allowed-ips><!-->Mandatory</-->
    <local_ip>192.168.18.30</local_ip>
  </remote>
```

### 2. Enable module archives in Filebeat configuration

```
$ vim /etc/filebeat/filebeat.yml
```

```
filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: true
```

