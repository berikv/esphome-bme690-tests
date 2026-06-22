# BME690 BSEC test snippets

These YAML files are minimal snippets for testing the BME690 PR branch. They intentionally leave out board, Wi-Fi,
OTA, logger, API, and I2C bus setup. Add one snippet to an existing working ESPHome config that already defines:

- an ESP32 ESP-IDF target
- an `i2c:` bus with `id: bus_a` or update `i2c_id`
- logger/API/OTA/Wi-Fi as needed for your device

Update `external_components.source` to the branch/ref being tested, then update `bsec_library` and `bsec_config`
to files available on your ESPHome build host. The BSEC library archive and config file must come from the same
Bosch BSEC release and target architecture.

Expected first-cycle result:

- BSEC 3.3.0 LP: `BSEC outputs: 13`
- BSEC 3.3.0 ULP: `BSEC outputs: 12`
- BSEC 3.2.1 LP: `BSEC outputs: 13`
- BSEC 3.2.1 ULP: `BSEC outputs: 13`

LP should use `update_interval: 3s`. ULP can use a slower poll interval, but the first BSEC cycle may take about
five minutes.
