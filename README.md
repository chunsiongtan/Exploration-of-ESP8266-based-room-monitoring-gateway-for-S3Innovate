## "Exploration of ESP8266 based room monitoring gateway for S3Innovate" ##


In this exploration session, S3 Innovate and Microsoft evaluates the feasibility of utilizing ESP8266 Microcontroller based IOT Device Gateway for remote monitoring of meeting rooms.

Core team:

- Teo Say Leng - Founder, S3 Innovate
- Nicholas Soon - Director, S3 Innovate
- Hafiz � Lead Hardware Engineer, S3 Innovate
- Tan Chun Siong ([@tanchunsiong](https://twitter.com/tanchunsiong)) � Senior Technical Evangelist, Microsoft
 
The ESP8266 is a Microcontroller Unit (MCU) manufactured by Espressif Systems with built-in Wi-Fi capabilities. Relatively low-cost with respect to computational capabilities, the ESP8266 MCU allows developers to build the device firmware using numerous SDKs and languages, notably the Arduino IDE.
In this session, we have setup an IOT Architecture loosely based off the [Azure IOT Reference Architecture](http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf). 

![IoT Architecture Diagram](/images/s3innovate/remote-monitoring-arch.png)



## Customer profile ##


S3 Innovate, a recent graduate of Microsoft�s BizSpark Plus program, is a Singapore based startup with core competency around collecting and connecting building data from sensors to the cloud. S3 Innovate offers building owners an integrated view of their building health and operational efficiency, which helps them optimize the power consumption performance of buildings for greater environmental sustainability.

As a leading provider of green solutions in the market, S3 Innovate recently implemented a pilot solution for the Building and Construction Authority (BCA), connecting the chillers of their commercial buildings to the Microsoft cloud with the aim of maintaining chiller efficiency and preventing unnecessary wastage of energy. When fully deployed across the island, this solution could help organizations reduce their energy consumption significantly, translating into tangible cost savings for organizations.

http://www.s3innovate.com/
Woodlands 11, #03-13, Singapore 737853
contact@s3innovate.com

 
## Problem statement ##

The granularity of S3 Innovate's current sensor network, is implemented at the meeting room level. In each room, the data collected includes telemetry such as Motion, Temperature, Humidity, Light Luminosity, Carbon Monoxide, Noise Level, and Volatile Organic Compounds. These parameters build up the fundamental blocks of data to evaluate the comfort and occupancy/utilization of the meeting rooms. 

Over time as customer base and number of gateway devices grow, they are looking for ways to scale their ingestion pipeline to cater to these growths. Keeping capital cost low is the first area to address as telemetry collecting devices is an upfront investment cost. Optimization of storage cost is the second area to address.

While exploring new devices, services and infrastructure strategy, S3 Innovate would like to future prove their system design, ensuring that the transition to data analytics projects in near future would be as seamless as possible.

Core requirement for device gateway

1. Ability to harden the system, 
2. Reduce the maintenance needs, and
3. Resilient to data corruption

Core requirement for historical telemetry data storage

1. Prepared in a state which is ready for ETL by popular data analytics systems, 
2. Relatively low in cost for long term storage
3. Resilient to data corruption

Here are scope of exploration for this session between Microsoft and S3 Innovate

- Feasibility evaluation of NodeMCU ( or compatible ESP8266 based gateway devices) for telemetry collection
- Demonstrate data ingestion based on best practices, optimizing for cost effectiveness, and preparation for future data analytics activities.
- Mitigate the reliance on structured database resource utilization for historical data with Azure Blob Storage
- Provision email alarms/alerts for telemetry values exceeding specific threshold value

"Keeping operational cost low and optimized is an important aspect of a business and should be key part a business' organization strategy"

Teo Say Leng - Founder, S3 Innovate
 
## Solution and steps ##

After the deep dive technical discussion with S3 Innovate, we understand that they are currently using ARM based device running RTOS (real time operating system) to send telemetry to a Virtual Machine with REST Web Services (hosted on cloud or on-premise). The gateway device sends telemetry single-directionally to the endpoint only. We have proposed the 2 areas which Microsoft and S3 Innovate will explore during this hackfest.

1.	Augment the existing infrastructure to utilize Azure Event Hub or Azure IoT Hub for data ingestion instead of REST endpoint running on a Virtual Machine
-	Historical telemetry data will be *stored on low cost Azure Blob Storage. If necessary, there is the option for [Cool Storage](https://azure.microsoft.com/en-us/blog/introducing-azure-cool-storage/)
-   Data format will in Blob stored in either CSV, JSON or Avro. Avro is preferred.
-	Stream analytics for threshold monitoring and streaming to PowerBI Dashboard.


    *At this point of writing, there is a preview feature named [Azure Event Hub Archive](https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-archive-overview) which helps you to save telemetry directly into Blob storage in Avro format.
    
    The data ingestion will be utilizing Azure Event Hub, and Stream Analytics will be used for threshold monitoring and streaming to PowerBI. Historical data will be stored in cost effective Azure Blob storage in Hadoop Compatible format, and partitioned by year, month and date to ensure compatibility with Map Reduce parallel processing. We will be using a simple worker role to send triggers based on threshold to stakeholder via SendGrid email service.


2.	Choose to implement a version 2.0 of their hardware from the list below (non-exhaustive classification).
-	Linux / Windows based, such as Avantech Devices, Intel Edison, Raspberry Pi,  etc...
-	Microcontroller based device with Wi-Fi capabilities such as NodeMCU, Arduino Yun, Arduino MKR1000, Arduino Zero, Adafruit Feather, etc...


    For hardware device, the decision leans towards Microcontroller based device with Wi-Fi capabilities, as it is core operation efficiency to reduce the cost.

If you are exploring your set of devices, do take a look a [Azure Certified for IoT device catalog - Preview
](https://catalog.azureiotsuite.com/) for the compatibility list with Azure IoT Hub / Azure Event Hub 

## Architecture diagrams ##

 ![IoT Architecture Diagram](/images/s3innovate/achitecturediagram.png)

As S3 Innovate has existing services and custom dashboard incorporated in their current implementation, the above architecture is based on additional service which could be easily linked to their existing infrastructure. The key components and requirement are to demonstrate the capability 

- to send telemetry to Azure,
- prepare data for future data analytics,
- and sent out alert for threshold monitoring.

The NodeMCU (ESP8266) device (on the left) has been programmed to send of telemetry onto Azure Event Hub. For Azure Event Hub, it supports AMQP or HTTPS protocol for ingestion. Before sending, a signature needs to be generated in the HTTP POST header.  The device requires current date time from a UDP Time Server, SHA256 and Base64 string conversation to sign the authentication key before telemetry is sent up to Azure Event Hub. This is likewise the case if you would like to utilize Azure IoT Hub.

When first ingested into Azure Event Hub, the Azure Event Hub Archive service will save the historical data into Azure Blob storage at a fixed Time Interval or File Size threshold. These files are saved on Azure Blob Storage as Avro format, and the folder structure are partitioned by /YYYY /MM /DD /HH /MIN. Such partition storage strategy allows services such as HDInsight (Microsoft's Hadoop Implementation) to easily load and perform Map-Reduce functionalities.

If you like to future reduce your cost or historical or archiving of messages, the icons in white illustrates an additional data workflow. The stream analytics would first average out telemetry values for a specific unit of time (i.e. 60 seconds). Assuming telemetry send from the device gateway every second, the averaging of data would help to save diskspace up to a factor of 60, but at the cost of lossy granularity of data.

There are 2 Stream Analytic Services utilized in this architecture. The first (lower) is purely to stream data into PowerBI Dashboard, and the second is to filter out telemetry which exceeds specific threshold value, and output these records into an Event Hub for "Alerts". A [Web Role](https://azure.microsoft.com/en-us/resources/samples/event-hubs-dotnet-user-notifications/) will monitor this "Alerts" Event Hub for messages and sent email via SendGrid to the respective stakeholders.

If you are wondering when to use Event Hub vs IoT Hub, refer to [comparison of Event Hub and IoT Hub documentation](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-compare-event-hubs)

![Wiring](/images/s3innovate/nodemcu.jpg)

The sensors utilized by S3 Innovate is currently powered by 5V onboard power supply. In comparison, the NodeMCU provides both 5V (Vin pin) and 3.3V power supply, but only accepts only 3.3v input, hence care needs to be taking when wiring by either using a [logic level convertor](https://learn.sparkfun.com/tutorials/using-the-logic-level-converter) or a [voltage divider](https://en.wikipedia.org/wiki/Voltage_divider) to ensure 5v is safety stepped down to 3.3v for input reading. The diagram on the above is a sketch of the actual prototype. A photo of the actual prototype is attached below. The sensors utilized includes [CO Gas sensor](https://www.sparkfun.com/products/9403), [PIR motion Sensor](https://www.adafruit.com/product/189), [Light Dependent Resistor](https://learn.adafruit.com/photocells/using-a-photocell) and a [Temperature Probe](https://www.maximintegrated.com/en/products/analog/sensors-and-sensor-interface/DS18B20.html). 

Note that these are maker / consumer grade sensors which are used for prototyping purposes. You might want to utilize commercial grade sensors when rolling out for production purposes.

 ![Actual Device](/images/s3innovate/nodemcu-photo.jpg)

The photo above is an effort to maximise the utilization of space, soldering the devices onto a perma-board.

The plan is to manufacture their own PCB board to further maximize the space utilization, and removal of USB and FTDI ports to harden it from a physical security point of view.

The Arduino sketch/source code for the board can be found at the [code artifact section](#codeartifact) below
## Technical Delivery and Considerations##


The requirement to send HTTPS / AMQP has numerous amount of consideration. From powerful RTOS typically this does not post an issue. For Microcontroller based devices, there are considerations in terms of (and additional security consideration)

- Capability of Wi-Fi and HTTPS Protocol
- List of cipher suite supported on client (taking into consideration the support cipher suite on Azure IOT Hub and/or Event Hub), and contingency plans in case cipher suite gets updated on Azure to deprecated unsecure protocols.
- Ability to connect to WPA2 Wi-Fi Hotspot
- Ability to provision/configure Wi-Fi SSID, Password, Ingestion Endpoint, Authentication Key without re-flashing the firmware
- Interval throughput between HTTPS post messages
- Number of GPIO pins and support for existing sensors
- Ability to custom manufacture the PCB board to remove IO peripherals for hardening purposes 
- Ability to flash �final� when programming the Microcontroller


**Security details**
  
As of 6th Dec 2016, the supported cipher to POST the message tp Azure Event Hub's HTTPS REST requires the below cipher suite (ordered in ranking of strength). 

Azure does not publish the cipher suites on the documentation pages, hence do utilize [SSL Labs](https://www.ssllabs.com) to scan the Event Hub and/or Iot Hub end points. Ensure that your IoT Gateway Devices support these Cipher Suite before deployment.

1. TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 
1. TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 
1. TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA
1. TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA
1. TLS_RSA_WITH_AES_256_GCM_SHA384
1. TLS_RSA_WITH_AES_128_GCM_SHA256
1. TLS_RSA_WITH_AES_256_CBC_SHA256 
1. TLS_RSA_WITH_AES_128_CBC_SHA256 
1. TLS_RSA_WITH_AES_256_CBC_SHA 
1. TLS_RSA_WITH_AES_128_CBC_SHA
1. TLS_RSA_WITH_3DES_EDE_CBC_SHA

If you are exploring the devices which are compatible with Azure IOT Hub / Azure Event Hub, do take a look a [Azure Certified for IoT device catalog - Preview
](https://catalog.azureiotsuite.com/)

**Device used**

The NodeMCU is a Microcontroller board based on the Expressif ESP8622 Wi-Fi SoC which allows developers to build their firmware either using Lua or C++
[NodeMCU Article on Wikipedia](https://en.wikipedia.org/wiki/NodeMCU)

The sensors used in this exploration session includes

- [Temperature Sensor ds18b20](https://www.maximintegrated.com/en/products/analog/sensors-and-sensor-interface/DS18B20.html)
- [Generic PIR Motion Sensor](https://www.adafruit.com/product/189)
- [Light Dependent Resistor with 10K Ceramic Resistor](https://learn.adafruit.com/photocells/using-a-photocell)
- [Generic MQ-7 CO Sensor](https://www.sparkfun.com/products/9403) with [breakout board](https://www.sparkfun.com/products/9403). You will also need a [simple voltage divider](https://en.wikipedia.org/wiki/Voltage_divider) created by using a 1K resistor and 2.2K resistor to step down 5V signal to 3.3V signal.
- [Adafruit Full-Sized Permaboard](https://www.adafruit.com/product/590)

You might consider using a simple breadboard as an alternative to the Adafruit Permaboard when you are testing out your circuit. My choice of soldering on the perma-board is the ease of creating something relatively permanent, before [creating your CAD file for PCB manufacturing](https://oshpark.com/).

**Device messages sent**

From our test, the NodeMCU can send a HTTPS POST message at minimal interval of around 2-3 seconds. The payload is around 400-500 bytes per message. Depending on the business needs, the frequency will be reduced to 10 seconds (or more) per message.

The sample HTTPS POST Message looks like this
     
    POST https://s3innovatehackfest.servicebus.windows.net /eventhubdevices/messages HTTP/1.1
    Host: s3innovatehackfest.servicebus.windows.net 
    Authorization: SharedAccessSignature sr=https%3A%2F%2Fs3innovatehackfest.servicebus.windows.net%2Feventhubdevices%2Fmessages&sig=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx%3D&se=1737504000&skn=xxxxx 
    Content-Type: application/atom+xml;type=entry;charset=utf-8 
    Content-Length: 139


    {"Dev":"sender","Utc":"2016-11-25T06:45:04","Celsius":24.75,"Lux":886,"Motion":0,"Gas":0,"Geo":"MS SG MIC","WiFi":1,"Mem":19016,"Id":1}


**SDKs used and languages**

The language and IDE used on Device Firmware development is Adruino IDE 1.6.5 and C++ and is base out of Dave Grove's [Securely stream data from ESP8266 MCUs to Azure IoT Hub over HTTPS/REST Github](https://github.com/gloveboxes/Arduino-ESP8266-Secure-Azure-IoT-Hub-Client) Project. This project has been forked and slightly modified to utilize Azure Event Hub and the above sensors.

Refer to this guide to how to [install Arduino IDE and the support for NODEMCU](http://www.instructables.com/id/Quick-Start-to-Nodemcu-ESP8266-on-Arduino-IDE/)

For the notification of threshold, we will be using an [web role sample](https://azure.microsoft.com/en-us/resources/samples/event-hubs-dotnet-user-notifications/) written using .NET. You will need [Visual Studio](https://www.visualstudio.com/) for this project.

<a name="codeartifact"></a>
**Code artifact**

[NodeMCU Arduino Source code and Guide for sending Telemetry to Azure Event Hub](https://github.com/tanchunsiong/Arduino-ESP8266-Secure-Azure-Event-Hub-Client)

[Web Role reding from Event Hub and send alerts via SendGrid](https://azure.microsoft.com/en-us/resources/samples/event-hubs-dotnet-user-notifications/)

**Learnings from Microsoft team and S3 Innovate team**

- Trouble shooting of hardware is a useful skill during a hackfest. It is good to understand

 - [Common Soldering Problems](https://learn.adafruit.com/adafruit-guide-excellent-soldering/common-problems)
 - [Multimeter](https://www.adafruit.com/product/2034)

 We have during the hackfest encountered some eccentric readings the sensors, which was quickly diagnosed as dry solder joint and resolved easily.

- For production, testing needs to be done to determine real-life operating environment. There include

 - The expect lifespan of the device in production environment, when exposed to extreme temperature and humidity.
 - Interferences from other components, and their effect on the device.
 - The Wi-Fi connectivity strength and acceptable range for stable connection.
 - Case studies, best practices from other devices utilizing ESP8266 sold in market
 - Certification to ensure compliance to local law

- Fabrication of PCB is a key process as cost of designing and ease of manufacturing is at a all time low. Knowledge in PCB design via [EagleCAD](https://cadsoft.io/) or other software is an essential skill. Some companies (non exhaustive) which allows online ordering via EagleCAD file includes

 - [OSHPark](https://oshpark.com/)
 - [Seeed Studio](https://www.seeedstudio.com/fusion_pcb.html)


## Conclusion ##

We see a myriad of IOT Devices available in the market today, ranging from popular branded devices to up and coming underdog brand names. The constant experimenting and tinkering with these new devices are part and parcel of the Maker's mentality which drives the IOT projects in the real-world environment. Nevertheless it takes the early adopter or innovator's mindset to implement these devices way before it is tested and proven on the field. 

There are risks involved in adopting new technologies, which can become lesser if mitigated properly implementing additional layers of control


- Measurable impact and benefits resulting from the implementation of the solution.
 - The current device gateway alone cost over $100 per node. This Hackfest demonstrated the feasibility of low cost gateway $10 device (NodeMCU V1.0 based on ESP8266) as an alternative. 
 - As there is lesser dependency on structured SQL Server for cool/history data which are rarely accessed, the CPU cycles and RAM utilization are more efficiently  used for data queries in the past 12 months.

- General lessons:

  - Periodical exploration of new IOT devices can often be a fruitful experience as devices gets smaller and increasingly affordable.  
  - For remote monitoring projects, the strategy of storage is important to manage the initial operational cost. Data analytics typically requires months and even years of historical data before qualitative analysis can be churned out from these raw telemetry data.

- Next Steps:

  - The currently prototype needs further work to be production and deployment ready, and moving forward would be to start designing and creating [custom Manufactured PCB Board](https://www.seeedstudio.com/fusion_pcb.html) for testing in real environment.


## Additional resources ##
If you are interested to find out the capabilities of NODEMCU based devices do check out 

- [Hackster.io NodeMCU Community](https://www.hackster.io/nodemcu/projects)

- [Hackster.io ESP based devices Community](https://www.hackster.io/esp)

- [Microsoft Internet of Things Blog](https://blogs.microsoft.com/iot/)

- [Microsoft Hackster Community Page](https://microsoft.hackster.io/en-US)

