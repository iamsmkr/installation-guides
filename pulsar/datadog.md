## Datadog
In order to use datadog to monitor pulsar instance:

1. Create datadog account

2. Install datadog agent
    ```
    $ DD_INSTALL_ONLY=true DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=5840c4202acaee60b8375ff645949b30 DD_SITE="datadoghq.eu" \
    bash -c "$(curl -L https://s3.amazonaws.com/dd-agent/scripts/install_script.sh)"
    ```

    **Note**: If the Agent is not already installed on your machine and you don't want it to start automatically after the installation, just prepend `DD_INSTALL_ONLY=true` to the above script before running it.

    [![Screenshot-2022-02-27-at-11-01-57-PM.png](https://i.postimg.cc/SNh4sXK4/Screenshot-2022-02-27-at-11-01-57-PM.png)](https://postimg.cc/DmBHpwsp)


3. Add `OpenMetrics` and `Pulsar` integrations

    [![Screenshot-2022-02-27-at-11-04-04-PM.png](https://i.postimg.cc/DZGVfFkm/Screenshot-2022-02-27-at-11-04-04-PM.png)](https://postimg.cc/5YfKphtM)

4. Clone `streamnative/pulsar-datadog` repo
    ```
    $ git clone git@github.com:streamnative/pulsar-datadog.git
    $ cd pulsar-datadog
    ```

5. Copy configs to datadog agent conf dir

    Enable datadog collector for Pulsar component by copying the collector configuration files are under the `configs` directory to the OpenMetrics configuration directory of the Datadog agent and then restart the agent.
    ```
    $ cp configs/* /etc/datadog-agent/conf.d/openmetrics.d
    $ sudo systemctl restart datadog-agent
    ```

6. Import dashboards in datadog ui
    - Create a dashboard
    
      [![Screenshot-2022-02-27-at-11-17-37-PM.png](https://i.postimg.cc/h4HY4Np7/Screenshot-2022-02-27-at-11-17-37-PM.png)](https://postimg.cc/ThJ0CN8R)      
      
    - Pick a layout
    
      [![Screenshot-2022-02-27-at-11-13-01-PM.png](https://i.postimg.cc/NMCxn2PH/Screenshot-2022-02-27-at-11-13-01-PM.png)](https://postimg.cc/34pp4Nh8)
      [![Screenshot-2022-02-27-at-11-13-16-PM.png](https://i.postimg.cc/x8W3fDQj/Screenshot-2022-02-27-at-11-13-16-PM.png)](https://postimg.cc/jn47c1CB)
      
    - Import the dashboards from `pulsar-datadog/dashboards`
    
      [![Screenshot-2022-02-27-at-11-13-33-PM.png](https://i.postimg.cc/NjvkZcbp/Screenshot-2022-02-27-at-11-13-33-PM.png)](https://postimg.cc/bsLn247t)
      [![Screenshot-2022-02-27-at-11-16-21-PM.png](https://i.postimg.cc/hGp8J7n5/Screenshot-2022-02-27-at-11-16-21-PM.png)](https://postimg.cc/QBKKwCHQ)
      
 7. Your dashboards are ready to monitor
 
    ![Screenshot-2022-02-27-at-11-01-57-PM.png](https://github.com/streamnative/pulsar-datadog/blob/master/images/Broker.png)

    
