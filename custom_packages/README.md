# custom_packages/

Local feature packages and **intentional overrides** of upstream framework files.

These files are kept out of `ui/`, `common/`, and `hardware/` so upstream merges stay reviewable: reconcile a custom variant against the framework file instead of editing the framework in place.

For AI assistants: read this index before changing anything under `custom_packages/`. For the alarm/timer feature, also read [alarm_clock.md](alarm_clock.md).

---

## Index

| File | Kind | Purpose |
|------|------|---------|
| [alarm_clock.yaml](alarm_clock.yaml) | Feature | Daily alarm + one-shot timer, ringing UI, DOW schedule. See [alarm_clock.md](alarm_clock.md). |
| [coffee_calculator.yaml](coffee_calculator.yaml) | Feature | Coffee dose calculator page; opened when HA `input_select.galley_dashboard_mode` is `coffee_calculator`. |
| [custom_backlight_time.yaml](custom_backlight_time.yaml) | Override | Like `common/backlight_time.yaml`, but idle timeout returns to `clock_page` instead of blanking. |
| [custom_flip_clock.yaml](custom_flip_clock.yaml) | Override | Like `ui/clock/flip_clock.yaml`, but skips digit redraws while the alarm is ringing (audio/PSRAM relief). |
| [custom_guition-esp32-s3-4848s040.yaml](custom_guition-esp32-s3-4848s040.yaml) | Hardware | Guition 4848 panel variant (ST7701S + higher backlight PWM) for flicker/OTA blank-screen issues. |
| [custom_guition-esp32-s3-4848s040-audio.yaml](custom_guition-esp32-s3-4848s040-audio.yaml) | Hardware | Same panel, but **speaker** instead of relay GPIOs (pins conflict). Required for `alarm_clock`. |
| [custom_info_hidden.yaml](custom_info_hidden.yaml) | Nav | Info page exists; skipped in normal swipe (optional switch to include it). |
| [custom_loading.yaml](custom_loading.yaml) | Override | Loading / start-page helpers used by nightstand and kitchen dash configs. |
| [custom_page_navigation_alarm.yaml](custom_page_navigation_alarm.yaml) | Nav | Swipe cycle with **`alarm_page`** stop (nightstand). Sibling of coffee-oriented info-hidden nav. |
| [custom_swipe_navigation.yaml](custom_swipe_navigation.yaml) | Nav | Page `on_swipe_*` hooks that call `show_next/previous_dashboard_page`. |

---

## Conventions

1. **Prefer a new `custom_*.yaml` over editing upstream** under `common/`, `hardware/`, or `ui/`.
2. File header comments should state: base file (if any), behavioral delta, and hard dependencies.
3. Device YAML composes these via `packages:` — see `brett-alarm-it.yaml` / `kitchendash.yaml` in the parent ESPHome config.
4. Feature packages that add LVGL pages must document which pages are **in the swipe cycle** vs drill-in only.

---

## Typical nightstand stack

```yaml
packages:
  hardware: !include esphome-modular-lvgl-buttons/custom_packages/custom_guition-esp32-s3-4848s040-audio.yaml
  backlight: !include esphome-modular-lvgl-buttons/custom_packages/custom_backlight_time.yaml
  loading_screen: !include esphome-modular-lvgl-buttons/custom_packages/custom_loading.yaml
  page_navigation: !include esphome-modular-lvgl-buttons/custom_packages/custom_page_navigation_alarm.yaml
  popup_system: !include esphome-modular-lvgl-buttons/ui/popup/popup_system.yaml
  alarm_clock: !include esphome-modular-lvgl-buttons/custom_packages/alarm_clock.yaml
```

Swipe targets on each dashboard page usually also include:

```yaml
<<: !include esphome-modular-lvgl-buttons/custom_packages/custom_swipe_navigation.yaml
```

Kitchen dash (no speaker yet) uses the non-audio Guition custom hardware file and does **not** include `alarm_clock.yaml`.
