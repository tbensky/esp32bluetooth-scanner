# ESP32 Bluetooth scanner with device name retrieval

I wanted to see if an ESP32 could find user-given names of advertising Bluetooth
devices (mainly phones) using classic Bluetooth (not BLE). I know nothing about Bluetooth, except
that for $8 on eBay, you can buy a ESP32 that has BT and BLE functionality.  Plus I did
a project a while back using the ESP8622. So I bought 3 ESP32s and went to work. (I was trying
to build a digital contact tracing system to help get us through the Covid-19 crisis using the ESP32).

The install procedure [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/) worked flawlessly on macOS.



I did a bunch of searching on my way to develop the code for the ESP32, and there seems to be a lot of
people wanting to probe BT device names, with few solutions, so I am presenting mine here.

 I was able to modify the stock ESP32 [bt_discovery.c](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/classic_bt/bt_discovery/main/bt_discovery.c) code to make device name retrieving work. Here's what I did.

The base code in bt_discovery.c basically does it all (i.e. the Bluetooth scanning), but simply does not include code to probe a discovered devices's name, once it chimes in as being present.  The modifications were minor: 

First modify this case block as follows:

```c
case ESP_BT_GAP_DISC_RES_EVT: {
        update_device_info(param);
        esp_bt_gap_read_remote_name(param->disc_res.bda); // yay! this works!!
        break;
```

As outlined [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/bluetooth/esp_gap_bt.html), the `ESP_BT_GAP_DISC_RES_EVT` event is triggered in the ``bt_app_gap_cb()`` callback function when a Bluetooth device has responded to the `esp_err_tesp_bt_gap_start_discovery()` call (which performs the BT discovery process). This seems to be the "device chiming in event," as a response to the scan by the ESP32.

The `update_device_info(param);` line uses values filled into the [`param` variable](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L339) to display the BT address and RSSI (signal strength) of the device. Could the device's name be far behind?

 The [A2DP union](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L226) was a clue on getting this to work.  In particular, [this line](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L336) contains the variable 
```
uint8_t rmt_name[ESP_BT_GAP_MAX_BDNAME_LEN + 1]; /*!< Remote device name */`
``` 

so I knew I was getting close. I then noticed the function [esp_bt_gap_read_remote_name()](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/bluetooth/esp_gap_bt.html#_CPPv427esp_bt_gap_read_remote_name13esp_bd_addr_t). Since  the `ESP_BT_GAP_DISC_RES_EVT` event means a device has chimed in, calling this function in its `case` made sense (as shown in the code block above). The parameter `param->disc_res.bda` is a 6-byte binary representation of the device's BT address (something like 83:d7:95:4a:07:1d in ASCII), that `esp_bt_gap_read_remote_name()` requires (i.e. find the name of the BT device with the address in `param->disc_res.bda`).


To capture the returned name, you have to add this block

```
case ESP_BT_GAP_READ_REMOTE_NAME_EVT:
            printf("ESP_BT_GAP_READ_REMOTE_NAME_EVT triggered. Name=%s\n",param->read_rmt_name.rmt_name);
            break;
```

into the callback ``bt_app_gap_cb()`` ``case`` structure. When this case triggered, the variable `param->read_rmt_name.rmt_name` will contain the remote device's name. `ESP_BT_GAP_READ_REMOTE_NAME_EVT` is the callback event for `esp_bt_gap_read_remote_name()`.

# Results

Works great, as long as your devices are in discovery mode.  Most small peripherals like headphones, etc. will be found all the time. With phones, not so, as they're not always in discovery mode.  The iPhone and iPad for example, are only discoverable when on, and you are in the Settings>General>Bluetooth menu. In this case, the ESP32 will find it and display its name every time (same with an iPad), but will not if you are not in this menu, or if the device is asleep.

I was naively hoping that any BT device near an ESP32 would chime in to the ESP32 probe for its name. I see now what a security risk that would be, as we don't want people walking around with their phones constantly broadcasting (anything). 

# Contact Tracer?

For a contact tracer, I was thinking of having Person A embed their Covid status into their device name (in some encrypted form), for Person B (with a probing ESP32 on their keychain/in their pocket) to decrypt and possibly alert them about.

## Work Cycle (macOS)
```
. $HOME/esp/esp-idf/export.sh
idf.py set-target esp32

...

idf.py build
idf.py -p /dev/tty.usbserial-0001 flash
(Cntr-] to exit monitor.)
```


