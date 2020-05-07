# ESP8266 HTTP Server Side Events
Uses BSD Sockets API to listen for incoming connections

Accepts `text/event-stream` from client

Uses `esp_log_set_putchar` to send console log to HTTP client using SSE
