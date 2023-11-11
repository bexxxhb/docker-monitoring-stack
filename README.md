# FIX for "Failed to upgrade legacy queries Datasource ${DS_PROMETHEUS} was not found"

Workarounds that worked in Grafana 9.1.5:

Storyline:

So you might be like me, you never defined a datasource UID in your provisioning file. You made a cool dashboard, then clicked "Share" and exported to JSON. Or you might have gone to Dashboard settings and selected "View as JSON" then copy-and-pasta'ed that json into a dashboard made through provisioning. Should be straight-forward, right?, but then you bring your Dashboard.json to a new Grafana instance only to find the data didn't load. For me, there wasn't even an error or log which was frustrating.

The Workaround:

Open your dashboard json file.
Find the UID that Grafana assigned to the datasource in the JSON. This will either look like a random string (e.g. PBFA97CFB590B2093 or it'll be the variable form ${DS_PROMETHEUS}, which is used when telling Grafana to "Share Externally". Find this value, it should be associated with your first (and subsequent) panel defined in the JSON (short scroll down).
In your text editor do a find and replace. "Find" your UID from step 2, (XXXXXXXXXXX or ${DS_PROMETHEUS}) and Replace it with a null string.
Find: "${DS_PROMETHEUS}"
Replace: ""
With the datasource UID undefined, the graph should now load up as expected.
Long-term Workaround:

You need to define an explicit UID for your datasource. For each provisioned datasource, Grafana allows you to specify an explicit UID for the datasource. If do not plan to share your dashboards with random people, you'll be okay to set an UID per datasource that you have. Use that UID across all environments that your dashboards will be shared in. This will allow you to Export/Import dashboards between container tear downs, keeping your teammates happy.

Reference to what I'm talking about on the Grafana docs:
https://grafana.com/docs/grafana/latest/administration/provisioning/#example-data-source-config-file

In short, add uid: <your-defined-uid-it-can-be-anything> to your datasource provisioning yaml:

datasources:
  - name: Prometheus
    type: prometheus
    orgId: 1
    url: http://my-prometheus:9090
    password:
    user:
    database:
    basicAuth: false
    basicAuthUser:
    basicAuthPassword:
    withCredentials:
    isDefault: true
    version: 1
    editable: true
    uid: myotheruidisanairplane
This will force Grafana to output all exported dashboards with the uid "myotheruidisanairplane". And as you redeploy Grafana, it'll always name your Prometheus instance "myotheruidisanairplane", thus not breaking importing your exported dashboards.

If you're actually sharing your dashboards with random people on the internet

Follow the workaround, and find-and-replace all UIDs to be a null-string.

# docker-monitoring-stack-gpnc
Grafana Prometheus Node-Exporter cAdvisor - Docker Monitoring Stack

## Makefile

[Note](https://docs.docker.com/compose/install/linux/): Due to `docker-compose` and the `compose` plugin, you might have one of the two installed. I have a `Makefile` that will detect which on you have installed.

You can list the targets using `make`.

## Boot

Boot the stack with docker compose (or `make up`):

```bash
docker-compose up -d
```

Ensure all containers are running:

```bash
docker-compose ps
```

The output should looke like this:

```bash
    Name                   Command                  State               Ports         
--------------------------------------------------------------------------------------
cadvisor        /usr/bin/cadvisor -logtostderr   Up (healthy)   8080/tcp              
grafana         /run.sh                          Up             0.0.0.0:3000->3000/tcp
node-exporter   /bin/node_exporter --path. ...   Up             9100/tcp              
prometheus      /bin/prometheus --config.f ...   Up             0.0.0.0:9090->9090/tcp
alertmanager    /bin/alertmanager --config ...   Up             0.0.0.0:9093->9093/tcp
```

## Access Grafana

Access grafana on [Grafana Home](http://localhost:3000/?orgId=1) (or `make open`) and you should see the two dashboards that was provisioned:

![](./assets/grafana-home.png)

Once you select the nodes dashboard, it should look something like this:

![](./assets/grafana-dashboard.png)

When you select ["Alerting" and "Alert rules"](http://localhost:3000/alerting/list) you will find the recording and alerting rules:

![](./assets/grafana-alerting-home.png)

We can expand the alerting rules:

![](./assets/grafana-alerting-rules.png)

And then we can view more detail on a alert rule:

![](./assets/grafana-alerting-detail.png)

And for our container metrics we can access the **Container Metrics** dashboard:

![](./assets/grafana-container-metrics.png)

## Endpoints

The following endpoints are available:

| Container      | Internal Endpoint         | External Endpoint     |
| -------------- | ------------------------- |---------------------- |
| Grafana        | http://grafana:3000       | http://localhost:3000 |
| Prometheus     | http://prometheus:9090    | http://localhost:9090 |
| Node-Exporter  | http://node-exporter:9100 | http://localhost:9100 |
| cAdvisor       | http://cadvisor:8080      | N/A                   |
| Alertmanager   | http://alertmanager:9093  | http://localhost:9093 |

## Cleanup

To remove the containers using docker compose (or `make clean`):

```bash
docker-compose down
```

## Stargazers over time

[![Stargazers over time](https://starchart.cc/ruanbekker/docker-monitoring-stack-gpnc.svg)](https://starchart.cc/ruanbekker/docker-monitoring-stack-gpnc)

## Resources

Heavily inspired from [this exporter guide](https://grafana.com/oss/prometheus/exporters/node-exporter/)
