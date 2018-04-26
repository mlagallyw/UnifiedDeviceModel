# Unified Device Model Proposal

## Starting point

Different and incompatible digital twin abstractions in major IoT cloud platforms.
[Richard: Obviously could beef this up a bit :-)]


## Terminology

* Thing or Device
	A physical or logical entity that is managed by an IoT Cloud Service

* Digital twin
	A (virtual) representation of a device in an IoT Cloud Service

* Device Description
	A serialized representation of a Device
	[Richard: I don't know what this means. It isn't the twin itself, and it isn't the model, so what is a device
	description? This seems to be an entity with no purpose.]

* Device Model (aka. Device Template, Device Blueprint)
	A schema that describes the structure and interface of a group of devices
	[Richard: I think it is important to avoid the word "template" or equating this to a template. A template is typically
	something like a "mold" or stamp for producing many of a thing, but doesn't carry the notion of a persistent
	relationship with the created thing. Device Models have a persistent relationship. Blueprint is a reasonable name]

* Properties
	A part of the device that contains state information and may change over time
	[Richard: Is it a part of the device? Or a part of the twin? Or the model? Or all three?]

* Messages
	Telemetry data that is sent from the device to the cloud
	[Richard: What about messages that go from cloud to device? What about messages not related to "data" such as alerts?]

## Microsoft's Device Twin

Device twins can contain arbitrary JSON objects as both tags and properties. The Device Twin is the cloud representation
of an actual device. Microsoft does not enforce any kind of model or schema on the type of data that can be sent from
devices, or the type of actions that can be invoked on devices. Each twin has a device ID.

"Tags" of a twin are used to save application specific data with the twin, such as the location or brand of a vending
machine.

The twin also contains various metadata about the device such as connection state, last activity time, and so forth.

The key thing that the twin exposes is the "desired" and "reported" state of the device as two separate fields.
[Richard: I added some more words here]

```javascript
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
```

## Microsoft's Device API

Get Device Twin
Get a device twin.

Invoke Device Method

Invoke a direct method on a device.

Update Device Twin

Updates tags and desired properties of a device twin.

https://docs.microsoft.com/en-us/rest/api/iothub/devicetwinapi/invokedevicemethod#cloudtodevicemethod


## Microsoft's Device Model Schema

Microsoft uses a Device Model Schema to define simulated devices for remote monitoring. The schema defines a set of
properties, telemetry-messages, a simulation model and cloud-to-device methods.

https://docs.microsoft.com/en-us/azure/iot-suite/iot-suite-remote-monitoring-device-schema
[Richard: This isn't part of the core cloud product but is part of a set of solutions you can download to get you
started with Azure IoT. Specifically, this is for running a simulation. We need to be clear in our understanding
that the design center for Azure IoT does not include the notion of a device model or blueprint.]

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
[Richard: Our device model is not a template. A template is 'A pattern or gauge, such as a thin metal plate with a cut
pattern, used as a guide in making something accurately, as in woodworking or the carving of architectural profiles'.
This implies that a template is used only to create an instance, but thereafter maintains no relationship with it.
The IoT Device Model is actually an interface definition, and maintains a relationship with the devices that claim
to implement it. I wouldn't say it is used to define new device instances at all. Rather, the device model represents
the interface or contract of a device dictating the properties and behavior of a device. Digital Twins use this
contract for knowing what kind of data will come from the device and what kind of actions can be sent to the device.]

Device models are managed by the IoT Cloud Service platform and are used to create and register device instances. The
IoT Cloud Service platform defines a REST-API to query, export and import device models.

A device model defines a set of metadata fields, properties, actions, message formats and links.
[Richard: I'm not sure it is worth calling out links. This is just part of the HATEOAS pattern and not intrinsic to
what a device model is].

It is protocol agnostic, however it defines globally unique message formats that define the types and content of
messages that a device model emits and receives. A device model has a globally unique id (urn) that by convention
follows reverse DNS naming conventions, similar to Java packages. This design allows for a global repository of device
models to be shared across IoT Cloud Services.

A concrete device and it's digital twin may implement more than one device model.

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
[Richard: We have to be clear on this point -- The Oracle Device Model is an essential part of the design space for
Oracle IoT and defines the *contract* that the device implements, whereas the Microsoft Device Model is used for
defining a simulated device. The Microsoft Device Model plays no part in Microsoft digital twin design, implementation,
or use.]

### Properties
The **Oracle device model** defines data types for properties (simple JSON types and a few extensions: URL, time+date).
A type range may be specified for numeric types; write access can be restricted. Properties may contain a description
and an alias.

The properties in the **Microsoft device model** example are plain JSON values. The example does not contain types and
type ranges, description or alias fields.

### Actions

Both models define actions that can be performed on a device (and a device twin). The **Microsoft device model** does
not define descriptions, alias or parameter types. All actions in the example do not contain parameters. The example
binds each action to an implementation via a Javascript file.

The **Oracle device model** defines actions with a parameter of a simple JSON type. An action can have a
description and an alias.

### Message formats

Both models provide a way to define the payload structure of messages.

# A unified approach

Define a common device model for describing digital twins at the enterprise side.
Protocol agnostic, security mechanism and lifecycle are initially out of scope.

Devices following this model can be defined by device manufacturers or cloud vendors
and can be shared across different cloud platforms. Enterprise applications can use
digital twins in the same way, independently from which cloud vendor is hosting them.

[Richard: We need to have an in depth discussion here of *why* this matters. Does it? If I'm Microsoft, I want to know
why I have to take on all this additional complexity to define this contract. Microsoft IoT sees itself as a
pass-through system, part of a constellation of services needed to create an IoT solution. Oracle IoT on the other hand
is a complete, comprehensive solution with vertical applications. As an application agnostic platform, Microsoft is
able to wave their hands about things like message formats, whereas for Oracle IoT with built in analytics and vertical
apps, knowing the structure of messages is crucial.]

[Richard: So, if I'm Microsoft and I'm doing a platform and not a vertical app, why do I care about Device Model
contracts? They're going to make development more difficult by creating a longer modify-build-test cycle. Is there a
way in which Device Models can be skipped and do things dynamically and have it work? Or are we convinced that
Device Models with concrete contracts the only way to build stable systems and can clearly articulate that?]

### Common set of metadata fields

The set of metadata values are limited to plain JSON types only.

[Richard: Are we talking device models here or digital twins! "activationTime" is related to a digital twin, not a 
device model. Same for "id" and "hardwareId", and yet you then have Property and Actions sections that are related
to the device model! This keeps getting conflated but they are fundamentally different things.]

#### candidates

            "id": "FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC",
            "hardwareId": "FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC-Oracle-CH101-SerialNr 1234",
            "type": "DIRECTLY_CONNECTED_DEVICE",
            "description": "Sample Chiller Device Twin 1",
            "created": 1524654286431,
            "createdAsString": "2018-04-25T11:04:46.431Z",
            "activationTime": 1524654287241,
            "activationTimeAsString": "2018-04-25T11:04:47.241Z",
            "state": "ACTIVATED",
            "name": "Chiller-02-1",
            "manufacturer": "Oracle",
            "modelNumber": "CH101",
            "serialNumber": "SerialNr 1234",
            "hardwareRevision": "",
            "softwareRevision": "",
            "softwareVersion": "",
            "enabled": true


### Property

        {
            "alias": "ora_lon",
            "name": "ora_longitude",
            "range": "-180.0,180.0",
            "type": "NUMBER",
            "writable": false
        }


#### Actions

        {
            "alias": "IncreasePressure-method",
            "argType": "INTEGER",
            "description": "Increase the pressure with the value in the argument",
            "name": "IncreasePressure",
            "range": "0,5"
        }


#### Message formats

t.b.d. in phase 2




# JSON schema (http://json-schema.org) for the Oracle device model (informative)

	{
		"$schema ": "http: //json-schema.org/schema#",
	    "id": "http://oracle.com/schemas/device-model.json",
		"title": "Oracle Device Model",
		"description": "JSON Schema representation of the Oracle device model serialisation format.",

		"type": "object",

		"properties": {
			"urn": { "type": "string" },
			"name": { "type": "string" },
			"description": { "type": "string" },
	        "system" : { "type" : "boolean" },
	        "draft": { "type" : "boolean" },
	  		"created": { "type": "integer" },
	  		"createdAsString": { "type": "string" },
	  		"lastModified": { "type": "integer" },
	  		"lastModifiedAsString": { "type": "string" },
	  		"userLastModified": { "type": "string" },

			"attributes": { "type": "array",
	                        "items": { "$ref": "#/definitions/attribute" } },
			"actions": { "type": "array",
	                     "items": { "$ref": "#/definitions/action" } },
			"formats": { "type": "array",
	                     "items": { "$ref": "#/definitions/format" } },
	        "links": { "type": "array",
					   "items": { "$ref": "#/definitions/link" } }
		},
		"required": ["urn", "name", "description", "attributes"],
		"additionalProperties": false,

	 	"definitions": {
	        "format_type" : { "enum": [ "ALERT", "DATA" ] },
	        "primitive_type" : { "enum": [ "NUMBER", "STRING", "BOOLEAN", "DATETIME", "INTEGER" ] },

		    "format_items": {
				"type": "object",
				"fields": {
				 	"type": "array",
				 	"items": {
				    	 	"name": { "type": "string" },
				     	"type": { "type": { "$ref": "#/definitions/primitive_type" } },
				     	"optional": { "type": "boolean" }
				 	},
					"required": [ "name", "type" ]
				}
	        },

			"attribute": {
				"type": "object",
				"properties": {
					"name": { "type": "string" },
	                "alias": { "type": "string" },
					"description": { "type": "string" },
					"type" : { "$ref": "#/definitions/primitive_type" },
	                "range": { "type": "string" },
	                "writable": { "type": "boolean" },
	                "defaultValue": {
	                    "oneOf": [
	                          { "type" : "boolean" },
	                          { "type" : "number" },
	                          { "type" : "string" }
	                     ]
	                }
		   	    },
		   	    	"required": [ "name", "description", "type" ]
	        },

			"action": {
				"type": "object",
				"properties": {
					"name": { "type": "string" },
					"description": { "type": "string" },
					"argType" : { "$ref": "#/definitions/primitive_type" }
				},
				"required": [ "name", "description" ]
			},

			"format": {
				"type": "object",
				"properties": {
	       	        "urn": { "type": "string" },
					"name": { "type": "string" },
					"description": { "type": "string" },
					"type" : { "$ref": "#/definitions/format_type" },
	                "value": { "$ref": "#/definitions/format_items" }
				}
			},

			"link": {
				"type": "object",
				"properties": {
					"href": { "type": "string" },
					"rel": { "type": "string" }
				},
				"required": ["href", "rel"]
			}
		}
	}


# A concrete Oracle device twin that implements several device models
[Richard: Where are the "desired" or "reported" states? This below isn't our virtual device, but this is our current
device object. We have Device Objects, Device Models, Virtual Devices, and Message Formats. We don't actually have a
JSON document today representing the "digital twin" as such. We need one, but the below doesn't have everyting that
is needed. We have some messiness we need to resolve w.r.t. virtual devices.]

[Richard: A Device Model represents a contract with the device, and a single device can implement multiple device
models, exactly like how a single class in Java can implement multiple interfaces.]

[Richard: A Device is a record in the database that represents the device. It has timestamps, description, IDs, etc.]

[Richard: A Virtual Device is the per-device-instance of the device model. It has attributes (properties) and such.]

[Richard: This design works because it allows a single Device to implement multiple Device Models and provides
namespacing for each Device Model, such that each Device Model can define its own names without worrying about conflict
should the same name be defined by two Device Models both implemented by a single Device. Unfortunately, this design is
horrible from a conceptual / complexity perspective. Was it over-engineered for real-world cases? Possibly. However it
did give us some nice benefits in our Gateway design, where we could implement internal things like "ping" testing as
a Device Model that all devices using our client library would implement.]

[Richard: So where does a Digital Twin really fit into our system? Is it a Virtual Device? The Device object itself?
Or is it some new entity that is a cross between Virtual Devices and Device objects? One thing is clear, it isn't the
device model (although it may refer to a device model, and may even embed the device model attributes, it isn't the
model itself. The ramification of this means that you **cannot** edit the device model portion of a digital twin,
you can only read it. To edit it, you would have to go find the corresponding device model and edit **that**, and if it
were supplied by a vendor as opposed to the developer themselves, it might not even be editable).]

[Richard: I feel like the Digital Twin should be a combination of Virtual Device and Device, and the model can be
embedded like it is today. The Digital Twin can have an ID, but the ID will not be the device ID, or the hardware ID.
Rather, it is the Digital Twin ID. To get a Digital Twin, you would need to know the device ID and the Device Model
URN so that you can find a Digital Twin that represents that aspect of the Device. OR, you have to accept that fact that
properties *do not* live on the Digital Twin directly, but are nested based on the device ID. Either approach is
logically consistent with the notion of a device implementing multiple device models. The other thing we can do is
revisit that notion that a device *can* implement multiple device models, and say that no, it cannot, and we were
wrong to do so, and should go back to an earlier design where we had "capabilities" as separate things from device
models and a device could implement multiple capabilities (like ping) without having them be device models. In any
event I think multiple device models makes sense for some use cases (for example a vendor may have a device that
conforms to one or more "standard" device model specifications but also has their own device model full of other
goodies and vendor extensions).]

        {
            "id": "FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC",
            "hardwareId": "FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC-Oracle-CH101-SerialNr 1234",
            "type": "DIRECTLY_CONNECTED_DEVICE",
            "description": "Sample Chiller Device Twin 1",
            "created": 1524654286431,
            "createdAsString": "2018-04-25T11:04:46.431Z",
            "activationTime": 1524654287241,
            "activationTimeAsString": "2018-04-25T11:04:47.241Z",
            "state": "ACTIVATED",
            "name": "Chiller-02-1",
            "manufacturer": "Oracle",
            "modelNumber": "CH101",
            "serialNumber": "SerialNr 1234",
            "hardwareRevision": "",
            "softwareRevision": "",
            "softwareVersion": "",
            "enabled": true,
            "deviceModels": [
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
                    ]
                },
                {
                    "urn": "urn:oracle:iot:dcd:capability:message_dispatcher",
                    "name": "Message Dispatcher Capability",
                    "description": "Oracle IoT device capability definition for messaging dispatcher",
                    "system": true,
                    "draft": false,
                    "created": 1515576128000,
                    "createdAsString": "2018-01-10T09:22:08.000Z",
                    "lastModified": 1515576128000,
                    "lastModifiedAsString": "2018-01-10T09:22:08.000Z",
                    "userLastModified": "iot",
                    "attributes": [],
                    "actions": [],
                    "formats": [
                        {
                            "urn": "urn:oracle:iot:dcd:capability:message_dispatcher:overflow",
                            "name": "Message Dispatcher Overflow Alert Message",
                            "description": "Gateway logging message overflow alert message format definition for message dispatcher.",
                            "type": "ALERT",
                            "deviceModel": "urn:oracle:iot:dcd:capability:message_dispatcher",
                            "value": {
                                "fields": [
                                    {
                                        "name": "description",
                                        "optional": false,
                                        "type": "STRING"
                                    },
                                    {
                                        "name": "maxNumMessages",
                                        "optional": false,
                                        "type": "INTEGER"
                                    }
                                ]
                            },
                            "sourceId": "urn:oracle:iot:dcd:capability:message_dispatcher",
                            "sourceType": "DEVICE_MODEL"
                        }
                    ]
                },
                {
                    "urn": "urn:oracle:iot:dcd:capability:direct_activation",
                    "name": "Direct Activation Capability",
                    "description": "Oracle IoT capability definition for direct device activation",
                    "system": true,
                    "draft": false,
                    "created": 1515576128000,
                    "createdAsString": "2018-01-10T09:22:08.000Z",
                    "lastModified": 1515576128000,
                    "lastModifiedAsString": "2018-01-10T09:22:08.000Z",
                    "userLastModified": "iot",
                    "attributes": [],
                    "actions": [],
                    "formats": []
                },
                {
                    "urn": "urn:oracle:iot:dcd:capability:diagnostics",
                    "name": "Device Diagnostics",
                    "description": "Oracle IoT capability definition for device diagnostics",
                    "system": true,
                    "draft": false,
                    "created": 1515576128000,
                    "createdAsString": "2018-01-10T09:22:08.000Z",
                    "lastModified": 1515576128000,
                    "lastModifiedAsString": "2018-01-10T09:22:08.000Z",
                    "userLastModified": "iot",
                    "attributes": [],
                    "actions": [],
                    "formats": [
                        {
                            "urn": "urn:oracle:iot:dcd:capability:diagnostics:test_message",
                            "name": "Diagnostics Test Connectivity Data Message",
                            "description": "Test connectivity data message format definition for device diagnostics.",
                            "type": "DATA",
                            "deviceModel": "urn:oracle:iot:dcd:capability:diagnostics",
                            "value": {
                                "fields": [
                                    {
                                        "name": "current",
                                        "optional": false,
                                        "type": "INTEGER"
                                    },
                                    {
                                        "name": "count",
                                        "optional": false,
                                        "type": "INTEGER"
                                    },
                                    {
                                        "name": "payload",
                                        "optional": false,
                                        "type": "STRING"
                                    }
                                ]
                            },
                            "sourceId": "urn:oracle:iot:dcd:capability:diagnostics",
                            "sourceType": "DEVICE_MODEL"
                        }
                    ]
                }
            ],
            "logs": {
                "logCount": 1,
                "logType": "ACTIVATION",
                "logs": [
                    {
                        "timeStamp": "2018-04-25T11:04:47Z",
                        "message": "Device Activated"
                    }
                ]
            },
            "connectivityStatus": "ONLINE",
            "lastHeardTime": 1524660118821,
            "onlineSinceTime": 1524660118821,
            "onlineSinceTimeAsString": "2018-04-25T12:41:58.821Z",
            "lastHeardTimeAsString": "2018-04-25T12:41:58.821Z",
            "links": [
                {
                    "href": "https://129.144.182.85:443/iot/api/v2/devices/FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC",
                    "rel": "self"
                }
            ],
            "devices": {
                "links": [
                    {
                        "href": "https://129.144.182.85:443/iot/api/v2/devices/FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC/devices",
                        "rel": "self"
                    }
                ]
            },
            "software": {
                "links": [
                    {
                        "href": "https://129.144.182.85:443/iot/api/v2/devices/FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC/software",
                        "rel": "self"
                    }
                ]
            },
            "metadata": {
                "links": [
                    {
                        "href": "https://129.144.182.85:443/iot/api/v2/devices/FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC/metadata",
                        "rel": "self"
                    }
                ]
            },
            "location": {
                "links": [
                    {
                        "href": "https://129.144.182.85:443/iot/api/v2/devices/FBE6F52D-CCE1-40F5-9C81-9F4B5C7F37CC/location",
                        "rel": "self"
                    }
                ]
            }
        },
