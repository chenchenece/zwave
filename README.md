# ZWave Device Connector 
Zwave device connector for BuildingDepotV3.6 [Done! :tada:]

## Update on 18 January 2017:

* Fix issues of library compatibility. Now there are need for helper module `post_bd.py`;

## Update on 17 January 2017:

*  Add features for dealing with alarm type signal, e.g. motion, vibrator sensor;

## Update on 16 January 2017:

*  Further generalization of `zwave_network.py`;
*  Better formalization of returned status string when interacting with `sens_zwave.py`;
*  Remove client id/key and email field of `config/bd_setting.json` for the purpose of security;

## Configurations of Device Parameters:

Look-up tables for parameter settings in `zwave.json` can be referred to [3].

## Features [updated on 16 January 2017]

The zwave connector module has been generalized for most of zwave related device.

The compatible value/sensor points includes:

*  various sensor data, e.g. temperatures, humidity etc.;
*  thermostats;
*  GRB bulbs;
*  dimmers;
*  node battery levels;
*  node power levels;
*  switches, e.g. sending control commands to smart plugs;


## General Phylosophy

The main challenge that the zwave device connector face is the slow speed for establish of zwave network. Typically the creation of zwave network will take roughly 10 ~ 20 seconds. Therefore the sampling frequency of device connector can be degenerated due to this fact. 

For solving this issue, a instance of zwave network is created by `zwave_network.py` program, which is idealy a long running program and should be considered as a background engine (black box) that is not supposed to be exposed to users. Within the contextual of zwave network, various operations such as reading data from nodes and sending command to device switch can be performed using `sens_zwave.py` module, which can be cosidered as a interface for external program to communicate with zwave network. The communication between `zwave_network.py` and `sens_zwave.py` is performed via TCP/IP socket. However only localhost programs are allowed to be connected to zwave network. This sofware stack configurations can be demostrated in figure 1.

In short, external programs are supposed to call `sens_zwave.py` for performing various operations, i.e. there are no need to call `zwave_network.py` directly.

![configurations](/img/config.png)

## Program Usage

Generally speaking, 5 options can be used to call `sens_zwave.py` module:

### Start Zwave Network

As is demonstrated in previous section, the operations of `sens_zwave.py` is performed within the contextual of `zwave_network.py`. Hence the first thing to do before performing various sensing/actuatng operations is to establish zwave network, which must be done manually using following command (as the initializing of zwave network is performed under sodo operation, the input of root password may be required):
```
$ python sens_zwave.py -s
```

### Quit Network

The zwave network will not be terminated automatically, since the creation of zwave network is a time consuming task, which can degenerate the system sampling frequency. Hence it need be terminated manually using following command in the final steps:
```
$ python sens_zwave.py -q
```

### Usage Information

The usage information of program can be retrieved using:
```
$ python sens_zwave.py
```
or
```
$ python sens_zwave.py -u
``` 

### Publishing Sensed Value to BuildingDepot

To publish the data of all sensor points on all network nodes to BuildingDepot stack, using following command:
```
$ python sens_zwave.py -r -1
```

Additionally, to publish all sensor points on a specific nodes (specified by node_id) to BuildingDepot stack, using following command:
```
$ python sens_zwave.py -r node_id
``` 

Note: the node id must be a positive integer larger than 2 and if there are no responses, two facts can be the cause:
* The node is not connected into network (perhaps due to accidently power failure or initial zwave product matching);
* The sensed value is not correctly specified. This can refer to the `listen` item in `zwave.json`;

### Sending Command to Switch on Zwave Device

To control zwave device, using following command:
```
$ python sens_zwave.py -w node_id value_label command
``` 

*  node_id is a positive integer larger than 2 specifying the id of the node;
*  value_label indicates the name of particular switch;
*  command is the switch command which includes on/off/toggle;

For example, using following command can activate the Switch1 of node 3:
```
$ python sens_zwave.py -w 3 Switch1 on
```

## Files

The main code files in this repo includes:

*  `zwave/sens_zwave.py`: the device connector program for interfacing with external modules;
*  `zwave/zwave_network.py`: the background zwave network engine for supporting various zwave oprations;
*  `zwave/check.py`: a debugging tool used to do final check on zwave network configurations and device pairing;
*  `zwave/post_bd.py`: a modified version of original `bd_connect.py` module;
*  `zwave/find_port.sh`: run this script to check the file path of USB port of zwave hub controller (zwave stick);
*  `config/zwave.json`: configuration file which need be put in `/Connectors/config/` directory;
*  `openzwave/`: the library repo copied from `python-openzwave`;

## System Configurations

### Library

The `python-openzwave` library is needed for this program, make sure to install it according to write-up available at <a href="https://github.com/OpenZWave/python-openzwave"> here </a> [1];

### Add Python Search Path to Sudoers

As the program is needed to run using super user permission. A copy of python search path need be added to `/etc/sudoers` config file.

Specifically, add ` env_keep += "PYTHONPATH" `to ` /etc/sudoers `;

### File Directories

*  Put `zwave.json` in the directory of ` /Connectors/config/ `;
*  Make a copy of `/python-openzwave/openzwave` to `/Connectors/openzwave/`;

## json file configurations

The detail of `/config/zwave.json` file can be demostrated as follows:

*  device: the device path of zwave USB stick, which can be checked by `sudo bash find_port.sh`. Typically, it should be `/dev/ttyACM0` or `/dev/ttyACM1`;
*  log: the level of logging information, the option can be `None`, `Info`, `Debug`;
*  log_file: the file name (and path) for saving program runtime logs;
*  write_log_file: whether the logging information need be saved to a file;
*  write_console: whether the logging information need be printed on console;
*  port: the socket port by which `zwave_network.py` is used to communicate with `sens_zwave.py`, which can be arbitary non-well known available port;
*  MAX_THREAD: the max number of thread that the instance of `ZwaveNetwork` of `zwave_network.py` can handle;
*  mapping: the user defined mapping between node id and node name. In particular, the node id is essentially the sequence of device during zwave device pairing process which should be a positive integer larger or equal to 2. The node id of 1 is specially reserved for zwave controller hub/Zwave USB stick;
*  config: parameter configurations. This is various within different device which normally can look up from manufacturer's user guide. For example the parameter of 111 of AeotecMultisensor6 indicate the sampling period of the device, i.e the time interval for device the update and sending data. The value of this parameter is in the units of seconds;
* listen: specify the the data of which sensor points of each nodes need be collected (and published to BuildingDepot stack). For example, following configuration indicates only Ultraviolet and Temperature are needed for node 2 with remaining values being discarded;
```
"2": [Ultraviolet, Temperature] 
```

## Getting Started

*  After pairing the zwave stick and zwave node device, plug zwave stick into the usb port of host. This step enable zwave stick to create a zwave mesh network;
*  Run `$ sudo bash find_port.sh` to find the file path of zwave stick, which is normally in the format of `/dev/ttyACM<x>;`;
*  Modify the value of `device` in `/Connectors/config/zwave.json` according to the value found in previous step;
*  Check the `check` item in `zwave.json` defines the correct nodes and sensor points;
*  Run `$ sudo python check.py` to make sure the zwave network is fully functioned;
*  Start network by running `$ python sens_zwave.py -s`;
*  Various operations of zwave device can now be performed using `sens_zwave.py`;
*  If there are nodes being added or removed, the zwave network need be restart for reconfigurations. This has to be done manually using following commands:
   ```
   $ python sens_zwave.py -q      # stop network service
   $ python sens_zwave.py -s      # start network service
   ``` 

## Terminologies

### Essential Concepts

*  node: sensor node in zwave network;
*  sensor points/values: each node may have multiple sensor points/values, for example a AeotecMultisensor6 node will have 6 values/sensor points;
*  node id: each node will be assigned a node id (a integer larger than 1) according to the sequence of device pairing. The node with id 1 is the zwave controller hub/zwave stick;
*  home id: each zwave network will have their own home id. The home ids of all nodes in same zwave network are identical;
*  mac id: a concatenation of manufacturer id, product type id and product id, which is used to uniquely identify a hardware. In particular, all sensor points/values in same node shares same mac id;
*  source identifier: a string in the format of `home_id:node_id[node_name]:value`, which is used to identify each sensor source/sensor points;

### Tags List:

*  node_name: the name of node;
*  quantity: the value of the node, such as temperature;
*  home_id: number used to identify a zwave network;
*  node_id: id of a node;
*  units: the units of sensed data;
*  mac_id: hardware id used to identify each sensor points (used as identity);

## Further Suggestions on BuildingDepot Stack

### Page Redirection After Delete Operation

It is noticed that when perform delete operation on sensor in central service, for example in page 8, the url query string will be cleared causing web page being redirected to first page, i.e.
```
Before Deletion: https://bd-exp.andrew.cmu.edu:81/central/sensor?page=8
```
```
After Deletion: https://bd-exp.andrew.cmu.edu:81/central/sensor
```

A better way for dealing with this may be remembering the query string parameter (`page = 8`), and redirect to current page after deletion operation.

### Export Data in Data Service

As is demonstrated in [2], one of target audience of BuildingDepot is analysis researchers who are interested in analyzing building data but may be proficient in writing application program. In many case, external sortwares, such as Matlab, are needed for performing advanced data analysis. Although currently data can be read via the RESTful API call of reading time series datapoints. However to export as a file, another connector program may need for implementing this specific task. Hence, as a very essential task, it is suggested to increase a data export feature near the graphic view of each sensor points of Data Service, for example, export time series data as a csv file format for further analysis.

### Online Sensor

In many cases, it is very often to compare time series data with a reference value, such as comparing indoor temperature with average outside temperature. Obviously, the data of indoor temperature can be easily acquired using temperature sensor and existing softare module. However, for outsite temperature, the data can be acquired easily from several official website such as [weather.com](https://weather.com/). Hence creating a online reference temperature sensor by gathering data from these web service can be a possible way to advance the project (in most of cases, these websites provide some useful RESTful API for suppporting software development). In short, in this case the type of sensor supported by BuildingDepot includes:

*  Physical Sensor: achieved by physical sensor device;
*  Virtual Sensor: virtualized sensor, achieved by analyzing data from physical sensor;
*  Online sensor: sensor data gathered from common web service, e.g. average temperature got from [weather.com](https://weather.com/);

## Further Suggestions on Connector Modules:

In current implementations, the connector will do following task in sequence:

*  Called by client app;
*  Read one set of data and post to buildingDepot stack or send command to switch;
*  Exit from connector and wait the next call of client app;

However since the initialization of zwave network is very slow, it is recommended to send/receive data in the contexul of instance of Network (this means idealy the zwave network is a long running program).

Another problem is that using above mothod the sampling period is controlled by (external) client app. However, the fact is that every zwave device have their own frequencies for updating value, for example, the sampling period of multisensor6 can be configured by parameter 111. This may cause problems on validity of data acquired. For instance, if the sampling period of updating value is 10 seconds, while the sampling period for request is 2 seconds, there will have roughly 4/5 data being useless and hence wasting database resources.

A effective method is to only allow zwave sensor dertermine the sampling period, while external client app only makes decisions on whether or not continue data acquisition. The specific process can be summarized as follows:

*  External app calls the connector;
*  The connector listen to request from various zwave nodes. Once there are updates in any nodes, the connector will 'wake up' and post data to BuildingDepot;
*  Connector will not stop unless the apps make a 'stop' call;

An example 'sniffing' program can be referred to `/zwave/sens_zwave.py` module;

## References

[1] python-openzwave library, [Online] Available at: <[https://github.com/OpenZWave/python-openzwave](https://github.com/OpenZWave/python-openzwave)> [Accessed on January 5, 2017]

[2] T.Weng, A.Nwokafor, Y.Agarwal, "BuildingDepot 2.0: An Integrated Management System for Building Analysis and Control", Buildsys' 13, November 2013.

[3] Library of Zwave Certified Device, [Online], Available at: <http://products.z-wavealliance.org/> [Accessed on January 18, 2017]

<hr/>
<i> Updated on Tuesday 17 January, 2017 </i>


