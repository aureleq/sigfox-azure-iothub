# sigfox - Microsoft Azure integration
This tutorial explains how to communicate between a sigfox device and Microsoft Azure IOT platform, as well as PowerBI for analytics.

## 1. Requirements
- sigfox device registered on the network
- subscription to Microsoft Azure (free trial account is sufficient)
- PowerBI account

## 2. SmartEverything Arduino code (optional)
If you have a SmartEverything board, you can use the arduino code in the repository. Temperature and Humidity will be reported everytime the Reset button is pressed.
Telit library is being used in this example.

## 3. Azure IOT Hub initial setup
Connect to https://portal.azure.com and add a new IOT Hub instance.
Create a new Resource-Group for the setup:
![IOT Hub setup](./Capture0.PNG)

Once the instance has been completed, go into "Settings -> Shared Access Policies":
![IOT Hub shared](./Capture1.PNG)

Copy the Connection String primary key, the string will have to be copied into sigfox backend:
![IOT Hub Key](./Capture2.PNG)

## 4. sigfox callback configuration
Connect to your sigfox backend account and select the device type to be configured.
Under Callbacks category, add a new callback and select Microsoft Azure IOT Hub:
![Callback setup](./Capture13.PNG)

Add the previously copied Azure Connection String:
![Callback setup 2](./Capture12.PNG)

Using the SmartEverything board, humidity and temperature are reported in a string format (2 bytes for each report). You can use the following custom payload and JSON body in the callback configuration:

`humidity::char:2 temperature::char:2`

`{
"device" : "{device}",
"data" : "{data}",
"humidity": "{customData#humidity}",
"temperature": "{customData#temperature}",
"time" : {time},
"duplicate" : {duplicate},
"snr" : {snr},
"station" : "{station}",
"avgSignal" : {avgSnr},
"lat" : {lat},
"lng" : {lng},
"rssi" : {rssi},
"seqNumber" : {seqNumber}
}`

If your callback configuration is correct, the number of monitored messages on your IOT Hub dashboard should increase when the device communicates.

## 5. Configure Stream Analytics
Back on the Azure portal, add a Stream Analytics instance:
![Analytics 1](./Capture4.PNG)

Add a new Input and select your IOT Hub instance as a source. Use default configuration for other fields as below:
![Analytics 2](./Capture5.PNG)

Add a new Output, select Power BI and authorize the connection to your Power BI account:
![Analytics 3](./Capture6.PNG)

Create a new Dataset and Table that will be used to store messages information:
![Analytics 4](./Capture7.PNG)

Finally, back on the Analytics dashboard create a new Query.
The query will take the date from the IOT Hub input and transfer it to our Power BI output. Humidity and Temperature values will also be converted to int and float values (our JSON callback report string values):
![Analytics 5](./Capture8.PNG)

- Query content:
`SELECT
    CAST(humidity as bigint) as hum,
    CAST(temperature as float) as temp,
    System.Timestamp AS Timestamp
INTO
    [DeviceBI]
FROM
    [DeviceInputStream]
`

## 6. Power BI visualization
Connect to your Power BI account (https://app.powerbi.com). Select the streaming dataset "TempHum" and edit it to define "hum" and "temp" as numbers.
Note that Power BI needs to receive at least one message to display the dataset:
![Power BI](./Capture9.PNG)

You can now create graphs from your dataset. Select a new Line Graph, simply drag and drop the timestamp variable on the Axis category and humidity/temperature variables to Values:
![Power BI 2](./Capture10.PNG)

Once you are done with your graph presentation, you can save it and display it in the Reports and Dashboards sections:
![Power BI 3](./Capture11.PNG)
