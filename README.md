### Foreword
The review was supposed to be submitted by January of 2024. Due to some issue with the [kit](https://community.element14.com/products/devtools/avnetboardscommunity/avnetboard-forums/f/rasynboard/54224/rasynboard-stopped-working), I had to wait for the replacement kit to arrive. Later I switched my job which required relocation, losing track of where the kit was and finally I found the time to complete it. **I would like to apologize to the community for the delayed submission of the review**.<br> 

### Board Description
Usage of Renesas controller [RA6M4](https://www.renesas.com/en/products/microcontrollers-microprocessors/ra-cortex-m-mcus/ra6m4-200mhz-arm-cortex-m33-trustzone-high-integration-ethernet-and-octaspi) is an odd choice considering the lack of resources to learn how to program Renesas microcontrollers. Luckily we don't need to as we can modify the template application provided by the Avnet.
The audio data is collected through [TDK T5838](https://invensense.tdk.com/products/digital/t5838/) and the motion data is acquired through IMU TDK [ICM-42670-P](https://invensense.tdk.com/products/motion-tracking/6-axis/icm-42670-p/)
The heavy lift task of processing the data and producing the inference result is assigned to [NDP120](https://www.syntiant.com/ndp120) which is also present in [Nicla](https://store-usa.arduino.cc/products/nicla-voice). 
[DA1660](https://www.renesas.com/en/products/wireless-connectivity/wi-fi/low-power-wi-fi/da16600mod-ultra-low-power-wi-fi-bluetooth-low-energy-combo-modules-battery-powered-iot-devices) acts the WiFi/Ble modem for the board connected using UART lines.<br>

![image](https://github.com/user-attachments/assets/63a82d4d-3572-4c44-b28c-88aa9c8ca0e8)
The board is really compact, here is the side by side comparison of Rasynboard with carrier and a SDcard adapter. <br>

### Performance
The Renesas controller does a good job of co-ordinating with NDP, sensors, wireless module and SDcard. Data transmission between these nodes happen seamlessly. Log can be viewed either through the UART line or though USB C connection.<br>

### Neural Engine
The Syntiant NDP120 Neural Decision Processor is a specialized chip for always-on applications in battery-powered devices, featuring the Syntiant Core 2 neural processor for running multiple Deep Neural Networks (DNN) with 25x the tensor throughput of its predecessor.<br>

### Wireless connectivity
The Renesas DA16600MOD is an ultra-low-power combo module integrating Wi-Fi and Bluetooth Low Energy (BLE) for battery-powered IoT devices. It features Renesas' VirtualZero™ technology for minimal power usage in sleep mode, supports year-plus battery life, and includes built-in Wi-Fi and BLE SoCs, 4MB Flash memory, and various timing components.<br>

### Storage
Rasynboard supports SDcard so that you can modify the config file, load newer models instead of flashing the board everytime. USB-C connection also exposes the SDcard storage in laptop so that we can modify the config and other required files on the go. On powering off and on, the board will read the config and update if changes are made. <br>

### Serial Connection
Rasynboard allows two ways to read the serial connection. Either by using USB connection or using the UART lines which requires USB-UART converter. Compared to USB, UART offers a better way to read serial messages as the UART gets initialized before the USB connection is initialized. If you are using wireless connections, it is better to go with UART as it prints more debug data.<br>

# Demos
Just before the original kit stopped responding, I made two blogs and posted. 
| Title | Description |
| ----------- | ----------- |
| [RAsynboard - Keyword Spotting - Out of box testing](https://community.element14.com/products/roadtest/b/blog/posts/rasynboard-_2d00_-keyword-spotting-_2d00_-out-of-box-testing) | Blog that contains the hardware, software requirement and briefly explains how to flash the board |
| [RAsynboard - Keyword Spotting - Out of box wireless edition](https://community.element14.com/products/roadtest/b/blog/posts/rasynboard---keyword-spotting---out-of-box-wireless-edition) | Blog contains how to enable the BLE client and view the inference details in laptop using the inbuilt ble server |
| Roadtest review | **In this review we will see how to use the WiFi module and send the inference data to AWS IoT console.** |

## Creating a thing in AWS IOT portal
You can follow either of the two methods to create the thing <br>
1.[Offcial AWS Document](https://docs.aws.amazon.com/iot/latest/developerguide/iot-moisture-create-thing.html)<br>
2.[Official Rasynboard guide](https://github.com/Avnet/RASynBoard-Out-of-Box-Demo/blob/rasynboard_v2_tiny/docs/awsIoTCore.md)<br>

### Modifying the configuration file
Select the Mode as 1 to enable the audio keyword matching models.<br>
```
Mode=1
```
Set the debug port as per your wish. (1 - UART 2- USB)<br>
```
Port=1 
```
Set the below options to 0 or 1 to disable or enable them<br>
```
Write_to_file=0        # write data to a *.csv file on the sdcard
Print_to_terminal=0    # output data to the serial debug terminal
```
Add your wifi configurations here<br>
```
Access_Point=abcdxyz
Access_Point_Password=12345678
```
Set the value to 1 to use the above configured password<br>
```
Use_Config_AP_Details=1
```
Since we are going to use AWS as our cloud provider, I set the value as 2. (0=No Cloud Connectivity, #  1=Avnet's IoTConnect on AWS,#  2=AWS)<br>
```
Target_Cloud=2
```
Enter the endpoint of MQTT server, pub/sub topic names and device name. For more details [click here](https://github.com/Avnet/RASynBoard-Out-of-Box-Demo/blob/rasynboard_v2_tiny/docs/AWSIoTCore.md)<br>
```
Endpoint=abcdefgh-abc.iot.ap-locale-n.amazonaws.com
Device_Unique_ID=rasynboard
MQTT_Pub_Topic=publish
MQTT_Sub_Topic=subscribe
```
Set the cert location as 1 so that the board loads the certificates from SDcard.
```
Cert_Location=1
```
Set the location of the certificate files
```
Root_CA_Filename=certs/AmazonRootCA1.pem
Device_Cert_Filename=certs/rasyn-certificate.pem.crt
Device_Cert_Filename=certs/rasyn-private.pem.key
```
The certificate might requires renaming as above format. Once all these changes are made, save the file.

### Testing out premade model
Once the models are loaded, it will take some time for the wifi credentials to be loaded and network connection to AWS IoT portal to establish. Once done use the predefined keywords "up", "down", "next" and "back". 
![Screenshot 2024-11-25 204536](https://github.com/user-attachments/assets/1f3ea49b-f641-4da2-98e3-56c02cd81397)

### Monitoring the inference result
You can observe the inference result in AWS IoT management console using this link. Make sure to replace the region with your region. <br>
```
https://ap-luck-n.console.aws.amazon.com/iot/home?region=ap-luck-n#/dashboard
```

![Screenshot 2024-11-25 204622](https://github.com/user-attachments/assets/c5fb5e4b-1ec7-4c34-92d1-7241078173fd)

### Sending custom message from cloud
Rasynboard supports processing data from cloud as well, from the monitor, create a topic called "subscribe" and send any message. It will be received the Rasynboard and displayed on serial monitor. <br>
![Screenshot 2024-11-25 204501](https://github.com/user-attachments/assets/2566551c-5374-4099-a63d-3dc70fa1cd43)
for example, I tried sending a json payload
```
{
  "message":"Hello from AWS IoT console"
}
```
on clicking publish, you can see the message received on Rasnboard as well.
![Screenshot 2024-11-25 204519](https://github.com/user-attachments/assets/adf1d914-0f92-41cb-86b3-ca54fcf757cf)


### Pros
Kit has enough sensors to test with and variety of connectivity options to send data to wherever we want. <br>
Smaller form factor, we can use it for wearables. <br>

### Cons
It is inconvenient to use UART for debugging and USB for flashing & powering up the device. <br>
Unfortunately, Rasynboard doesnt show up in Edge impulse boards. So creating custom model was not possible. <br>
Hard to fix the board if it stops responding. <br>

