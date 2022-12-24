# RaspberryPi-CAN-DAQ-MVP
A minimum viable product (MVP) to serve as a proof of concept for the intended WFR 2023 data acquisition system (DAQ) 

## Raspberry Pi setup
### Hardware
This project has been tested to work with a MCP 2515 based CAN Bus board, which connects to SPI0 on the Raspberry Pi. I used a Raspberry Pi 3B V1.2, but it should work on any 3 and above type RPi models, (although for better compatability with InfluxDB, a RPi 4 series is recommended but more on that later)

<img src="https://user-images.githubusercontent.com/25854486/209448905-cbbaac77-50bf-4a75-8132-85bc5a5c5922.png" width="200">


The pin connection is as follows: 


| MCP2515 Module| Raspberry Pi Pin #| Pin Description  |
| ------------- |:---------------------:| :-----:|
| VCC           | pin 2                 |5V (it's better to use external 5V power)|
| GND           | pin 6                 |   GND |
| SI            | pin 19                |    GPIO 10 (SPI0_MOSI)|
| SO            | pin 21                |    GPIO 9 (SPI0_MISO)|
| SCK           | pin 23                |    GPIO 11 (SPI0_SCLK)|
| INT           | pin 22                |    GPIO 25 |
| CS            | pin 24                |    GPIO 8 (SPI0_CE0_N)|

### Drivers
#### SocketCAN 
This project heavily relies on [SocketCAN](https://docs.kernel.org/networking/can.html) which from my understanding is the Linux kernels default CAN Bus interface implementation, that aims to treat CAN Bus interface devices similar to regular network devices and mimic the TCP/IP protocol to make it easier to use. It also natively supports MCP2515 based controllers. Since this is built in, nothing needs to be installed, however the `/boot/config.text` needs to have the following line appended to it:
```
dtoverlay=mcp2515-can0,oscillator=8000000,interrupt=25 
```
This gets the Linux kernel to automatically discover the CAN Controller on the SPI interface. If your interface pin has a different oscilator frequency, you can change that here. 
Now reboot the Pi and check the kernel messsages (you can bring this up in the terminal by using the command: `dmesg | grep spi)` or `dmesg` to view all kernal messages), and you should see the following:
```
[    8.050044] mcp251x spi0.0 can0: MCP2515 successfully initialized.
```
Finally, to enable the CAN Interface run the following in the terminal:
```
 sudo /sbin/ip link set can0 up type can bitrate 500000 
```
If you are using a different bitrate on your CAN Bus, you can change the value. You will need to run this command after every reboot, however you can set it to run automatically on startup by appending the following to the `/etc/netwrok/interfaces` file:
```
auto can0
iface can0 inet manual
    pre-up /sbin/ip link set can0 type can bitrate 500000 triple-sampling on restart-ms 100
    up /sbin/ifconfig can0 up
    down /sbin/ifconfig can0 down
```
And that's everything to get the CAN Bus interface working! Since SocketCAN acts like a regular network device, we can get some statistic information by using the `ifconfig` terminal command. 
#### CAN-utils
[CAN-utils](https://github.com/linux-can/can-utils#basic-tools-to-display-record-generate-and-replay-can-traffic) is a set of super useful CAN debugging tools for the SocketCAN inteface. You can install it using the following command:
```
sudo apt-get install can-utils 
``` 
This allows us to dump CAN Bus traffic to logs, send test messages and simulate random traffic. Please see the readme on their github page for more details.



