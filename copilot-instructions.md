# Copilot Instructions — ESPHome Modular LVGL Buttons

## Project Overview

This repository implements a modular UI system for ESPHome using LVGL.

Architecture is based on:
- YAML includes (`packages`, `!include`)
- Per-entity modular components under `ui/<type>/`
- Abstract script contracts to decouple UI from implementation
- Shared infrastructure under `common/` and `pages/`

⚠️ Critical: Behavior is distributed across multiple files. No file is self-contained.

---

## Core Architecture Rules

### 1. Entity Model

Each entity type is implemented in:

ui/<type>/
  local.yaml
  remote.yaml
  detail.yaml (complex types only)

- local.yaml: controls native ESPHome components
- remote.yaml: controls Home Assistant via services and sensors
- detail.yaml: UI layer only (no direct entity interaction)

All entity types MUST support both local and remote variants.

---

### 2. Abstract Script Contract

- All entity actions are exposed via scripts named ${uid}_<action>
- UI layers (detail pages, popups) MUST call scripts only
- UI must NEVER reference:
  - component IDs
  - Home Assistant entity IDs

When modifying or adding features:
- Always preserve script interfaces
- Do not bypass the script layer

---

### 3. Local vs Remote Behavior

Local and remote implementations must be functionally identical:

- Same UI
- Same globals
- Same script names

They differ ONLY in:
- how state is read
- how actions are executed

---

### 4. Detail Page Rules (Complex Types Only)

- Declared in ui/<type>/detail.yaml
- Included via packages in both local and remote files
- Defines all shared globals for that type
- Uses ONLY abstract scripts

Never:
- reference entity IDs
- duplicate logic already implemented in scripts

---

## Dependency Model (CRITICAL)

This codebase has implicit cross-file dependencies.

Before suggesting ANY change:

1. Trace:
   - globals
   - scripts
   - widget IDs
   - includes (packages)
2. Identify:
   - where they are defined
   - where they are consumed
3. Confirm nothing else depends on them

⚠️ Removing or modifying one file can break unrelated features.

---

## Folder Rules

Do NOT modify unless explicitly required:
- common/
- pages/
- hardware/

---

## Variable Contract

All entity includes must support:

- uid
- entity_id
- row, column
- text, icon
- optional: row_span, column_span, page_id

Rules:
- Namespace EVERYTHING with ${uid}_
- Never reuse IDs across entities

---

## Naming & Namespacing

Everything must be namespaced:

- scripts → ${uid}_...
- globals → ${uid}_...
- widgets → ${uid}_...

No exceptions.

---

## Data Representation

- Brightness → float (0.0–1.0)
- Color temp → Kelvin
- Sliders → 0–255 internally
- HA attributes → parse strings

---

## UI Rules

- No fixed pixel coordinates
- Use alignment, grid, percentage sizing
- Must support multiple resolutions

---

## Theme Rules

Use ONLY theme variables:
- $button_on_color
- $button_off_color
- $icon_on_color
- $icon_off_color
- $label_on_color
- $label_off_color
- $icon_font

---

## Popup System

- All popups live in top_layer

To open:
- show popup_scrim
- show popup
- run popup_start_timeout

To close:
- scrim tap or popup_force_close

---

## Adding New Entity Types

1. Create ui/<type>/
2. Define script contract
3. Implement local + remote
4. Add detail.yaml if needed
5. Define globals
6. Namespace everything

---

## ESPHome Syntax Rules

- Use ${...}
- No {% if %}
- Use inline expressions

---

## Required Copilot Behavior

1. Never assume files are independent
2. Always trace dependencies first
3. List all affected files
4. Identify scripts, globals, includes
5. Prefer minimal changes

---

## ESPHome Version

ESPHome 2026.5.0+
