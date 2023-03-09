# IoT-LoRaWAN-Exercise
Last time you either acted as a sensor or a server exchanging messages or commands with each other.
This time around we will do a repeat but you will all be sensors and instead of Wi-Fi you will use LoRaWAN. 

Your friendly TA will act as the server this time around and in addition to being the server he also brings a LoRaWAN gateway that the server is connected to.

The server software listens for messages and in reply to each message it asks for a new piece of information. Either `co2`, `humidity` or `temperature`.

The goal is then for you to connect your devices through the gateway to the server, and send the measurements it asks for, to it. The data is expected to be sent as a ascii-encoded strings.

Roughly the steps you are going to follow are to:
1. get your devices up and running again and measuring air quality
2. Figure out how to connect to a LoRaWAN network stack. For you to connect to it,The TA needs some information, and so do you. which?
3. Tell him that information.
4. start sending measurements over LoRaWAN
5. listen for responses
6. measure the sensor that has been asked for

The Pycom website provides some nice documentation on how to use LoRaWAN on their devices. Before enabling LoRaWAN it is important that you remember to connect the external antenna! if not you might damage your device. The correct port for the antenna is the one to the right of the big LED. 


## Food for thought
> - How would you compare LoRaWAN and Wi-Fi? In terms of security, bandwidth?
>- You are communicating with the server over ascii. That's not optimal, since it wastes a lot of data. Why? How else could you do it?

## Reading from the scd30 sensor
below is the code needed in main.py to read from the sensor. The code has graciously been provided by your fellow student Theodor, and only been adapted slightly here.
Remember this is written for Wi-Fi. LoRaWAN is slightly different so it will require some adaptation

```python3
from network import WLAN
from machine import I2C
import time
from scd30 import SCD30
import socket

# constants for the wifi connection and sensor
i2c = I2C(2)
scd30 = SCD30(i2c, 0x61)
wlan = WLAN(mode=WLAN.STA)

# Setup for WiFi connection
wlan.connect(ssid='name', auth=(WLAN.WPA2, 'password'))
while not wlan.isconnected():
    machine.idle()
    time.sleep_ms(1000)
    print("Connecting to WiFi...")
print("WiFi connected succesfully")
print(wlan.ifconfig())

time.sleep(1)

# setup socket for connection
s = socket.socket()
host = '192.168.4.1'
addr = socket.getaddrinfo(host,1234)[0][-1]
s.connect(addr)
print('socket connected')

while True:
    received = s.recv(10000)
    print('received: ', received)

    if (len(received) > 0):
        
        # Wait for sensor data to be ready to read (by default every 2 seconds)
        while scd30.get_status_ready() != 1:
            time.sleep_ms(500)
        ans = scd30.read_measurement()
        
        # convert the answers to byte
        if(received == b'humidity'):
            s.send(str.encode(str(ans[2])))
        elif(received == b'temperature'):   
            s.send(str.encode(str(ans[1])))
        elif(received == b'co2'):
            s.send(str.encode(str(ans[0])))
```

for this to work you also need the scd30 library in a `lib/` folder in your project. That can be found here https://github.com/agners/micropython-scd30/blob/master/scd30.py
