# Silakka54 – Vial + LED keymaps for KVM keyboard ports

Based on the upstream `vial_led` keymap (v1.4led, Squalius-cephalus/silakka54):
8 layers, Caps Word, Layer Lock, per-layer RP2040-Zero LED colour.

## Why

Some KVM switches (tested: UGREEN 50744 / CM200, 2-port HDMI) only relay part
of a composite HID device on their hotkey-capable keyboard port. With stock
QMK the mouse lives on the shared endpoint (report IDs, non-boot interface,
EP4) and mouse keys are silently dropped.

Observed on the KVM keyboard port:

| Interface                 | Relayed? |
|---------------------------|----------|
| Boot keyboard (EP1)       | yes      |
| Boot mouse (EP2)          | yes      |
| Raw HID / Vial (non-boot) | no       |
| Shared EP (non-boot)      | no       |

It is not yet known whether the KVM filters by interface class (boot vs
non-boot) or by endpoint number (only EP1/EP2).

## Keymaps

### `vial_led_kvm` (stable)

`MOUSE_SHARED_EP = no` → mouse on its own boot-mouse interface (EP2).
Mouse keys work through the KVM. Media keys and Vial do not.

| IF | Content                      | EP      |
|----|------------------------------|---------|
| 0  | Boot keyboard                | 1       |
| 1  | Vial raw HID                 | 3 / 4   |
| 2  | Boot mouse                   | 2       |
| 3  | Shared: system, consumer, NKRO | 5     |

### `vial_led_kvm_exp` (experimental)

Uses three opt-in core options added to `tmk_core/protocol/usb_descriptor.*`:

- `KVM_SHARED_EP_BEFORE_RAW` – allocate the shared endpoint before raw HID (EP2)
- `KVM_SHARED_EP_BOOT_MOUSE` – advertise the shared interface as boot mouse
- `KVM_RAW_BOOT_KEYBOARD` – advertise the raw HID (Vial) interface as boot keyboard

| IF | Content                              | EP    |
|----|--------------------------------------|-------|
| 0  | Boot keyboard                        | 1     |
| 1  | Vial raw HID (flagged boot keyboard) | 3 / 4 |
| 2  | Shared: mouse, system, consumer, NKRO (flagged boot mouse) | 2 |

If the KVM gates by class *or* by endpoint number, mouse **and** media keys
should pass. Vial passes only if the KVM gates by class and relays OUT reports.

## Build

    make silakka54:vial_led_kvm
    make silakka54:vial_led_kvm_exp

Flash the `.uf2` to **both** halves (RP2040 bootloader → `RPI-RP2`), then
restore the layout (`.vil`) in Vial.
