# eps32bluetooth-scanner

I wanted to see if an ESP32 could find user-given names of advertising
phones on classic Bluetooth (not BLE). I did a bunch of searching on my
way to develop the code for the ESP32, and there seems to be a lot of
people wanting to probe the device names, with few solutions.

I was able to modify the stock ESP32 bt_discover.c code [found here](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/classic_bt/bt_discovery/main/bt_discovery.c) to make this work.

This repo is the code that does it and notes to myself on how I got it all to work. 
