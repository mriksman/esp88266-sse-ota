# ESP8266 HTTP Server Side Events and Minimal OTA
Uses BSD Sockets API to listen for incoming connections

Accepts `text/event-stream` from client and sends `event:progress` updates to the client.

Uses `esp_log_set_putchar` to send console log to HTTP client using SSE

Accepts POST for bin file upload. Intended for ESP8266, so it checks the entry address in the first 8 bytes received. Espressif expect you to;
* Have OTA partition at `0x10000` and one at `0x110000` (for > 1MB flash sizes)
* If it's < 1MB, then use `make ota` which will compile the app bound for app1 with the modified entry address.
* But they haven't modified the `CMakeLists.txt` file to do the same.

Otherwise, you get this error;
```
esp_ota_ops: **Important**: The OTA binary link data is error, please refer to document <<ESP8266_RTOS_SDK/examples/system/ota/README.md>> for how to generate OTA binaries
```
In order to use OTA on 1MB ESP8266, replace lines 96,97 in `components/esp8266/CMakeLists.txt` from
```
partition_table_get_partition_info(app_offset "--partition-boot-default" "offset")
partition_table_get_partition_info(app_size "--partition-boot-default" "size")
```
to
```
if (DEFINED PARTITION_NAME)
   partition_table_get_partition_info(app_offset "--partition-name ${PARTITION_NAME}" "offset")
   partition_table_get_partition_info(app_size "--partition-name ${PARTITION_NAME}" "size")
else()
   partition_table_get_partition_info(app_offset "--partition-boot-default" "offset")
   partition_table_get_partition_info(app_size "--partition-boot-default" "size")
endif()
message(STATUS "Using partition located at offset " ${app_offset} " with size " ${app_size})
```
And in your project's **main** `CmakeLists.txt` file (the program that will end up in `OTA_1`) add the following
```
set(PARTITION_NAME app1)
```
where `app1` is the label of your `OTA_1` partition defined in your partitions csv.

ESP App Description
--------------------
This code will check the first 8 bytes received to ensure
1. It has a valid Magic Byte of `0xE9` as the first byte
2. It has the correct entry address (as described above)

In order for this to work, ESP8266 RTOS SDK had the modifications made in commit `525c34b`. However, the linker script has the `.rodata .desc` being saved in the wrong memory area. For the code to work, the `esp_app_desc` data must be shifted into the `.text` section.

Open file `components/esp8266/ld/esp8266.project.ld.in` and move the following lines from 200 to line 168;
```
*(.rodata_desc .rodata_desc.*)
*(.rodata_custom_desc .rodata_custom_desc.*) 
```

