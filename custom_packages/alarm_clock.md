# Alarm clock package

Source: [`alarm_clock.yaml`](alarm_clock.yaml)

On-device daily alarm and one-shot timer for Guition 4848 panels with the NS4168 speaker. Designed as a **feature package**: include it from a device YAML; do not fork it into each panel config.

**Scope today:** one daily alarm + one timer. Multi-alarm slots are deferred until the current firmware proves stable in daily use.

---

## Dependencies

| Requirement | Package / notes |
|-------------|-----------------|
| Speaker hardware | `custom_guition-esp32-s3-4848s040-audio.yaml` (not the relay Guition file) |
| Popup scrim/timeout | `ui/popup/popup_system.yaml` |
| Swipe includes `alarm_page` | `custom_page_navigation_alarm.yaml` |
| Per-page swipe hooks | `custom_swipe_navigation.yaml` on swipeable pages |
| Clock overlay | Device `clock_page` should set `multiple_widgets_per_cell: true` so the status label can share the flip-clock cell |
| On-device WAVs | Under `assets/sounds/` (chimes, chords, jingle, tap, transition, xylophone) |

Reference device: parent config `brett-alarm-it.yaml`.

---

## Pages

| Page ID | In swipe? | Role |
|---------|-----------|------|
| `alarm_page` | Yes | Daily alarm settings (time, days, enable, sound, gentle wake, link to timer) |
| `timer_page` | No | HH:MM:SS one-shot timer setup |
| `alarm_time_page` | No | Keypad editor for alarm clock time |
| `alarm_ringing_page` | No | Full-screen ring UI |
| `alarm_dismiss_confirm_page` | No | Hold-dismiss confirmation |
| `clock_page` (extended) | Yes | Adds floating `alarm_status_label` |

Drill-in pages are opened by scripts; they are not swipe stops.

---

## Behaviors (invariants)

Keep these true when editing:

1. **One audio / ring path** — daily alarm and timer share ringing, snooze, and dismiss UI.
2. **Timer overrides the daily alarm** while the timer is armed, snoozing, or ringing (avoids dual audio load). Starting a timer dismisses an active daily ring/snooze first.
3. **Gentle wake** applies to the **daily alarm only**. Timer always rings at max digital level (no ramp).
4. **Day-of-week** via `alarm_days_mask` (bit0=Sun … bit6=Sat, ESPTime `day_of_week` 1–7). Default all days on. No catch-up if a day was skipped.
5. **Clock status** (when the daily alarm is enabled): `Next Mon • 07:00` for the next fire; `No alarm scheduled` if no days selected. Timer/snooze lines take priority when active.
6. **Snooze length** is shared (HA/web number `alarm_snooze_minutes`). Per-alarm snooze is not implemented.
7. **Separate sounds** — `alarm_sound_choice` vs `timer_sound_choice`; one shared picker popup (`sound_picker_for_timer` selects which).
8. **Timer state is not restored across reboot** (conservative). Pending daily snooze can survive reboot as before.

Ring UI: tap = snooze (until snooze lockout), hold = dismiss confirm.

---

## Device UI map

- **Alarm page:** time, `S M T W T F S` days summary → days popup, enable, sound popup, gentle wake, **Timer** button → `timer_page`.
- **Timer page:** right-fill HH:MM:SS keypad, timer sound, Start/Stop (Stop cancels countdown, snooze, and ringing), Back → alarm page.
- **Clock status tap:** opens timer page if a timer is counting or timer-snoozing; otherwise alarm page.

---

## Porting to another panel (e.g. kitchen dash)

Checklist:

1. Hardware must expose the speaker stack (GPIO conflict with relays on this board).
2. Include popup system + alarm package + alarm page navigation (or fold `alarm_page` into that panel’s swipe scripts).
3. Ensure `clock_page` allows the status overlay.
4. Add any missing fonts/glyphs used by alarm UI (`nunito_2026`, etc.).
5. Do **not** include this package on a panel without audio — it will fail or waste RAM for unused ring paths.

Kitchen dash historically predates the alarm overlay; until the package is included, its flip clock is unchanged.

---

## HA / web entities (high level)

Exposed for configuration and diagnostics (names may vary with device `name:`):

- Enable, hour, minute, DOW (on-device mask; toggled from LVGL)
- Alarm / timer sound selects
- Gentle wake switch; gentle minutes; snooze minutes; cutoff minutes
- Volume min/max (digital ramp bounds; analog pot still affects loudness)
- Playback volume diagnostic sensor

Snooze/ramp/cutoff are intentionally **not** full LVGL editors — change from HA or the ESPHome web UI.

---

## Edit guidance

- Prefer extending `alarm_clock.yaml` over copying logic into device YAML.
- Avoid adding a second concurrent media pipeline.
- When touching schedule logic, update both `check_alarm_schedule` and `update_clock_status_label`.
- YAML file header comments are the short living summary; keep this doc aligned when invariants change.
