# ESP8266 HTTP Server Side Events and Minimal OTA
Uses BSD Sockets API to listen for incoming connections

Accepts `text/event-stream` from client

Uses `esp_log_set_putchar` to send console log to HTTP client using SSE

Accepts POST for bin file upload. Intended for ESP8266, so it checks the entry address in the first 8 bytes received. Espressif expect you to;
* Have OTA partition at `0x10000` and one at `0x110000` (for > 1MB flash sizes)
* If it's < 1MB, then use `make ota` which will compile the app bound for app1 with the modified entry address.
* But they haven't modified the `CMakeLists.txt` file to do the same.

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
And in your project's `CmakeLists.txt` file add the following
```
set(PARTITION_NAME app1)
```
