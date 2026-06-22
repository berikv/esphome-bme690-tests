# BME690 BSEC test snippets

These YAML files are minimal snippets for testing the BME690 PR branch. They intentionally leave out board, Wi-Fi,
OTA, logger, API, and I2C bus setup. Add one snippet to an existing working ESPHome config that already defines:

- an ESP32 ESP-IDF target
- an `i2c:` bus with `id: bus_a`, or update `i2c_id`
- logger/API/OTA/Wi-Fi as needed for your device

Update `external_components.source` to the branch under test if needed. Update `bsec_library` and `bsec_config` to
files available on your ESPHome build host. The BSEC library archive and config file must match:

- the same Bosch BSEC release, for example 3.2.1.0 or 3.3.0.0
- the MCU target archive, for example `esp32_s3` or `esp32_c2c3`
- the same sample rate family, for example `3s`/LP or `300s`/ULP
- the same sensor voltage and history window, for example `33v_3s_4d`

## Test matrix

| File | Library | Config | Rate | Expected first cycle |
| --- | --- | --- | --- | --- |
| `bme690-bsec330-lp.yaml` | BSEC 3.3.0.0 | `bme690_iaq_33v_3s_4d` | LP, `3s` | `BSEC outputs: 13` |
| `bme690-bsec330-lp-custom-config.yaml` | BSEC 3.3.0.0 | `bme690_iaq_33v_3s_28d` | LP, `3s` | `BSEC outputs: 13` |
| `bme690-bsec330-ulp.yaml` | BSEC 3.3.0.0 | `bme690_iaq_33v_300s_4d` | ULP, `5s` | `BSEC outputs: 12` |
| `bme690-bsec321-lp.yaml` | BSEC 3.2.1.0 | `bme690_iaq_33v_3s_4d` | LP, `3s` | `BSEC outputs: 13` |
| `bme690-bsec321-ulp.yaml` | BSEC 3.2.1.0 | `bme690_iaq_33v_300s_4d` | ULP, `5s` | `BSEC outputs: 13` |

The custom 28-day config file is included at `config/bme690_iaq_33v_3s_28d.config`. Copy it to a path that your
ESPHome build can read, or update `bme690-bsec330-lp-custom-config.yaml` to point at this file directly.

## Expected logs

A passing test should show the BSEC control path and virtual outputs, for example:

```text
[I][bme690:...]: BSEC control: ...
[I][bme690:...]: BSEC outputs: 13
[S][sensor]: 'BME690 Temperature' >> ...
[S][sensor]: 'BME690 CO2 Equivalent' >> ...
```

LP should use `update_interval: 3s`. Slower LP polling can trigger `BSEC_W_SC_CALL_TIMING_VIOLATION` warning 100.
ULP can take up to 5 minutes for the first values to appear.

BSEC 3.2.x exposes `breath_voc_equivalent` in ppm. BSEC 3.3.x disables Breath VOC for BME690 and exposes
`tvoc_equivalent` in ppb for LP only, so only the BSEC 3.3 LP snippets include it.

`temperature_offset` is board-specific self-heating compensation. Start at `0`, then tune it to roughly
`raw die temperature - ambient temperature` for your board and enclosure.

## Reporting results

Run one YAML snippet at a time from your ESPHome checkout. Replace the config path and device name with your local
setup:

```bash
python3 -m esphome run /path/to/your/config.yaml --device your-device.local 2>&1 | tee bme690-test.log
```

Run each test up to 10 minutes. For ULP tests, run long enough to see the first BSEC cycle; this can take
around 5 minutes. The key result is whether BSEC publishes the expected virtual outputs, not whether IAQ is fully
calibrated yet.

Please paste the relevant log lines and this filled-in table when reporting results:

```markdown
| Test | ESP | BSEC | Rate | Config | Outputs | Temp | CO2 | TVOC | IAQ accuracy after 10 min | Notes |
| --- | --- | --- | --- | --- | ---: | --- | --- | --- | --- | --- |
| bsec330_lp |  | 3.3.0.0 | LP | bme690_iaq_33v_3s_4d |  |  |  |  |  |  |
| bsec330_lp_custom_28d |  | 3.3.0.0 | LP | bme690_iaq_33v_3s_28d |  |  |  |  |  |  |
| bsec321_lp |  | 3.2.1.0 | LP | bme690_iaq_33v_3s_4d |  |  |  |  |  |  |
| bsec330_ulp |  | 3.3.0.0 | ULP | bme690_iaq_33v_300s_4d |  |  |  |  |  |  |
| bsec321_ulp |  | 3.2.1.0 | ULP | bme690_iaq_33v_300s_4d |  |  |  |  |  |  |
```

Use `yes`/`no` for Temp, CO2, and TVOC. `Outputs` is the number from the `BSEC outputs: ...` log line. For BSEC 3.3
LP, TVOC is expected. For BSEC 3.2.x and BSEC 3.3 ULP, TVOC is not expected.
