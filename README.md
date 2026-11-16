# NaWFstore

English | [Русский](README.ru.md)

NaWFstore manages Wi-Fi profiles, station reconnects and optional SoftAP sessions. The portable C++20 core uses injected storage, radio and clock interfaces. The ESP-IDF port supplies NVS, Wi-Fi and a monotonic clock. The transport-independent console parser accepts character input. Applications own their UI, task scheduling and provisioning transport. Licensed under MIT.

## Integration

For PlatformIO with `framework = espidf`, add a `lib_deps` entry pointing at this repository and pin the version or Git SHA. A local package can use an absolute `symlink://` path. Register the ESP-IDF SDK requirements `esp_wifi esp_event esp_netif esp_timer nvs_flash` in the consumer component. Use the same compile definitions for the library and the consumer.

For native ESP-IDF, add the checkout to `EXTRA_COMPONENT_DIRS` or use a pinned Git entry in the consumer's `idf_component.yml`. Require the component by the name resolved by CMake. Use one integration route per build.

Initialize NVS and the default ESP event loop in the application, initialize `EspIdfWifiDriver`, then call `Manager::initialize`. Call `poll()` regularly. Serialize manager calls in one task. `status().driver.connected` indicates IP acquisition. Expose only password-free snapshots to logs or remote clients.

## Optional timer and retries

`enable(0)` enables Wi-Fi indefinitely. A positive duration (up to 86400 seconds) applies to that session. A separate SoftAP timeout is set in `ApConfig`. The current station timeout invokes the same shutdown as `disable()`, stopping both STA and SoftAP.

In the default `StartupPolicy::application` mode, the application passes its boot defaults in `Config`. Defaults enable Wi-Fi without a time limit. `set_startup` updates the live configuration in this mode. Persist it separately in the application if desired. Profile changes preserve any pre-existing NWF1 startup fields.

A single-product firmware can explicitly select `StartupPolicy::stored` to load and save the startup setting alongside the profiles. MAIN uses this mode for compatibility with existing settings. When several firmwares share a board, each can choose whether to honor those settings.

Enabled profiles are tried in descending priority, preserving order for ties. A working link is kept. `ConnectionTiming` can override the default attempt timeout (20 s), delay between profiles (250 ms) and delay after a cycle (15 s).

## Persistent data

- Default partition and namespace are `nvs` and `nawfstore`. Keys `cfg0` and `cfg1` alternate CRC-checked `NWF1` generations. The previous valid generation remains available. Factory profiles seed missing storage. An intentionally saved empty list is respected.
- `NAWFSTORE_MAX_PROFILES` defaults to 5 and allows 1–8. It limits additions. All builds can read, use and preserve up to eight existing profiles.
- On the shared 16 MiB LCD7 board preserve NVS at `0x9000`, size `0x6000` (24 KiB). Verify the installed partition table and app slot before upload. On other boards preserve their provisioned layout. The library never erases NVS or mounts SPIFFS.
- `nvs_save_uncertain` means a commit or verification failed and the candidate may already be durable. Restart the firmware to reload from flash before trying another persisted edit.
- The ESP-IDF radio keeps its station configuration in RAM. Credentials in NVS require device-level NVS encryption for protection at rest. Keep real credentials out of Git and ordinary diagnostics.

Other build switches are `NAWFSTORE_WITH_NVS`, `NAWFSTORE_WITH_CONSOLE`, `NAWFSTORE_WITH_SOFTAP` and `NAWFSTORE_WITH_BOOTSTRAP`. `NAWFSTORE_WITH_WEB` is a compatibility define for application adapters. Tests build from `tests/` with CMake and CTest.
