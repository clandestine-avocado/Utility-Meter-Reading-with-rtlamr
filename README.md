### Purpose


RTL = 

AMR = **A**utomated **M**eter **R**eading

Utilities often use "smart meters" to optimize their residential meter reading infrastructure. Smart meters transmit consumption information in the various industrial, scientific and medical ([ISM](https://en.wikipedia.org/wiki/ISM_radio_band)) bands allowing utilities to simply send readers driving through neighborhoods to collect commodity consumption information. One protocol in particular: Encoder Receiver Transmitter (ERT) by Itron is fairly straight forward to decode and operates in the 900MHz ISM band, well within the tunable range of inexpensive rtl-sdr dongles.

This project is a software defined radio receiver for these messages. We make use of an inexpensive rtl-sdr dongle to allow users to non-invasively record and analyze the commodity consumption of their household.

There's now experimental support for data collection and aggregation with [rtlamr-collect](https://github.com/bemasher/rtlamr-collect)!

[![Build Status](https://travis-ci.org/bemasher/rtlamr.svg?branch=master&style=flat)](https://travis-ci.org/bemasher/rtlamr)
[![AGPLv3 License](https://img.shields.io/badge/license-AGPLv3-blue.svg?style=flat)](http://choosealicense.com/licenses/agpl-3.0/)

### Requirements
//
Second time around, I used Snap to get all requirements, and install.
https://snapcraft.io/install/rtlsdr/raspbian
On a Raspberry Pi running the latest version of Raspbian snap can be installed directly from the command line:
sudo apt update
sudo apt install snapd

You will also need to reboot your device:
```
sudo reboot
```
After this, install the core snap in order to get the latest snapd:
```
sudo snap install core
```
Install rtlsdr, use the following command:
```
sudo snap install rtlsdr
```

//
- GoLang >=1.3 (Go build environment setup guide: http://golang.org/doc/code.html)
- rtl-sdr
  - Windows: [pre-built binaries](https://ftp.osmocom.org/binaries/windows/rtl-sdr/)
  - Linux: [source and build instructions](http://sdr.osmocom.org/trac/wiki/rtl-sdr)

### Building

This project requires the package [`github.com/bemasher/rtltcp`](http://godoc.org/github.com/bemasher/rtltcp), which provides a means of controlling and sampling from rtl-sdr dongles via the `rtl_tcp` tool. This package will be automatically downloaded and installed when getting rtlamr. The following command should be all that is required to install rtlamr.

    go get github.com/bemasher/rtlamr

This will produce the binary `$GOPATH/bin/rtlamr`. For convenience it's common to add `$GOPATH/bin` to the path.

### Usage

See the wiki page [Configuration](https://github.com/bemasher/rtlamr/wiki/Configuration) for details on configuring rtlamr.

Running the receiver is as simple as starting an `rtl_tcp` instance and then starting the receiver:

```bash
# Terminal A
$ rtl_tcp

# Terminal B
$ rtlamr
```



### Run Server Remotely
If you want to run the spectrum server on a different machine than the receiver you'll want to specify an address to listen on that is accessible from the machine `rtlamr` will run on with the `-a` option for `rtl_tcp` with an address accessible by the system running the receiver.

### Message Types

The following message types are supported by rtlamr:

- **scm**: Standard Consumption Message. Simple packet that reports total consumption.
- **scm+**: Similar to SCM, allows greater precision and longer meter ID's.
- **idm**: Interval Data Message. Provides differential consumption data for previous 47 intervals at 5 minutes per interval.
- **netidm**: Similar to IDM, except net meters (type 8) have different internal packet structure, number of intervals and precision. Also reports total power production.
- **r900**: Message type used by Neptune R900 transmitters, provides total consumption and leak flags.
- **r900bcd**: Some Neptune R900 meters report consumption as a binary-coded digits.

### Sensitivity

Using a [Nooelec RTL-SDR with RTL2832U & R820T](https://www.amazon.com/gp/product/B008S7AVTC/ref=ppx_yo_dt_b_asin_title_o09_s00?ie=UTF8&psc=1) with the provided antenna, I can reliably receive standard consumption messages from ~300 different meters and intermittently from another ~600 meters. These figures are calculated from the number of messages received during a 25 minute window. Reliably in this case means receiving at least 10 of the expected 12 messages and intermittently means 3-9 messages.

### Compatibility

Currently the only tested meter is the Itron C1SR. However, the protocol is designed to be useful for several different commodities and should be capable of receiving messages from any ERT capable smart meter.

Check out the table of meters I've been compiling from various internet sources: [ERT Compatible Meters](https://github.com/bemasher/rtlamr/blob/master/meters.md)

If you've got a meter not on the list that you've successfully received messages from, you can submit this info via a form available at the link above.



---

### My Meters:

|Utility|Model|Type#|FCC_ID|rtl-amrID|FreqRange_mHz|Links|
|-------|-------|-----|-------|-------|-------|-------|
|Gas|Itron 100GDLS|12|EWQ100GDLAS|76356921|903.0-926.85|[FCC Link 1:](https://apps.fcc.gov/oetcf/tcb/reports/Tcb731GrantForm.cfm?mode=COPY&RequestTimeout=500&tcb_code=&application_id=DnaFgjHGnUo76ite3zO8MA%3D%3D&fcc_id=EWQ100GDLBS) and [FCC Link 2](https://fccid.io/EWQ100GDLAS)|
|Electric|41ER-1|4|EO941ER-1|50056711|910.0-920.0|[FCC Link](https://apps.fcc.gov/oetcf/eas/reports/Eas731GrantForm.cfm?mode=COPY&RequestTimeout=500&application_id=yWEjtEDMQV6HDbrnO7QtuQ%3D%3D&fcc_id=EO941ER-1)|

---
Electric Meter ID above never reports; surveyed via rtlamr2mqtt and these are the results. ID 34183618 appears to me the most frequent SCM reporting meter, but this does not mean it is my meter.
![Meter Image](assets/emeters.jpg "Utility Meter Image")




### Main Folders for reference
```
/home/pi/rtl-sdr
/home/pi/go
```
---


### Command Line testing - log to txt file
Once you get the SDR dongle up and running, you can get at data from all the meters your SDR can receive, depending on your antanae set up. Then you need to look for your meter ID within the data.

First, you'll want to print the data directly to terminal to test if you are actually receiving data:
```/home/pi/go/bin/rtlamr``` 

Once you've confirmed data is being received, you will want to print the data to a text file as a CSV so you can sort and filter in Excel. There, you can look for your meter ID:
```/home/pi/go/bin/rtlamr -format=csv >> /home/pi/rtl-sdr/meter_dump.txt```

Once you find your meter ID, you can add a filter to get only your meters:
|meter|command|
|-----|-----|
|Elec|```/home/pi/go/bin/rtlamr -filterid=50056711 -msgtype=scm -format=csv >> /home/pi/rtl-sdr/ELEC.txt```|
|Gas|```/home/pi/go/bin/rtlamr -filterid=76356921 -msgtype=scm+ -format=csv >> /home/pi/rtl-sdr/GAS.txt```|


Examples:

```
/home/pi/go/bin/rtlamr -filterid=76356921,50056711 -msgtype=scm,scm+
```



Other Useful rtl-amr options:

|rtlamr flag options|Description|
|-----|-----|
|-duration=0s|time to run for, 0 for infinite, ex. 1h5m10s|
|-filterid=|display only messages matching an id in a comma-separated list of ids|
|-filtertype=|display only messages matching a type in a comma-separated list of types|
|-format=plain|decoded message output format: plain, csv, json, or xml|
|-msgtype=|comma-separated list of message types to receive: all, scm, scm+, idm, netidm, r900 and r900bcd|
|-samplefile=NUL|raw signal dump file|
|-single=|true/false; one shot execution, if used with -filterid, will wait for exactly one packet from each meter id|
|-symbollength=|symbol length in samples (8, 32, 40, 48, 56, 64, 72, 80, 88, 96)|
|-unique=|true/false: suppress duplicate messages from each meter|
|-version=|true/false: display build date and commit hash|

---

### Python to Collect and Send Data
Now that you have identified your meter(s) it is time to create a python script to receive the gas meter transmission and send the consumption data/reading over [MQTT](https://en.wikipedia.org/wiki/MQTT). The python script itself is located here: ```/home/pi/rtl-sdr/gas_rtlamr2mqtt.py```


``` 
import subprocess
import paho.mqtt.client as mqtt
import time
import json

client = mqtt.Client("SDR Meter Reader")
client.username_pw_set(username="XXX",password="XXX")
client.connect("192.168.1.237", 1883, 60)
client.loop_start()


try:
  while True:
      completed = subprocess.run(['/home/pi/go/bin/rtlamr','-filterid=76356921','-msgtype=scm+','-single=true', '-format=json', '-duration=10m'], stdout=subprocess.PIPE, stderr=subprocess.DEVNULL)
      try:
         data=json.loads(completed.stdout.decode("utf-8"))
      except ValueError:
         print("Error")
      else:
         reading = data['Message']['Consumption']
         client.publish("home/gasmeter",reading,0,True);
         print("Reading:",reading)
except KeyboardInterrupt:
  print("interrupted!")
  client.loop_stop()
  client.disconnect()

```

---
### Create a Service for the Script
Now we're going to define a [service](https://www.hostinger.com/tutorials/manage-and-list-services-in-linux/) to run this script. The service definition (defined in the ```XXX.service file```) must be in the ```/lib/systemd/system``` folder. Our service is going to be called ```gasmeter2mqtt.service```. When finished, if for any reason the script gets aborted (power outage, reboot of system, etc), the service will be restarted automatically and start running the python script again, resuming your data collection.

Navigate to the system folder:
```
cd /lib/systemd/system/
```

Create the service file:
```
sudo nano gasmeter2mqtt.service
```

Populate the service file:
```
[Unit]
Description=RTLAMR Software Defined Radio intercept of gas meter via python script
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/python /home/pi/rtl-sdr/gas_rtlamr2mqtt.py
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

Adjust permissions for the service and the script:
```
sudo chmod 644 /lib/systemd/system/gasmeter2mqtt.service
chmod +x /home/pi/hello_world.py
```

Reload services for changes to take effect:

```
sudo systemctl daemon-reload
```

Enable the service:

```
sudo systemctl enable gasmeter2mqtt.service 
OR 
sudo systemctl enable elecmeter2mqtt.service 
OR 
sudo systemctl enable meters2mqtt.service

```

Start the service:

```
sudo systemctl start gasmeter2mqtt.service 
OR 
sudo systemctl start elecmeter2mqtt.service 
OR 
sudo systemctl start meters2mqtt.service
```






-filterid=: display only messages matching an id in a comma-separated list of ids.
```
/home/pi/go/bin/rtlamr -filterid=50056711,76356921 -msgtype=scm,scm+
```

-filtertype=: display only messages matching a type in a comma-separated list of types
```
/home/pi/go/bin/rtlamr -msgtype=scm,scm+
```







Service related commands:
|Function|Command|
|-----|-----|
|Create service| See above|
|Enable service|```sudo systemctl enable gasmeter2mqtt.service```|
|Reload services|```sudo systemctl daemon-reload```|
|Disable service|```sudo systemctl disable gasmeter2mqtt.service```|
|Start service|```sudo systemctl start gasmeter2mqtt.service```|
|Check status|```sudo systemctl status gasmeter2mqtt.service```|
|Start service|```sudo systemctl start gasmeter2mqtt.service```|
|Stop service|```sudo systemctl stop gasmeter2mqtt.service```|
|Check log|```sudo journalctl -f -u gasmeter2mqtt.service```|

---


|Function|Command|
|-----|-----|
|Create service| See above|
|Enable service|```sudo systemctl enable elecmeter2mqtt.service```|
|Reload services|```sudo systemctl daemon-reload```|
|Disable service|```sudo systemctl disable elecmeter2mqtt.service```|
|Start service|```sudo systemctl start elecmeter2mqtt.service```|
|Check status|```sudo systemctl status elecmeter2mqtt.service```|
|Start service|```sudo systemctl start elecmeter2mqtt.service```|
|Stop service|```sudo systemctl stop elecmeter2mqtt.service```|
|Check log|```sudo journalctl -f -u elecmeter2mqtt.service```|

---

|Function|Command|
|-----|-----|
|Create service| See above|
|Enable service|```sudo systemctl enable meters2mqtt.service```|
|Reload services|```sudo systemctl daemon-reload```|
|Disable service|```sudo systemctl disable meters2mqtt.service```|
|Start service|```sudo systemctl start meters2mqtt.service```|
|Check status|```sudo systemctl status meters2mqtt.service```|
|Start service|```sudo systemctl start meters2mqtt.service```|
|Stop service|```sudo systemctl stop meters2mqtt.service```|
|Check log|```sudo journalctl -f -u meters2mqtt.service```|

---


