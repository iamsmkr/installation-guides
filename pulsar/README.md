## Zookeeper 
**Note**: Ignore this section if you intend to start zookeeper that comes along with pulsar distribution.

1. Download Zookeeper
    ```sh
    $ wget https://dlcdn.apache.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz
    ```

2. Update `zoo.cfg`

    Make following changes to the configuration file or copy [zoo.cfg](https://github.com/iamsmkr/installation-guides/blob/main/pulsar/zoo.cfg) to `/opt/zookeeper/conf`.
    ```
    dataDir=/opt/zookeeper/data
    autopurge.purgeInterval=12
    admin.enableServer=true
    admin.serverPort=2182
    4lw.commands.whitelist=*
    ```
    
3. Start Zookeeper
    ```sh
    $ ./bin/zkServer.sh start
    ```

4. Verify Zookeeper 
    ```sh
    $ echo stat | nc localhost 2181
    ```

## Pulsar
1. Download and install pulsar
    ```sh
    $ wget https://archive.apache.org/dist/pulsar/pulsar-2.9.1/apache-pulsar-2.9.1-bin.tar.gz
    $ tar xvfz apache-pulsar-2.9.1-bin.tar.gz
    ```

2. Update `zookeeper.cfg`

    Make following changes to the configuration file `zookeeper.conf` or copy [zoo.cfg](https://github.com/iamsmkr/installation-guides/blob/main/pulsar/zoo.cfg) to `conf` and rename to `zookeeper.conf`.
    ```
    autopurge.purgeInterval=12
    admin.enableServer=true
    admin.serverPort=2182
    4lw.commands.whitelist=*
    metricsProvider.httpPort=7001
    ```
    
3. Start zookeeper
    ```sh
    $ ./bin/pulsar-daemon start zookeeper
    ```

4. Verify Zookeeper 
    ```sh
    $ echo stat | nc localhost 2181
    ```

5. Initialize cluster metadata in Zookeeper
    ```sh
    $ ./bin/pulsar initialize-cluster-metadata \
      --cluster pulsar-cluster-1 \
      --zookeeper localhost:2181 \
      --configuration-store localhost:2181 \
      --web-service-url http://localhost:8080 \
      --web-service-url-tls https://localhost:8443 \
      --broker-service-url pulsar://localhost:6650 \
      --broker-service-url-tls pulsar+ssl://localhost:6651
    ```

6. Update `bookkeeper.conf`
    
    Make following changes to the configuration file or copy [bookkeeper.conf](https://github.com/iamsmkr/installation-guides/blob/main/pulsar/bookkeeper.conf) to `/opt/pulsar/conf`.
    ```
    allowLoopback=true
    journalMaxSizeMB=2048
    logSizeLimit=10240000
    flushEntrylogBytes=0
    diskUsageThreshold=0.99
    diskUsageWarnThreshold=0.99
    diskUsageLwmThreshold=0.99
    dlog.bkcEnsembleSize=1
    dlog.bkcWriteQuorumSize=1
    dlog.bkcAckQuorumSize=1
    ```

7. Start Bookkeeper
    ```sh
    $ ./bin/pulsar-daemon start bookie  # Or,
    $ ./bin/pulsar bookie   # This doesn't create logs under `logs` directory
    ```

8. Verify Bookkeeper
    ```sh
    $ ./bin/bookkeeper shell bookiesanity
    ```
    
9. Update `broker.conf`
    
    Make following changes to the configuration file or copy [broker.conf](https://github.com/iamsmkr/installation-guides/blob/main/pulsar/broker.conf) to `/opt/pulsar/conf`.
    ```
    zookeeperServers=localhost:2181
    brokerServicePortTls=6651
    webServicePortTls=8443
    clusterName=pulsar-cluster-1
    brokerDeleteInactiveTopicsEnabled=false
    brokerDeleteInactiveTopicsFrequencySeconds=3600
    brokerDeleteInactiveTopicsMaxInactiveDurationSeconds=7200
    managedLedgerDefaultEnsembleSize=1
    managedLedgerDefaultWriteQuorum=1
    managedLedgerDefaultAckQuorum=1
    defaultRetentionTimeInMinutes=-1
    defaultRetentionSizeInMB=-1
    ```

10. Start Pulsar
    ```sh
    $ ./bin/pulsar-daemon start broker  # Or,
    $ ./bin/pulsar broker   # This doesn't create logs under `logs` directory
    ```

11. Verify Pulsar
    ```sh
    $ ./bin/pulsar-admin topics create public/default/topic1
    $ ./bin/pulsar-client consume persistent://public/default/topic1 -s "subs1"
    $ ./bin/pulsar-client produce persistent://public/default/topic1 --messages "msg1"
    ```

## Node Exporter
1. Download Node Exporter
    ```sh
    $ wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
    $ tar -zvxf node_exporter-1.3.1.linux-amd64.tar.gz
    ```

2. Start Node Exporter
    ```sh
    $ cd node_exporter-1.3.1.linux-amd64/
    $ ./node_exporter
    ```

### On macOS
1. Install Node Exporter
    ```sh
    $ brew install node_exporter
    ```

2. Start Node Exporter
    ```sh
    $ node_exporter
    ```

3. Verify Installation
    ```sh
    $ curl http://localhost:9100/metrics
    ```

## Pulsar Manager
1. Download Pulsar Manager
    ```sh
    $ wget https://dist.apache.org/repos/dist/release/pulsar/pulsar-manager/pulsar-manager-0.2.0/apache-pulsar-manager-0.2.0-bin.tar.gz
    $ tar -xvzf apache-pulsar-manager-0.2.0-bin.tar.gz
    $ tar -xvf pulsar-manager/pulsar-manager.tar
    ```

2. Setup UI
    ```sh
    $ cp -r pulsar-manager/dist pulsar-manager/pulsar-manager/ui
    ```

3. Update `bkvm.conf`
    
    Make following changes to the configuration file or copy [bkvm.conf](https://github.com/iamsmkr/installation-guides/blob/main/pulsar/bkvm.conf) to `/opt/pulsar-manager/pulsar-manager/`.
    ```conf
    bkvm.enabled=false
    ```

3. Start Pulsar Manager 
    ```sh
    $ ./bin/pulsar-manager --redirect.host=http://localhost \
        --redirect.port=9527 insert.stats.interval=600000 \
        --backend.jwt.token=token \
        --jwt.broker.token.mode=PRIVATE \
        --jwt.broker.private.key=file:///path/broker-private.key \
        --jwt.broker.public.key=file:///path/broker-public.key 
    ```

4. Create User
    ```sh
    $ curl  -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \  
          -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
          -H "Content-Type: application/json" \
          -X PUT http://localhost:7750/pulsar-manager/users/superuser  \
          -d '{"name": "admin", "password": "apachepulsar", "description": "test", "email": "username@test.org"}'
    ```

5. Create tunnel (if required)
    ```sh
    $ ssh -i aws.pem -N -f -L 7750:localhost:7750 ubuntu@ec2-3-82-189-53.compute-1.amazonaws.com
    ```

6. Create Environment

    - Goto pulsar manager @ http://localhost:7750/ui/index.html 
    - Login using username/password as `admin/apachepulsar`
    - Create an environment with following details:
    
        ```
        Environment Name = standalone
        Service URL      = http://127.0.0.1:8080
        ```


## Prometheus
1. Copy Prometheus template [prometheus.yml](https://github.com/iamsmkr/installation-guides/blob/main/pulsar/prometheus.yml) to `/opt/prometheus`.

2. Run Prometheus docker container
    ```sh
    $ docker run --network="host" -p 8080:8080 -p 9090:9090 \
        -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
        prom/prometheus
    ```

3. Create tunnel (if required)
    ```sh
    $ ssh -i aws.pem -N -f -L 9090:localhost:9090 ubuntu@ec2-3-82-189-53.compute-1.amazonaws.com
    $ ssh -i aws.pem -N -f -L 8080:localhost:8080 ubuntu@ec2-3-82-189-53.compute-1.amazonaws.com
    ```

4. Access Prometheus
    
    - Access targets @ http://localhost:9090/classic/targets
 
5. Access Metrics

    - Access broker metrics @ http://localhost:8080/metrics/
    - Access bookie metrics @ http://localhost:8000/metrics/
    - Access zookeeper metrics @ http://localhost:7001/metrics/
    - Access node metrics @ http://localhost:9100/metrics/

### On macOS
On mac, `opt` directory not already added as shared path from host to docker will return following error:
```
docker: Error response from daemon: Mounts denied:
The path /opt/prometheus/prometheus.yml is not shared from the host and is not known to Docker.
You can configure shared paths from Docker -> Preferences... -> Resources -> File Sharing.
See https://docs.docker.com/desktop/mac for more info.
```

Add `opt` to `~/Library/Group Containers/group.com.docker/settings.json` and restart docker daemon.
```
"filesharingDirectories" : [
  "\/Users",
  "\/Volumes",
  "\/datadrive",
  "\/private",
  "\/tmp",
  "\/opt"
],
```

## Grafana
1. Run Grafana docker container
    ```sh
    $ docker run --network="host" -p 3000:3000 \
        -e PULSAR_PROMETHEUS_URL="http://localhost:9090" \
        -e PULSAR_CLUSTER="pulsar-cluster-1" \
        streamnative/apache-pulsar-grafana-dashboard:latest
    ```

2. Create tunnel (if required)
    ```sh
    $ ssh -i aws.pem -N -f -L 3000:localhost:3000 ubuntu@ec2-3-82-189-53.compute-1.amazonaws.com
    ```

3. Access dashboards

   - Goto Grafana @ http://localhost:3000/ 
   - Login using username and password as `admin/happypulsaring`

## Zoo Navigator

1. Start a web-based??**ZooKeeper UI and editor/browser**??
    
    ```bash
    $ docker run -d -p 9000:9000 \
      -e HTTP_PORT=9000 \
      --name zoonavigator \
      --restart unless-stopped \
      elkozmon/zoonavigator:latest
    ```
    
2. Create tunnel (if required)
    ```bash
    $ ssh -i aws_ethereum.pem -N -f -L 9001:localhost:9000 \
    ubuntu@ec2-3-82-189-52.compute-1.amazonaws.com
    ```

---

## Important Notes
- In order to start pulsar clean just delete `data/*` directory and reinitialize cluster metadata in Zookeeper like so:

    ```sh
    $ ./bin/pulsar initialize-cluster-metadata \
      --cluster pulsar-cluster-1 \
      --zookeeper localhost:2181 \
      --configuration-store localhost:2181 \
      --web-service-url http://localhost:8080 \
      --web-service-url-tls https://localhost:8443 \
      --broker-service-url pulsar://localhost:6650 \
      --broker-service-url-tls pulsar+ssl://localhost:6651
    ```
