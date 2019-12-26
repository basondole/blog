## Streaming Telemetry

Streaming telemtry allows for measurements and data that is collected from remote devices to be sent to receiving equipment for monitoring,
it works in a push manner where the devices are not periodically polled for information as with SNMP,
and this solves performance issue that may be caused by stressed CPU due to aggressice SNMP polling.
Streaming telemetry provides real-time analytics-ready data using yang data model encoded as `json` `xml` or `GBP` streamed over tcp or udp.
GBP stands for Google Protocol Buffers which is like a smaller, faster and simpler xml.

Streaming telemetry aims to modernize the collection of network and device metrics to keep up with the scale of next-generation networks, and provide new ways to access the huge variety of metrics that network devices can now generate.  
Steaming telemetry uses a push-based mechanism with which data can be transmitted automatically and continuously from various remote sources (such as routers, switches, firewalls, etc.) to some centralized platform for storage and analysis.
Streaming telemetry has the potential to provide near real-time network data, achieved with push-based data collection
- Transport options: `tcp` `udp` `gRPC`
- Session Initiation options: `dial-out` (device sends data to collector) and `dial-in` (collector connects into the device)
- Encoding options: `json` `xml` and `gbp`

### Streaming formats
With streaming telemetry, the telemetry data is described using YANG, a structured data modelling language, encoded in JSON, XML or using GBP and is then streamed over TCP, UDP or gRPC

#### Native Streaming
This format uses a proprietary data model defined by the equipment vendor, using `gpb` as a means of serializing and structuring telemetry data messages. Data is transported via UDP and is exported close to the source, such as directly from a line card or network processing unit (NPU).  

#### OpenConfig Streaming
This format utilizes OpenConfig data models and key/value pair based Google protocol buffer messages for streaming telemetry data. Unlike native format, data is transported via gRPC over HTTP2 and is exported to the collector centrally from the Routing Engine (RE).
