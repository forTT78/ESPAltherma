# mDNS responder with a NULL check (Arduino core 2.x only)

`mdns.c`, `mdns_private.h` and `mdns_networking.h` are copies of `components/mdns` from
ESP-IDF v4.4.7 (commit 38eeba213a), the IDF that Arduino core 2.0.17 (`espressif32@7.0.1`)
is built from. License: Apache-2.0 (Espressif), see the SPDX header in each file.

Why: the board panicked in the `mdns` task (issue #639):

```
LoadProhibited, excvaddr 0x4
#0 _mdns_remove_scheduled_answer  mdns.c:1324
#1 mdns_parse_packet              mdns.c:3533
```

When known answers from the LAN have removed every answer of a scheduled ("distributed")
packet, `q->answers` is NULL and the next known answer reads `a->type` at address 0x4.
esp-protocols mdns 1.10.0 and later check `if (a)` there; this copy adds the same check and
is otherwise unchanged (changes are marked `ESPAltherma:`).

How it is used: the core links mDNS from the prebuilt `libmdns.a`. The linker only takes an
archive member for symbols still undefined, so with this object in the build it skips
`mdns.c.obj` and still takes `mdns_networking_lwip.c.obj` from the archive. The file compiles
to nothing on core 3.x (`ESP_ARDUINO_VERSION_MAJOR >= 3`), which keeps the core's mdns.

Remove this folder when the project moves to a core whose mdns has the check.
