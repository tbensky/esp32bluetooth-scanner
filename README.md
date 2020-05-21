# ESP32-luetooth-scanner with device name retrieval

I wanted to see if an ESP32 could find user-given names of advertising
devices (mainly, phones) on classic Bluetooth (not BLE). I know nothing about Bluetooth, except
that for $8 on eBay, you can buy a ESP32 that has BT and BLE functionality.  Plus I did
a project a while back using the ESP8622. So I bought 3 ESP32s and went to work. (I was trying
to build a digital contact tracing system to help get us through the Covid-19 crisis using the ESP32).

The install procedure [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/get-started/) worked flawlessly on macOS.

I did a bunch of searching on my way to develop the code for the ESP32, and there seems to be a lot of
people wanting to probe BT device names, with few solutions. I was able to modify the stock ESP32 [bt_discovery.c](https://github.com/espressif/esp-idf/blob/master/examples/bluetooth/bluedroid/classic_bt/bt_discovery/main/bt_discovery.c) code to make this work. Here's how.

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

You don't need the `printf`, and as outlined [here](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/bluetooth/esp_gap_bt.html), this case is triggered in the ``bt_app_gap_cb()`` callback function when the Bluetooth device has responded to the `esp_err_tesp_bt_gap_start_discovery()` call (which performs the BT discovery process).

The `update_device_info(param);` line uses values filled into the [`param` variable](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L339) to display the BT address and RSSI (signal strength).  The [A2DP union](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L226) was a clue on getting this to work.  In particular, [this line](https://github.com/espressif/esp-idf/blob/a352097/components/bt/host/bluedroid/api/include/api/esp_gap_bt_api.h#L336) contains the variable 
```
uint8_t rmt_name[ESP_BT_GAP_MAX_BDNAME_LEN + 1]; /*!< Remote device name */`
``` 

so I knew I was getting close. I then noticed the function [esp_bt_gap_read_remote_name()](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/api-reference/bluetooth/esp_gap_bt.html#_CPPv427esp_bt_gap_read_remote_name13esp_bd_addr_t). Since `ESP_BT_GAP_DISC_RES_EVT` means a device has chimed in, calling this function in its `case` made sense. The parameter `param->disc_res.bda` is a 6-byte binary representation of the devices BT address (something like 803:d7:95:4a:07:1d in ASCII).


To capture the returned name, you have to add this block

```
case ESP_BT_GAP_READ_REMOTE_NAME_EVT:
            printf("ESP_BT_GAP_READ_REMOTE_NAME_EVT triggered. Name=%s\n",param->read_rmt_name.rmt_name);
            break;
```

into the callback ``bt_app_gap_cb()`` ``case`` structure. When this is triggered, the variable `param->read_rmt_name.rmt_name)` will contain the remote device's name `ESP_BT_GAP_READ_REMOTE_NAME_EVT` is the callback event for `esp_bt_gap_read_remote_name()`.

# Results

Works great.  I was bummed though, because an iPhone is only discoverable when it is on, and you are on the Settings->Bluetooth screen. In this case, the ESP32 will find it and display it's name every time (same with an iPad). I was naively hoping that any BT device would chime in to the ESP32 probe for a 



