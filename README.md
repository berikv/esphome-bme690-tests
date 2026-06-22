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

BSEC 3.2.x exposes `breath_voc_equivalent`. BSEC 3.3.x disables Breath VOC for BME690, so those snippets leave it
out.

`temperature_offset` is board-specific self-heating compensation. Start at `0`, then tune it to roughly
`raw die temperature - ambient temperature` for your board and enclosure.
