# BME690 tester snippet guidance

This repository contains minimal YAML snippets for external BME690 PR testing. Keep it simple: testers should be able
to copy one snippet into an existing ESPHome config, adjust paths, run it, and paste back a small result table.

## Scope

- Board, Wi-Fi, OTA, API, logger, or I2C bus setup are not part of these snippets.
- Full upload/run harness is not part of this repo.
- Adjust BSEC library and config paths as documented.

## Generating a test matrix

A test harness should run one snippet at a time and report a Markdown table. The matrix should capture:

- `Test`: snippet name, for example `bsec330_lp`
- `ESP`: tested MCU, for example `ESP32-C6` or `ESP32-S3`
- `BSEC`: Bosch BSEC library version
- `Rate`: `LP` or `ULP`
- `Config`: Bosch config variant
- `Outputs`: number from the `BSEC outputs: ...` log line
- `Temp`: whether a BME690 Temperature sensor line was published
- `CO2`: whether a BME690 CO2 Equivalent sensor line was published
- `TVOC`: whether a BME690 TVOC Equivalent sensor line was published
- `IAQ accuracy after 10 min`: final IAQ accuracy text or numeric value after about 10 minutes for LP tests
- `Notes`: warnings, timing violations, path/config mismatches, or calibration observations

Use this table skeleton:

```markdown
| Test | ESP | BSEC | Rate | Config | Outputs | Temp | CO2 | TVOC | IAQ accuracy after 10 min | Notes |
| --- | --- | --- | --- | --- | ---: | --- | --- | --- | --- | --- |
| bsec330_lp |  | 3.3.0.0 | LP | bme690_iaq_33v_3s_4d |  |  |  |  |  |  |
| bsec330_lp_custom_28d |  | 3.3.0.0 | LP | bme690_iaq_33v_3s_28d |  |  |  |  |  |  |
| bsec321_lp |  | 3.2.1.0 | LP | bme690_iaq_33v_3s_4d |  |  |  |  |  |  |
| bsec330_ulp |  | 3.3.0.0 | ULP | bme690_iaq_33v_300s_4d |  |  |  |  |  |  |
| bsec321_ulp |  | 3.2.1.0 | ULP | bme690_iaq_33v_300s_4d |  |  |  |  |  |  |
```

## Interpreting results

- `CO2 Equivalent` is a BSEC estimated output, not a direct CO2 measurement.
- BSEC 3.3 LP should publish `tvoc_equivalent` in ppb.
- BSEC 3.2.x should publish `breath_voc_equivalent` in ppm instead of TVOC.
- BSEC 3.3 ULP is not expected to publish TVOC.
- LP should generally use `update_interval: 3s`. Slower polling can produce timing warning 100.
- ULP may take around 5 minutes before the first BSEC output cycle.
