# eps32bluetooth-scanner

I wanted to see if an ESP32 could find user-given names of advertising
phones on classic Bluetooth (not BLE). I did a bunch of searching on my
way to develop the code for the ESP32, and there seems to be a lot of
people wanting to probe the device names, with few solutions. (I was trying
to build a digital contact tracing system for Covid-19 using the ESP32).

I was able to modify the stock ESP32 bt_discovery.c code [found here](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/classic_bt/bt_discovery/main/bt_discovery.c) to make this work.

This repo is the code that does it and notes to myself on how I got it all to work. The base code in bt_discovery.c
basically does it all, but did not include calls to probe a discovered phone's name.  The modifications were
minor":

```c
case ESP_BT_GAP_DISC_RES_EVT: {
        printf("ESP_BT_GAP_DISC_RES_EVT triggered.\n");
        update_device_info(param);
        //param->disc_res.bda is a 6 byte binary version of the bt device id
        esp_bt_gap_read_remote_name(param->disc_res.bda); // yay! this works!!
        break;
```
