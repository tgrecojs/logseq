- 📌 The WebSocket protocol was created to give applications in web browsers access to TCP-based socket APIs, hence “WebSockets”
  hid:: U2LMejNdEe6wJ39JIshK0A
  updated:: 2023-08-05T06:57:21.847666+00:00
- 📌 They both run over TCP connections, but the WebSockets functionality is more comparable with TCP itself rather than MQTT.
  hid:: WShxRjNdEe6KRKdKRRPayw
  updated:: 2023-08-05T06:57:31.539330+00:00
- 📌 Once an MQTT connection is established, any number of messages can be sent through it in both directions, data from sensor to back-end, and commands the other way. WebSockets allows the same behavior when working in conjunction with MQTT, but at the cost of higher overheads.
  hid:: YNg2VjNdEe69WhN3XeO_lQ
  updated:: 2023-08-05T06:57:44.439071+00:00
- 📌 MQTT is a communication protocol with features specifically targeted at IoT solutions:
  hid:: Zym29jNdEe6Vnsc33hAIEw
  updated:: 2023-08-05T06:57:55.049725+00:00
- 📌 Uses TCP connections, for reliability (assured delivery and packet error checking), fragmentation and ordering.
  hid:: 4-BqHjNdEe6KUQ9ZlBOZOw
  updated:: 2023-08-05T07:02:17.588469+00:00
  Aims to minimize data overhead of each MQTT packet.
  The last known good data value for a device can be stored (retained messages).
  Notifications when client unexpectedly disconnects (will message) to allow client state to be monitored.
  Bi-directional message flow - data from and commands to devices can use the same TCP connection.
  Publish subscribe routing, which allows the easy addition of more consumers and producers of data. 
  📝 ![MQTT Pub/sub](https://www.hivemq.com/img/blog/mqtt-publish-subscribe.svg)
- 📌 The MQTT commands are CONNECT, SUBSCRIBE, PUBLISH, UNSUBSCRIBE and DISCONNECT. Topics are the medium of distribution, to which clients can PUBLISH and SUBSCRIBE.  All authorized subscribers to a topic will receive all messages published to the topic. MQTT topics do not have to be pre-defined: client applications can create them simply by publishing or subscribing to them.
  hid:: Cz8ILDNeEe64rVdFAmXSTw
  updated:: 2023-08-05T07:02:30.322397+00:00
- 📌 The WebSocket Protocol was created to allow web applications to conduct two-way communications with web servers, as HTTP cannot do that without awkward workarounds. 
  hid:: JPM_mjNeEe61b9OntRWvlA
  updated:: 2023-08-05T07:03:37.368251+00:00
  📝 [rfc6455 - websockets RFC](https://datatracker.ietf.org/doc/html/rfc6455)
- 📌 Using WebSockets to Tunnel MQTT 
  hid:: dADvdDNeEe6UQJMoSSfdJw
  updated:: 2023-08-05T07:05:26.096035+00:00
  📝 There is a lot of IT infrastructure geared towards securing HTTP connections while shutting out unwanted TCP traffic. Many organizations are reluctant to open the MQTT standard ports (1883 and 8883) to allow MQTT applications to communicate with the intra-organization infrastructure
- 📌 he WebSocket protocol allows MQTT communications to use the already existing HTTP facilities: TCP port 80, firewalls, proxies and so on
  hid:: eAxc3jNeEe6Vn8fACU8H3A
  updated:: 2023-08-05T07:05:32.875572+00:00