# eps32bluetooth-scanner

I wanted to see if an ESP32 could find user-given names of advertising
devcies (mainly, phones) on classic Bluetooth (not BLE). I knew nothing about this, except
that for $8 on eBay, the ESP32 had BT and BLE functionality.  Plus I did
a project a while back using the ESP8622. So I bought 3 of them, and went to work. (I was trying
to build a digital contact tracing system to help get us through the Covid-19 crisis using the ESP32).

The install procedure [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/) was flawless for macOS.

I did a bunch of searching on my way to develop the code for the ESP32, and there seems to be a lot of
people wanting to probe BT device names, with few solutions. I was able to modify the stock ESP32 bt_discovery.c code [found here](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/classic_bt/bt_discovery/main/bt_discovery.c) to make this work. Here's how.

The base code in bt_discovery.c basically does it all, but did not include calls to probe a discovered devices's name, once a device chimes in as being present.  The modifications were minor. 

First modify this case block as follows:

```c
case ESP_BT_GAP_DISC_RES_EVT: {
        printf("ESP_BT_GAP_DISC_RES_EVT triggered.\n");
        update_device_info(param);
        //param->disc_res.bda is a 6 byte binary version of the bt device id
        esp_bt_gap_read_remote_name(param->disc_res.bda); // yay! this works!!
        break;
```

You don't need the `printf`, and as outlined [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/bluetooth/esp_gap_bt.html), this case is triggered in the call-back function when the Bluetooth device has responded to the `esp_err_tesp_bt_gap_start_discovery()` call (which performs the BT discovery process).

The `update_device_info(param);' line uses values filled into the `param` variable [outlined here](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h).  The [A2DP union](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L226) was a clue on getting this to work.  If you look at it, but
