# rfAcquirer(Initial Documentation)
rfAcquirer is python based application which interacts with iDRACs and continuously  collects raw telemetry and Lclog data at an given interval. It has different services which works independently to perform the collection of the data and creation of transaction files that helps to move the data to archive. These services are:
1. TelemetryAcquirer: It collects telemetry data from iDRACs.
2. LclogAcquirer: It collects Lclog data from iDRACs. 
3. Json2Tx: It creates transaction files that helps to move data to the archive.

One can use it in a docker container. Running the application in a container is recommended because it standardizes and simplifies the installation process.

#### FAQ : 

1. How to edit the TelemetryAcquirer config files? [Ans 1](#faqAcquirerconfig)
2. How to edit the LclogAcquirer config files? [Ans 2](#faqLClogconfig)
3. How to check the health of iDRAC?** 


## Table of Contents

1. [Running in a docker container](#rfAcquirer-docker) 
    1. [Getting rfAcquirer docker image](#get-rfAcquirer-docker) 
    2. [Running rfAcquirer in a container](#run-rfAcquirer-docker)
    3. [Stopping and removing the container](#stop-rfAcquirer-docker)
2. [Building a new version for the docker containers](#build-rfAcquirer-docker)
3. [rfAcquirer usage](#rfAcquirer-usage)
    1. [Executing rfAcquirer services](#rfAcquirer-execution)
	    1. [TelemetryAcquirer](#Acquirer)
	    2. [LclogAcquirer](#LClog)
	    3. [Json2Tx](#Json2Tx)
    2. [Config File Example](#config-file)
	    1. [TelemetryAcquirer](#Acquirer_config)
	    2. [LclogAcquirer](#LClog_config)
	    3. [Json2Tx](#Json2Tx_config)

## 1. Running rfAcquirer in a docker container (recommended) <a id="rfAcquirer-docker"></a>


### 1.1. Getting rfAcquirer docker image <a id="get-rfAcquirer-docker"></a>
```
$ docker pull mliptest/rfacquire:latest
```

### 1.2. Running rfAcquirer in a container <a id="run-rfAcquirer-docker"></a>
```
$ docker run -dit --name acquirer_v1 --net=host -v $(pwd)/config:/home/rfAcquirer/config -v $(pwd)/data:/home/rfAcquirer/data -v $(pwd)/logs:/home/rfAcquirer/logs  --entrypoint bash mliptest/rfacquire:latest
```
- $(pwd)/config:/home/rfAcquirer/config 
	- Mount the project root directory at the given docker path to share config.json with host machine.
- $(pwd)/data:/home/rfAcquirer/data
	- Mount the project root directory at the given docker path to share data with host machine.
- $(pwd)/logs:/home/rfAcquirer/logs
	- Mount the project root directory at the given docker path to share logs with host machine.


### 1.3. Stopping and removing the container <a id="stop-rflclog-docker"></a>
```
$ docker stop acquirer_v1  && docker rm acquirer_v1 
```

## 2. Building a new version for the docker containers <a id="build-rfAcquirer-docker"></a>

Build the application image with rfAcquirer installed. Replace  `repository` and `<version>` with the repository and new version to build:
```
$ git clone https://github.com/JonHass/Kaleidoscope.git
$ cd rfAcquirer
$ docker build -f Dockerfile -t <repository>:<version>
```

Publish the newly created images to dockerhub. Replace `repository`  and  `<version>` with the repository and new version to publish:
```
$ docker push <repository>:<version>
```

## 3. rfAcquirer usage <a id="rfAcquirer-usage"></a>


### 3.1. Executing rfAcquirer services <a id="rfAcquirer-execution"></a>
- rfAcquirer has three different services working independently to collect data and store it into archive.

**Note:**  It is recommended  to execute the commands in daemon mode.
#### 3.1.1. TelemetryAcquirer <a id="Acquirer"></a>
-- To start the collection of telemetry data from the iDRACs one needs to start telemetryAcquirer. Below is the execution command of telemetryAcquirer:
```
$ docker exec -d acquirer_v1 bash startServices.sh -i I101 -r /home/rfAcquirer/config/configTelemetryAcquirer_I101.cfg -s telemetryAcquirer
```
- -i I101:
	- Instance ID 
- -r /home/rfAcquirer/config/configTelemetryAcquirer_I101.cfg:
	- Docker path of the config file.
- -s telemetryAcquirer
	- Service name.

-- To stop the collection of telemetry data from the iDRACs one needs to stop telemetryAcquirer. Below is the termination command of telemetryAcquirer:
```
$ docker exec -d acquirer_v1 bash stopServices.sh -i I102 -s telemetryAcquirer
```
- -i I101:
	- Instance ID 
- -s telemetryAcquirer
	- Service name.

#### 3.1.2. LclogAcquirer<a id="LClog"></a>
-- To start the collection of LClog data from the iDRACs one needs to start lclogAcquirer. Below is the execution command of lclogAcquirer:
```
$ docker exec -d acquirer_v1 bash startServices.sh -r config/configLclogAcquire.cfg -s lclogAcquirer
```
- -r config/configLclogAcquire.cfg:
	- Docker path of config file.
- -s lclogAcquirer:
	- Service name.

-- To stop the collection of LClog data from the iDRACs one needs to stop lclogAcquirer. Below is the termination command of lclogAcquirer:

```
$ docker exec -d acquirer_v1 bash stopServices.sh -s lclogAcquirer
```
- -s lclogAcquirer:
	- Service name.

#### 3.1.3. Json2Tx<a id="Json2Tx"></a>
-- To create transaction file which is used to transfer data from host machine to archive one needs to start json2tx service. To start the service execute below command:
```
$ docker exec -d acquirer_v1 bash startServices.sh -s json2tx
```
- -s json2tx:
	- Service name.

-- To stop the creation of transaction file one needs to stop json2tx service. To stop the service execute below command:
```
$ docker exec -d acquirer_v1 bash stopServices.sh -s json2tx
```
- -s json2tx:
	- Service name.

### 3.2. Config File Example <a id="config-file"></a>

-   The default config can be customized for all three services. Each service has a separate config file with different formats and all the fields are mandatory. Example of default config files are as below: 

#### 3.2.1. TelemetryAcquirer <a id="Acquirer_config"></a>
```
{
    "batch_size":2,
    "file_collection_time":"3600",
    "iDRACS": [

        "https://root:calvin@192.168.14.212",
        "https://root:calvin@192.168.13.202",
        "https://root:calvin@192.168.15.17",
        "https://root:calvin@192.168.7.58"

    ],
    "ports_list":[
        8109,
        8055

    ],
    "SOURCE_IP":"192.168.12.228",
    "onlySubscription": "False",
    "file_collection_path":"/home/rfAcquirer/data/telemetry",
    "log_file_path": "/home/rfAcquirer/logs",
    "log_file_size":"5000000"
}
```
- The `batch_size` is number of iDRACs to be communicated in a single iteration.
- The `file_collection_time`  is the interval for which raw json file is created. Interval must be in seconds.
- The `iDRACS` is the list of URLs of the iDRACs with username and password.
- The `ports_list` is the list of ports to interact with the iDRACs.
- The  `SOURCE_IP` is the HOST IP address.
- When `onlySubscription` is **True** it will just subscribe the iDRACs with the ports or if it is **False** then it will subscribe as well as collect telemetry data.
- The `file_collection_path` is the data directory where data will be stored in the structured format.
- The `log_file_path` is the docker directory path for the logs.
- The `log_file_size` is the maximum file size of log files. Size is Bytes.

#### 3.2.2. LclogAcquirer<a id="LClog_config"></a>
```
{
    "job": {
      "interval": 1,
      "lclog_output_dir": "/app/output",
      "lclog_checkpoint_dir": "/app/checkpoint",
      "lclog_complete_dir": "/app/complete",
      "log_file_path": "/app/logs",
      "log_file_size":"5000000",
      "log_level": "DEBUG"
    },
    "redfish": {
      "servers": [
        "https://root:calvin@192.168.13.108:443",
        "https://root:calvin@192.168.7.54:443",
        "https://root:calvin@192.168.13.202:443",
        "https://root:calvin@192.168.15.17:443",
        "https://root:calvin@192.168.7.58:443",
        "https://root:calvin@192.168.14.212:443"
      ]
    }
}
```
- The section `job` will define defines parameters of the operation.
	- The `job/interval`  is time for which program will wait to again get lclog from same iDRAC. The interval is in minutes.
	- The `job/lclog_output_dir`  is the directory path where lclog data file will be created.
	- The `job/lclog_checkpoint_dir` is the path where 0 bytes file will be created for checking the last lclog id received.
	- The `job/lclog_complete_dir` is the directory path where completed files will be stored.
	- The `job/log_file_path`is the directory path where logs for this service will be stored.
	- The `job/log_file_size` is the maximum file size of log files. Size is Bytes.
	- The `job/log_level` defines the logging level. It can be INFO, ERROR, DEBUG or WARNING.
- The section `redfish` 	defines the property of iDRACs.
	- The `redfish/servers` is the list of URLs of the iDRACs with username and password.

#### 3.2.3. Json2Tx<a id="Json2Tx_config"></a>
```
{
    "TelemetryDataPath": "/home/rfAcquirer/data/telemetry",
    "acfRDpath": "/home/rfAcquirer/data/RD",
    "LclogDataPath":"/home/rfAcquirer/data/lclog",
    "SourceHost": "192.168.12.228",
    "TargetHost": "192.168.7.127",
    "FileOperations": "mv",
    "ArchivePath": "/home/archive/Dell/Kaleidoscope",
    "log_file_path":"/home/rfAcquirer/logs",
    "log_file_size":"5000000"
}
```

The `TelemetryDataPath` is the docker path of the telemetry data directory.
The `acfRDpath`  is the docker path where the transaction file to be created.
The `LclogDataPath` is the docker path of the LClog data directory.
The `SourceHost` is the HOST IP address.
The `TargetHost` is the archive IP address.
The `FileOperations` is operation to be performed between source and target. It can be either **mv**(move) or **cp**(copy).
The `ArchivePath` is the path where the data will be stored in the archive.
The `log_file_path` is the docker path of the log file.
The `log_file_size` is the maximum file size of the log files.


## 6. FAQ Answers <a id="faq"></a>


**Ans 1:**<a id="faqAcquirerconfig"></a>
To open the config file navigate to the mounted config directory by following the command as below:

```
# Replace <mounted_config_director> with the project root  config directory path.
$ cd <mounted_config_director>  
$ vim configTelemetryAcquirer_I101.cfg

```

**Note:**

-   To create a new config file, copy the default file and change the suffix of the name of the file(eg. I101 in the name to I102).  
-   One can change the values or create as many config file as they need with different instance Id and suffix to simultaneously run them.

**Ans 2:**<a id="faqLClogconfig"></a>
To open the config file navigate to the mounted config directory by following the command as below:

```
# Replace <mounted_config_director> with the project root config directory path.
$ cd <mounted_config_director>  
$ vim configLclogAcquire.cfg

```



