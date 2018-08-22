# Analysis of Microsoft's and Oracle's device models

## What is a Device Model?

A device model is a formal description of a class of physical devices.
The description contains entries for metadata fields (tags), properties (attributes) and actions.

Metadata fields contain static values such as "device manufacturer", "hardware version", "creation date".

Properties contain values that typically change over time, such as sensor readings, actuator settings, status values or configuration information.

Actions are called to trigger an activity on the device.

This abstract model of a device can be applied to all devices in the market. It intentionally does not talk about messages and protocols or specific program languages.

## Purpose of a Device Model

A device model can be used to create applications that interact with a device. In today's world, specific applications have to be written per device class. These applications typically operate with an implicit model of a device, i.e. they contain code to manipulate device properties or to call actions. The programmer has to read a device manual to implement his application.

With a device model a more generic application (or library) can be created, which can interact with all devices that implement this common description. This reduces the effort for the integration of devices in IoT scenarios.

## Purpose of this document

There are several device models in the market, which are used in conjunction with specific protocols or define a generic mechanism for protocol binding.
Some of them are standardised in different SDOs such as the W3C, OCF, IPSO and others.

Large IoT cloud vendors also use different models in their products. Some of them only define metadata and properties but leave it to the applications to handle actions and messages.

This fragmentation limits interoperability and market adoption of IoT devices.

This analysis was conducted to identify commonalities of two real-world device models to lay the groundwork for a unified device model, which serves as a common abstraction across cloud and device platforms. The benefit of a unified device model is to simplify device integration and interoperability between (typically small embedded) devices and IoT service platforms from multiple vendors.

A high level of interoperability brings faster time-to-market, since it minimizes the integration effort between device manufacturers and cloud vendors.
It protects the customer investment and makes device and applications future-proof, since it enables easy integration across multiple clouds.

Furthermore it enables migration scenarios, where devices can continue to be used when one of the cloud platform providers discontinues his service.

## Background

Products from major cloud vendors and IoT standards:

##### W3C Web of Things
[https://w3c.github.io/wot-thing-description/]()

##### Microsoft's Device Twin
[https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/iot-hub/iot-hub-devguide-device-twins.md]()

##### Amazon's Device Shadow
[https://docs.aws.amazon.com/iot/latest/developerguide/what-is-aws-iot.html]()

##### Mozilla's Web Thing API
[https://iot.mozilla.org/wot/]()

## Terminology

The following section contains terminology definitions that are used by this specification. Not that some
products in the market use the same concepts under a different terminology, e.g. "attributes" for "properties", or "properties" for "class or devcie metadata".

* Device (Device-Instance)

	A physical or logical entity that is managed by an IoT Cloud Service

* Device Class

	A group of devices that implement the same Device Model

* Digital Twin

	A (virtual) representation of a device in an IoT Cloud Service

* Device Description  
	A textual description of a device instance

* Device Model 	
	A textual description of a device class

* Property  
	A part of the device that contains state information and may change over time

* Action  
	A part of a device that can be invoked from the outside to change the state of a device

* Metadata  
	A part of the device that contains values that describe the device class or device instance

* Class Metadata  
	Metadata elements that are common for all devices of the class.

* Device Metadata (Instance Metadata)
	Metadata elements that apply for instances of the class. Examples are the geographic location, software version, name, id.

* Messages  
  	Data that is being sent between the device and the cloud


## Microsoft's device twin

Device twins can contain arbitrary JSON objects as both tags and properties.


	{
	    "deviceId": "myDeviceId",
	    "etag": "AAAAAAAAAAc=",
	    "status": "enabled",
	    "statusUpdateTime": "0001-01-01T00:00:00",    
	    "connectionState": "Disconnected",    
	    "lastActivityTime": "0001-01-01T00:00:00",
	    "cloudToDeviceMessageCount": 0,
	    "authenticationType": "sas",    
	    "x509Thumbprint": {    
	        "primaryThumbprint": null,
	        "secondaryThumbprint": null
	    },
	    "version": 2,
	    "tags": {
	        "location": {
	            "region": "US",
	            "plant": "Redmond43"
	        }
	    },
	    "properties": {
	        "desired": {
	            "telemetryConfig": {
	                "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",
	                "sendFrequencyInSecs": 300
	            },
	            "$metadata": {
	            ...
	            },
	            "$version": 4
	        },
	        "reported": {
	            "connectivity": {
	                "type": "cellular"
	            },
	            "telemetryConfig": {
	                "configId": "db00ebf5-eeeb-42be-86a1-458cccb69e57",
	                "sendFrequencyInSecs": 300,
	                "status": "Success"
	            },
	            "$metadata": {
	            ...
	            },
	            "$version": 7
	        }
	    }
	}


## Microsoft's Device API

The Device twin API defines the following methods:

| Function | Description |
| -------- | ----------- |
| Get Device Twin | Get a device twin. |
| Invoke Device Method | Invoke a direct method on a device. |
| Update Device Twin | 	Updates tags and desired properties of a device twin. |

https://docs.microsoft.com/en-us/rest/api/iothub/devicetwinapi


## Microsoft's Device Model Schema

Microsoft uses a Device Model Schema to define simulated devices for remote monitoring. The schema defines a set of properties, telemetry-messages, a simulation model and cloud-to-device methods.

https://docs.microsoft.com/en-us/azure/iot-suite/iot-suite-remote-monitoring-device-schema

	{
	  "SchemaVersion": "1.0.0",
	  "Id": "elevator-01",
	  "Version": "0.0.1",
	  "Name": "Elevator",
	  "Description": "Elevator with floor, vibration and temperature sensors.",
	  "Protocol": "AMQP",
	  "Simulation": {
	    // Specify the simulation behavior
	  },
	  "Properties": {
	    // Define properties
	  },
	  "Telemetry": [
	    // Specify telemetry
	  ],
	  "CloudToDeviceMethods": {
	    // Specify methods
	  }
	}

### Microsoft Device Model Schema example (chiller-02)

The following schema example from https://github.com/Azure/device-simulation-dotnet/blob/master/Services/data/devicemodels/chiller-02.json defines a device model of a chiller.

It defines a concrete device instance (properties and tags) with a simulation model and interactions via methods.
The model defines a specific protocol (MQTT) and a Telemetry message template and schema for message payloads.  It describes interactions via a set of CloudToDevice methods, which are implemented by Javascript callbacks.

The simulation is initialized with an initial state and an update script (chiller-02-state.js) which is
called at regular intervals.

The example below defines the schema as well as a simulation and a set of properties and tags for a specific device instance.

	{
	    "SchemaVersion": "1.0.0",
	    "Id": "chiller-02",
	    "Version": "0.0.1",
	    "Name": "Faulty Chiller",
	    "Description": "Faulty chiller with wrong pressure sensor. Pressure too high.",
	    "Protocol": "MQTT",
	    "Simulation": {
	        "InitialState": {
	            "online": true,
	            "temperature": 75.0,
	            "temperature_unit": "F",
	            "humidity": 70.0,
	            "humidity_unit": "%",
	            "pressure": 250.0,
	            "pressure_unit": "psig",
	            "simulation_state": "high_pressure"
	        },
	        "Interval": "00:00:10",
	        "Scripts": [
	            {
	                "Type": "javascript",
	                "Path": "chiller-02-state.js"
	            }
	        ]
	    },
	    "Properties": {
	        "Type": "Chiller",
	        "Firmware": "1.0",
	        "Model": "CH101",
	        "Location": "Building 43",
	        "Latitude": 47.639318,
	        "Longitude": -122.134271
	    },
	    "Tags": {
	        "Location": "Building 43",
	        "Floor": "3",
	        "Campus": "Redmond"
	    },
	    "Telemetry": [
	        {
	            "Interval": "00:00:10",
	            "MessageTemplate": "{\"temperature\":${temperature},\"temperature_unit\":\"${temperature_unit}\",\"humidity\":${humidity},\"humidity_unit\":\"${humidity_unit}\",\"pressure\":${pressure},\"pressure_unit\":\"${pressure_unit}\"}",
	            "MessageSchema": {
	                "Name": "chiller-sensors;v1",
	                "Format": "JSON",
	                "Fields": {
	                    "temperature": "double",
	                    "temperature_unit": "text",
	                    "humidity": "double",
	                    "humidity_unit": "text",
	                    "pressure": "double",
	                    "pressure_unit": "text"
	                }
	            }
	        }
	    ],
	    "CloudToDeviceMethods": {
	        "Reboot": {
	            "Type": "javascript",
	            "Path": "Reboot-method.js"
	        },
	        "FirmwareUpdate": {
	            "Type": "javascript",
	            "Path": "FirmwareUpdate-method.js"
	        },
	        "EmergencyValveRelease": {
	            "Type": "javascript",
	            "Path": "EmergencyValveRelease-method.js"
	        },
	        "IncreasePressure": {
	            "Type": "javascript",
	            "Path": "IncreasePressure-method.js"
	        }
	    }
	}

## Oracle's Device Model

Oracle's device model serves as a blueprint/template to define new device instances (digital twins).
Device models are managed by the IoT Cloud Service platform and are used to create and register device instances. The IoT Cloud Service platform defines a REST-API to query, export and import device models.

A device model defines a set of metadata fields, properties, actions, message formats and links.
It is protocol agnostic, however it defines globally unique message formats
A device model has a globally unique id (urn).

A concrete device and it's digital twin may implement more than one device models.

### Oracle device model example (chiller-02)


	{
	    "urn": "urn:com:oracle:iot:chiller-02",
	    "name": "chiller-02",
	    "description": "Faulty chiller with wrong pressure sensor. Pressure too high.",
	    "system": false,
	    "draft": false,
	    "created": 1524652689668,
	    "createdAsString": "2018-04-25T10:38:09.668Z",
	    "lastModified": 1524653984604,
	    "lastModifiedAsString": "2018-04-25T10:59:44.604Z",
	    "userLastModified": "IoTAdmin",
	    "attributes": [
	        {
	            "description": "",
	            "name": "Type",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "Firmware",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "Model",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "Location",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "Latitide",
	            "type": "NUMBER",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "Longitude",
	            "type": "NUMBER",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "Location_tag",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "floor level",
	            "name": "Floor_tag",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "name of the campus",
	            "name": "Campus_tag",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "online status",
	            "name": "online",
	            "type": "BOOLEAN",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "temperature",
	            "type": "NUMBER",
	            "writable": true
	        },
	        {
	            "description": "C for Celsius, F for Fahrenheit",
	            "name": "termperature_unit",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "humidity of the chiller",
	            "name": "humidity",
	            "type": "NUMBER",
	            "writable": true
	        },
	        {
	            "description": "percentage or grams per m3",
	            "name": "humidity_unit",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "description": "",
	            "name": "pressure",
	            "type": "NUMBER",
	            "writable": true
	        },
	        {
	            "description": "psi or pascal or bar",
	            "name": "pressure_unit",
	            "type": "STRING",
	            "writable": true
	        },
	        {
	            "alias": "ora_lat",
	            "name": "ora_latitude",
	            "range": "-90.0,90.0",
	            "type": "NUMBER",
	            "writable": false
	        },
	        {
	            "alias": "ora_lon",
	            "name": "ora_longitude",
	            "range": "-180.0,180.0",
	            "type": "NUMBER",
	            "writable": false
	        },
	        {
	            "alias": "ora_alt",
	            "name": "ora_altitude",
	            "type": "NUMBER",
	            "writable": false
	        },
	        {
	            "alias": "ora_accuracy",
	            "name": "ora_uncertainty",
	            "type": "NUMBER",
	            "writable": false
	        },
	        {
	            "name": "ora_zone",
	            "type": "STRING",
	            "writable": false
	        },
	        {
	            "alias": "ora_mssi",
	            "name": "ora_txPower",
	            "type": "INTEGER",
	            "writable": false
	        },
	        {
	            "name": "ora_rssi",
	            "type": "NUMBER",
	            "writable": false
	        }
	    ],
	    "actions": [
	        {
	            "alias": "Reboot-method",
	            "description": "Reboot the chiller",
	            "name": "Reboot"
	        },
	        {
	            "alias": "FirmwareUpdate-method",
	            "argType": "STRING",
	            "description": "Update the firmware to the version specified in the argument",
	            "name": "FirmwareUpdate"
	        },
	        {
	            "alias": "EmergencyValveRelease-method",
	            "description": "Open valve for emergency reasons",
	            "name": "EmergencyValveRelease"
	        },
	        {
	            "alias": "IncreasePressure-method",
	            "argType": "INTEGER",
	            "description": "Increase the pressure with the value in the argument",
	            "name": "IncreasePressure",
	            "range": "0,5"
	        }
	    ],
	    "formats": [
	        {
	            "urn": "urn:com:oracle:iot:chiller-02-sensors",
	            "name": "chillersensors",
	            "description": "",
	            "type": "DATA",
	            "deviceModel": "urn:com:oracle:iot:chiller-02",
	            "value": {
	                "fields": [
	                    {
	                        "name": "temperature",
	                        "optional": false,
	                        "type": "NUMBER"
	                    },
	                    {
	                        "name": "temperature_unit",
	                        "optional": false,
	                        "type": "STRING"
	                    },
	                    {
	                        "name": "humidity",
	                        "optional": false,
	                        "type": "NUMBER"
	                    },
	                    {
	                        "name": "humidity_unit",
	                        "optional": false,
	                        "type": "STRING"
	                    },
	                    {
	                        "name": "pressure",
	                        "optional": false,
	                        "type": "NUMBER"
	                    },
	                    {
	                        "name": "pressure_unit",
	                        "optional": false,
	                        "type": "STRING"
	                    }
	                ]
	            },
	            "sourceId": "urn:com:oracle:iot:chiller-02",
	            "sourceType": "DEVICE_MODEL"
	        }
	    ],
	    "links": [
	        {
	            "href": "https://129.144.182.85:443/iot/api/v2/deviceModels/urn%3Acom%3Aoracle%3Aiot%3Achiller-02",
	            "rel": "self"
	        },
	        {
	            "href": "https://129.144.182.85:443/iot/api/v2/deviceModels/urn%3Acom%3Aoracle%3Aiot%3Achiller-02",
	            "rel": "canonical"
	        }
	    ]
	}

# Comparison of the two device models

Both models define a set of properties and interactions with the device via actions.

### Properties
The **Oracle device model** defines data types for properties (simple JSON types and a few extensions: URL, time+date).
A type range may be specified for numeric types; write access can be restricted.
Properties may contain a description and an alias.

The properties in the **Microsoft device model** example are plain JSON values. The example does not contain types and type ranges, description or alias fields.

### Actions

Both models define actions that can be performed on a device (and a device twin). The **Microsoft device model** does not define descriptions, alias or parameter types. All actions in the example do not contain parameters. The example binds each action to an implementation via a Javascript file.

The **Oracle device model** defines actions with **only a single** parameter of a **primitive JSON** type. An action can have a
description and an alias.

### Message protocols and formats

Both models provide a way to define the payload structure of messages between the device and the cloud service via message formats. Even though the payload formats are defined in the model, the actual transport protocols are usually proprietary and not exposed through the model.


