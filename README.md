# Vigibot-Single-ESP32

- if you want to control multiple ESP32 robots/gadgets, use this guide instead:
https://github.com/efeuentertainment/Vigibot-Multiple-ESP32

- if you want to control a single ESP32 robot/gadget, continue.

This code allows control of ESP32 robots or gadgets over WLAN using vigibot.
A raspberry is still required for the camera and needs to be on the same network. the serial data usually sent and received to an Arduino over UART ( https://www.robot-ma...cation-example/ ), is being sent over WLAN instead, allowing control of small robots or gadgets.
usage example: a pi with a camera watches an arena where multiple small ESP32 robots can be driven, the ESP32 bots do not have an on-board camera.
second usage example: open or close a door or toy controlled fron an ESP32.

1) insert your network info and upload the .ino sketch to your ESP32 from the arduino IDE using an usb cable.

2) open the serial monitor to confirm a successful WLAN connection.

3) the pi needs to create a virtual serial point that's being forwarded over WLAN to the ESP32. install socat on your raspberry:
`sudo apt install socat`

4) create a new script:
`sudo nano /usr/local/socat-esp.sh`

5) paste the following code and replace your ESP's IP:
```
#!/bin/bash

sudo socat pty,link=/dev/ttyVigi0,rawer,shut-none pty,link=/dev/ttyEsp0,rawer,shut-none &

while true
do
  sleep 1
  sudo socat -T15 open:/dev/ttyEsp0,raw,nonblock tcp:192.168.1.64:7070
  date
  echo "restarting socat"
done
```

6) add permissions
`sudo chmod +x /usr/local/socat-esp.sh`

7) run the script
`sudo /usr/local/socat-esp.sh`

8) test the pi <-> ESP connection by logging in as root in a new shell (needs root) :
`su -`
and sending a test text frame:
`echo '$T  hello from cli       $n' > /dev/ttyVigi0`
the arduino serial monitor now says "hello from cli "

9) if it works add it to start at boot:
`sudo nano /etc/rc.local`
and add
`/usr/local/socat-esp.sh &`
above the line "exit 0".

10) add a new entry on the vigibot online remote control config -> SERIALPORTS -> 2 with value: "/dev/ttyVigi0"

11) add a new entry on the vigibot online remote control config -> SERIALRATES -> 2 with value: "115200"

12) set WRITEUSERDEVICE to "2"

13) set READUSERDEVICE to "2"

restart the client and wake your robot up, if it wakes up then everything works.
