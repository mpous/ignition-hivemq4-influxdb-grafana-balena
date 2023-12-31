# Industrial IoT Gateway using Ignition Edge with HiveMQ and balena

This is an IIoT reference architecture to run on your IIoT Edge Gateway with HiveMQ MQTT broker, Influx, Grafana and Ignition Edge running on a [balena](https://balena.io) device.


![](/assets/balena-ignition-hivemq.png)


The example application is reading from Modbus and OPC UA sensors using a variation of the MING stack (MQTT HiveMQ, InfluxDB, Ignition Edge and Grafana) to use the advantages of the Edge Computing to digitalize a factory. In this case, we are using Inductive Automation Ignition Edge, HiveMQ and balena to add a Unified Name Space (UNS) for data modeling. We are going to use MQTT Sparkplug B to format the data being sent over MQTT to Ignition in the cloud through HiveMQ Cloud Enterprise instance.


## Disclaimer

This project is for educational purposes only. Do not deploy it into your premises without understanding what you are doing. USE THE SOFTWARE AT YOUR OWN RISK. THE AUTHORS AND ALL AFFILIATES ASSUME NO RESPONSIBILITY FOR YOUR SECURITY.

We strongly recommend you to have some coding and networking knowledge. Do not hesitate to read the source code and understand the mechanism of this project or contact the authors.


## Deploy the code

Running this project is as simple as deploying it to a balenaCloud application. You can do it in just one click by using the button below:

[![](https://www.balena.io/deploy.png)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena)

Follow instructions, click Add a Device and flash an SD card with that OS image dowloaded from balenaCloud. Enjoy the magic 🌟Over-The-Air🌟!

![balenaCloud-ignition-hiveMQ-influx-grafana](https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena/assets/173156/b9015ebc-8e15-4913-ae96-b520d4cf1280)


## Log in to the services

The services are exposed in different ports. On the default configuration they use the default port and credentials to access them. Check the documentation for each of them to know how to change them using variables.

|Service|Port|Username|Password|
|:--|---|---|---|
|HiveMQ|8080 (http)|admin|hivemq|
|Ignition|9088 (http)|balena|balena|
|Grafana|3000 (http)|admin|admin|
|InfluxDB|8086 (http) |balena|balenabalena|


## Configure Ignition Edge

Once the application is successfully deployed on your balenaCloud fleet, you should be able to see your devices running the services based on the MING stack philosophy. The services for this application are:

* Ignition Edge to capture all the data from sensors and PLCs.
* HiveMQ4 as MQTT broker to enable an event driven architecture.
* InfluxDB as local time-series database.
* Grafana as a local data visualization tool.

Access to the Ignition UI using your local ip address with the `9088` port and install Ignition Edge. Create a username and password for Ignition and once the Ignition Edge is successfully installed, access to the Ignition Edge.

![Ignition Edge running on balena](https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena/assets/173156/19f33163-7239-4bb3-8361-f3b255fa90fe)


To receive the data from the edge IIoT gateway with HiveMQ and balena you will need to install the MQTT module packages on Ignition Edge made by Cirrus Link. For this project we will only need to install the `MQTT engine` and the `MQTT Transmission` modules. Download the [MQTT Cirrus Link modules](https://inductiveautomation.com/downloads/third-party-modules/8.1.27) for Ignition here.

![MQTT Engine and MQTT Transmission running on Ignition Edge](https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena/assets/173156/1afc0657-1270-452c-ac8f-93b07f2c9df7)

When the MQTT Modules are successfully installed, you can go to the left menu on MQTT and create the servers (`Engine` and `Transmission`). Remember that the URL starts with tcp:// and add the port 1883 or your MQTT port from the edge machine MQTT broker.

![Configure the MQTT modules successfully](https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena/assets/173156/c8be62cd-5d95-49ec-aaea-379acfdf145d)

Once you start capturing the data, check with a tool such as MQTT Explorer or similar if the data goes through your local MQTT broker.

![MQTT Explorer](https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena/assets/173156/97a70e87-1f49-4940-bad4-4a646282eff2)

Remember that it will be encoded using Sparkplug B as we explain in the blogpost.


## Configure the HiveMQ MQTT broker

### Configure the Sparkplug InfluxDB extension

As we are using Sparkplug B with MQTT, through Ignition, we will install the [HiveMQ Sparkplug InfluxDB extension](https://www.hivemq.com/extension/sparkplug-influxdb-extension/) in the HiveMQ service. The aim of this extension is to store automatically all the data (and metadata) that is published to the broker to InfluxDB. To manage the extension you will need to configure some device variables through balenaCloud and during the deployment. 

Variable Name | Value | Description | Default
------------ | ------------- | ------------- | -------------
**DOCKER_INFLUXDB_INIT_MODE** | `string` | Value is `setup` or `cloud` | `setup`
**DOCKER_INFLUXDB_INIT_BUCKET** | `string` | Bucket's name | `balena`
**DOCKER_INFLUXDB_INIT_ORG** | `string` | Organization name | `balena`
**DOCKER_INFLUXDB_INIT_USERNAME** | `string` |  InfluxDB UI username | `balena`
**DOCKER_INFLUXDB_INIT_PASSWORD** | `string` |  InfluxDB UI passowrd | `balenabalena`
**DOCKER_INFLUXDB_INIT_ADMIN_TOKEN** | `string` | Token to interact with the Influx via API. It needs to be the same as the variable `HIVEMQ_SPARKPLUG_TOKEN`. | `PB0Pjh4_iou_uNkww4unb-WkGQ-tTe3KoWbUsb4O9rlxOhuj3kgkhD7tH1jXZxzleNob8WzONGbh0tM1SjHQWQ==`

These variables will set up the HiveMQ Sparkplug extension that will send the data through InfluxDB, as previously set up.

Variable Name | Value | Description | Default
------------ | ------------- | ------------- | -------------
**HIVEMQ_SPARKPLUG_ADDRESS** | `string` | Name of the service running locally on balena or address. | `influx`
**HIVEMQ_SPARKPLUG_BUCKET** | `string` | Bucket's name | `balena`
**HIVEMQ_SPARKPLUG_DATABASE** | `string` | Database's name | `balena`
**HIVEMQ_SPARKPLUG_MODE** | `string` | InfluxDB mode | `setup`
**HIVEMQ_SPARKPLUG_ORGANIZATION** | `string` | Organization's name | `balena`
**HIVEMQ_SPARKPLUG_PORT** | `string` |  InfluxDB port | `8086`
**HIVEMQ_SPARKPLUG_TOKEN** | `string` | Token to interact with the Influx via API. It needs to be the same as the variable `DOCKER_INFLUXDB_INIT_ADMIN_TOKEN` | `PB0Pjh4_iou_uNkww4unb-WkGQ-tTe3KoWbUsb4O9rlxOhuj3kgkhD7tH1jXZxzleNob8WzONGbh0tM1SjHQWQ==`


### Configure the bridge extension

These variables you can set them in the balenaCloud `Device Variables` tab for the device (or globally for the whole application). None of them are mandatory and the MQTT broker in the edge might work without any variable defined.

Variable Name | Value | Description | Default
------------ | ------------- | ------------- | -------------
**HIVEMQ_BRIDGE_EXTENSION** | `boolean` | Enable bridge extension and delete the `DISABLED` file on the bridge-extension folder | `false`
**HIVEMQ_HOST_URL** | `STRING` | URL of the Host connected via the Bridge Extension. | ```HIVEMQ_HOST_URL```
**HIVEMQ_HOST_PORT** | `STRING` | Port of the host connected via the Bridge Extension. | ```HIVEMQ_HOST_PORT```
**HIVEMQ_HOST_USERNAME** | `STRING` | Username of the MQTT simple authentication to connect the Bridge Extension. | ```HIVEMQ_HOST_USERNAME```
**HIVEMQ_HOST_PASSWORD** | `STRING` | Password of the MQTT simple authentication to connect the Bridge Extension. | ```HIVEMQ_HOST_PASSWORD```
**HIVEMQ_TLS_ENABLED** | `boolean` | Adds the TLS tags to use the MQTT over TLS through the Bridge Extension. | ```false```
**HIVEMQ_TOPICS_CONFIGURATION** | `STRING (XML)` | Topic tag XML definition on the bridge-extension.xml file. | ```<topics><topic> What it goes here </topic></topics>```
**HIVEMQ_LICENSE** | `STRING` | Your license file cntent in one unique line separated by \| Automatically the system will generate a `license.lic` file with the base64 content from this variable. | 
**HIVEMQ_REST_API_ENABLED** | `boolean` | Enables to change the config.xml file with the `rest-api` tag. | `false`
**HIVEMQ_REST_API_CONFIGURATION** | `STRING (XML)` | REST API tag XML definition on the config.xml file. | ```<rest-api><enabled>true</enabled><listeners><http><port>8888</port><bind-address>0.0.0.0</bind-address></http></listeners></rest-api>```


## Check if everything works as expected

Once the data is stored locally in the local time-series database from the local MQTT broker managed by HiveMQ. We also see that the local HiveMQ MQTT broker is publishing the data to the HiveMQ Cloud Enterprise system. We can test this with the MQTT Explorer or similar tools.

The next steps will be to configure the HiveMQ Cloud Enterprise account in order to use the cloud MQTT broker in order have all the Unified Namespace centralized in the cloud getting bridged data from all the IIoT edge gateways deployed on shop floors.


![Architecture diagram](https://github.com/mpous/ignition-hivemq4-influxdb-grafana-balena/assets/173156/511f66b7-1df2-47c6-986c-47e5ef5d2ff5)


Once you receive properly data on the HiveMQ Cloud enterprise free account, you can deploy Ignition Enterprise on another balena device in order to visualize the data from multiple gateways centralized on a unique SCADA. Read [how to deploy Ignition on balena here](https://github.com/mpous/ignition-balena).


## Attribution

- This is in part working thanks of the work of [Kudzai Manditereza](https://github.com/kmanditereza) from HiveMQ and Marc Pous from balena.io.

## Troubleshooting

If you detect any issue using this block, feel free to contact us at the [forums.balena.io](https://forums.balena.io).

