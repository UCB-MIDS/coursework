### Event driven processing for the Internet of Things
This lab builds upon the homework and introduces some of the tools in use today 
for Internet of Things processing

####Getting the MQTT broker running
Let us reconnect to the VM we created for the homework.  We installed mosquitto clients but this time, let
us install the mosquitto broker as well
```
apt-get install mosquitto
```
Notice that mosquitto clients and the broker itself are extremely lightweight, the total install size is just a few MB.
Now, let us start mosquitto in daemon mode
```
mosquitto -d
```
The default MQTT port is 1883. If you prefer to change this port, just use the -p option in your command lines throughout.
Let us set up a simple listener on our newly created MQTT broker:
```
mosquitto_sub -t test -h localhost -p 1883 &
```
This command will run in the background and print the messages received on the topic called "test".
Now, let us publish a message to this topic:
```
mosquitto_pub -t test -h localhost -p 1885 -m "hello world"
```
You should see the message printed below, e.g.
```
1464024282: New connection from 127.0.0.1.
1464024282: New client connected from 127.0.0.1 as mosq_pub_5063_slapi.
hello world
```
As you can see, it is incredibly easy to get started with MQTT. The tools are lightweight and easy to understand. Mosquitto can 
process thousands of messages per second; and if you need more scalability, you could use a HiveMQ or VerneMQ cluster.

Now, let us create a test endpoint that will imitate a temperature sensor. Create a file called temp_sensor.sh and add these lines:
```
#!/bin/bash

while [ true ]
do
  temp=10
  echo {"temp": $temp}
  sleep 0.3
done
```
Don't forget to make the file runnable
```
chmod a+x temp_sensor.sh
```
mosquitto_pub will terminate its connection to the broker after each message is published unless you ask it not to.  
The -l option below will keep the connection open:
```
./temp_sensor.sh | mosquitto_pub -l -h localhost -p 1883 -t test
```
You should see our imitated sensor sending data!  Once you have verified that everything works, Control-C the script.

Let us also kill the background listener:
```
fg
Control-C
```
**Quick Assignment:** modify the above script so that it emits the temperature of 20 with the probability of about 10% . Verify
that it works.

####Connecting to the IBM Internet of Things Foundation
It is totally fine to play with a standalone MQTT bus, but we want jetpacks!  Let's connect to IBM's hosted IOT foundation 
system.  It is fairly representative of the state of the industry in that it provides a hosted MQTT broker 
plus a useful structure for working with devices and gateways.

Esteban plese add stuff here.

#####Adding a new device
Click "add device" and when prompted, create a new device type called "temp_sensor" with no metadata.  then, create a new device
and give it a name.  I named mine "bernie".  At this point, you will be presented with the credentials that you can use to connect to your new device e.g.
```
Organization ID qlsgr4
Device Type temp_sensor
Device ID bernie
Authentication Method token
Authentication Token Zj)Ogh616VfODFdae+
```
go to the devices dash and check on the health of your device:
```
https://yourorg.internetofthings.ibmcloud.com/dashboard/#/devices/browse
```
Now, let us publish a hello universe message to our device input topic.  Let us agree to call our application "temp_filter" - see how it enters the topic we are using:
```
mosquitto_pub -m "hello universe" -h qlsgr4.messaging.internetofthings.ibmcloud.com -p 1883 -t iot-2/evt/temp_filter/fmt/json -u use-token-auth -P "Zj)Ogh616VfODFdae+" -i d:qlsgr4:temp_sensor:bernie
```
**Quick Assignment:** modify the above script to send to this topic in a loop and start it.
Your device now should appear online and you should see its data coming in.

####Creating a NodeRed application
Now, let us create a visual event-driven application that can consume this data and do something with it.
first let us set up our node red environment.

Creating and connecting a Node-RED device simulator
Use Node-RED to create a device simulator and send simulated device data to your {{site.data.keyword.iot_short}} organization.
{:shortdesc}

Node-RED is a tool for wiring together hardware devices, APIs and online services in new and interesting ways. For more information, see the Node-RED web site.

You can run your Node-RED instance in your own environment or use it as a {{site.data.keyword.Bluemix_notm}} application. The following process includes the instructions for {{site.data.keyword.Bluemix_notm}}.

To create and connect the Node-RED device simulator:

Create the Node-RED device simulator Use the device simulator to send MQTT device messages to {{site.data.keyword.iot_short}}. The device simulator simulated sending data for a freight container to an MQTT broker such as {{site.data.keyword.iot_short}}.

Log in to {{site.data.keyword.Bluemix_notm}} at: https://console.ng.bluemix.net
Select the Catalog tab.
Locate the Boilerplates section of the service catalog and click Node-RED Starter Community BETA. Tip: Click here to go directly to the Node-RED Starter page.
On the Node-RED Starter page, select the space where you want to deploy Node-RED, verify the Create an app selections, and click Create to add Node-RED to your Bluemix organization.
For example:
Space: dev
Name: myDevice
Host: myDevice
Leave the rest of the options at their default values. After the application is deployed, you are brought to the Start coding with Node-RED page.
Note: The staging process might take a few minutes.

Click the app link at the top of the page to open Node-RED.
Example: http://simulatedDevice.mybluemix.net
Click Go to your Node-RED flow editor to open the editor.
Copy the Node-RED flow data that you find in the Node-RED node flow data section of this document.
In the Node-RED flow editor, click the menu in the upper right corner and select Import > Clipboard.
Paste the clipboard into the import nodes input field and and click Ok. The device simulator flow is imported to the flow editor.
Register your device with {{site.data.keyword.iot_short}}
Follow these steps to connect the the Node-RED sample device:

In {{site.data.keyword.Bluemix_notm}}, go to Dashboard
Select the space in which you deployed {{site.data.keyword.iot_short}}.
Click the {{site.data.keyword.iot_short}} tile.
Click Launch dashboard to open the {{site.data.keyword.iot_short}} dashboard.
Select Devices
Click Add Device
Click Create device type.
Enter a descriptive name and description for the device type, such as sample_device.
Optional: Enter device type attributes.
Optional: Enter device type meta data.
Click Create to add the new device type.
Click Next to add your device.
Enter a device ID such as Device001.
Optional: Enter device meta data.
Click Next to add a device connection with an auto-generated authentication token.
Verify that the summary information is correct, and then click Add to add the connection.
In the device information page that opens, copy and save the device information:
Organization ID
Device Type
Device ID
Authentication Method
Authentication Token
Tip: You will need Organization ID, Authentication Token, Device Type, and Device ID in the next few steps to finalize the configuration of the Node-RED application to complete the connection.
Connect your device to {{site.data.keyword.iot_short}}

Open the Node-RED flow editor.
Double-click the Publish to IoT node.
Verify that the topic entry is: iot-2/evt/event_name/fmt/json Tip: The /event_name part of the topic sets the event name for the published events.
Click the edit icon next to the Server field.
Update the Server name to correspond to your {{site.data.keyword.iot_short}} organization.
{organization_ID}.messaging.internetofthings.ibmcloud.com
Where {organization_ID} is the Organization ID that you saved earlier. 6. Update the Client ID with your organization ID, device type, and device ID:

d:{organization_ID}:{device_type}:{device_ID}
Where {organization_ID}, {device_type}, and {device_ID} where the values that you saved earlier. 7. Click the Security tab. 8. Verify that the Username field has the value use-token-auth. 9. Update the Password field with the Authentication Token that you saved earlier. 10. Click Update and then OK. 12. In the upper right corner of the Node-RED flow editor, click Deploy. 13. Verify that the Publish to IoT node connection status displays connected. Tip: If the connection is not made, double check the settings that you entered. Even a small cut and paste error will cause the connection to fail.

Validate the device connection

In another browser tab or window, open the {{site.data.keyword.iot_short}} dashboard.
Select Devices and click Device001 or the name of the device that you added, if different.
The device information page opens. This view lets you see the connection status for your device. The device should show as disconnected at this stage.
Back in your Node-RED flow editor, click the button on the Inject node to generate an asset payload.
The payload contains the following data points:
{"d":
{ "name":"My Device",
  "location":
  {
    "longitude":-87.90,
    "latitude":43.03
  },
  "velocity":13,
  "type":"GPS"
}
}
{:codeblock}

In the debug tab of the right pane, verify that messages are created.
Back in the {{site.data.keyword.iot_short}} device information page, the device should now be connected. Verify that you see the same data points received from the device.
Event  Datapoint    Value  Time Received
event_name d.name  My Device   May 16, 2016 2:22:18 PM
event_name d.location.longitude    -87.90  May 16, 2016 2:22:18 PM
event_name d.location.latitude 43.03   May 16, 2016 2:22:18 PM
event_name d.velocity  13  May 16, 2016 2:22:18 PM
event_name d.type  GPS May 16, 2016 2:22:18 PM
{:codeblock}

You have now connected your sample IoT device to {{site.data.keyword.iot_short}} and can see device data.

Node-RED node flow data

{: #flow_data}
Copy the text below to your clipboard and then paste it into the import clipboard when importing nodes in the Node-RED flow editor.
Important: Make sure that you copy all the text including the leading and trailing square brackets.

[{"id":"e658d1b6.c187e8","type":"mqtt-broker","z":"965c3822.02c558","broker":"organization_ID.messaging.internetofthings.ibmcloud.com","port":"1883","clientid":"d:organization_ID:device_type:device_ID","usetls":false,"verifyservercert":true,"compatmode":true,"keepalive":"60","cleansession":true,"willTopic":"","willQos":"0","willRetain":"false","willPayload":"","birthTopic":"","birthQos":"0","birthRetain":"false","birthPayload":""},{"id":"23f82ce8.126dc4","type":"mqtt out","z":"965c3822.02c558","name":"Publish to IoT","topic":"iot-2/evt/event_name/fmt/json","qos":"","retain":"","broker":"e658d1b6.c187e8","x":480,"y":273,"wires":[]},{"id":"92e2f51f.5126","type":"inject","z":"965c3822.02c558","name":"Send data","topic":"","payload":"","payloadType":"num","repeat":"","crontab":"","once":false,"x":92,"y":265,"wires":[["832b025d.2d24c"]]},{"id":"832b025d.2d24c","type":"function","z":"965c3822.02c558","name":"Device payload","func":"var name1 = \"My Device\";\nvar type1 = \"GPS\";\nvar longitude1 = [-98.49,-97.74,-96.79,-94.20,-90.19,-94.57,-87.62,-87.90,-93.26,-95.99];\nvar latitude1 = [29.42,30.26,32.77,36.37,38.62,39.09,41.87,43.03,44.97,41.25];\nvar velocity1 = context.get('velocity1')||0;\n\n\n\nvelocity1 = velocity1+1;\nif(velocity1 > 20) velocity1=0;\ncontext.set('velocity1',velocity1);\n\nvar counter1 = context.get('counter1')||0;\ncounter1 = counter1+1;\nif(counter1 > 9) counter1 = 0;\ncontext.set('counter1',counter1);\n\nmsg = {\n \tpayload: JSON.stringify(\n \t { d:{\n \"name\" : name1,\n \"location\" : \n {\n \"longitude\" : longitude1[counter1],\n \t \"latitude\" : latitude1[counter1]\n },\n \"velocity\" : velocity1,\n \t \"type\" : type1\n \t }\n \t }\n )\n}\n\nreturn msg;","outputs":1,"noerr":0,"x":270,"y":108,"wires":[["f04ddebd.f55298","23f82ce8.126dc4"]]},{"id":"f04ddebd.f55298","type":"debug","z":"965c3822.02c558","name":"","active":true,"console":"false","complete":"false","x":250,"y":384,"wires":[]},{"id":"d606058f.d3422","type":"comment","z":"965c3822.02c558","name":"Configure Publish to IoT for your environment","info":"1. Double click the node.\n2. Click the edit icon next to the Server field.\n3. Update the Server name to correspond to your Watson IoT Platform organization.\n `organization_ID.messaging.internetofthings.ibmcloud.com`\n\n4. Update the Client ID with your orgainzaiton ID, device type, and device ID:\n `d:organization_ID:device_type:device_ID`\n\n5. Click the Security tab.\n6. Verify that the Username field has the value use-auth-token.\n7. Update the Password field with the Authentication Token that you saved earlier.\n8. Click Update and then OK.\n9. In the upper right corner of the Node-RED flow editor, click Deploy. ","x":570,"y":241,"wires":[]},{"id":"b997cfe2.ebfd7","type":"comment","z":"965c3822.02c558","name":"Function Node","info":"The Function node allows you to pass each message though a JavaScript function.\n\nThe Javascript function creates the message payload that is sent to your Watson IoT Platform organization.","x":271,"y":77,"wires":[]},{"id":"1ce4e378.c5a565","type":"comment","z":"965c3822.02c558","name":"Debug Node","info":"The Debug node causes any message to be displayed in the Debug sidebar. By default, it just displays the payload of the message, but it is possible to display the entire message object.","x":250,"y":414,"wires":[]},{"id":"ba916f5e.695db8","type":"comment","z":"965c3822.02c558","name":"Inject Node","info":"The Inject node allows you to inject messages into a flow, either by clicking the button on the node, or setting a time interval between injects.","x":75,"y":236,"wires":[]}] {:codeblock}