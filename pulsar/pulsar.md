## Zookeeper

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
2. Initialize cluster metadata in Zookeeper
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

3. Update `bookkeeper.conf`
    
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

3. Start Bookkeeper
    ```sh
    $ ./bin/pulsar bookie   # Or,
    $ ./bin/pulsar-daemon start bookie 
    ```

4. Verify Bookkeeper
    ```sh
    $ ./bin/bookkeeper shell bookiesanity
    ```
    
5. Update `broker.conf`
    
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

6. Start Pulsar
    ```sh
    $ ./bin/pulsar-daemon start broker  # Or,
    $ ./bin/pulsar broker
    ```

7. Verify Pulsar
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

    - Access pulsar manager @ http://localhost:7750/ui/index.html and create an environment with following details:
    
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
    - Access metrics @ http://localhost:8080/metrics/

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

   - Access Grafana @ http://localhost:3000/ 
   - Login using username and password as `admin/happypulsaring`

