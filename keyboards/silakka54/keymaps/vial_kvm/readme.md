# Silakka54 – Vial keymap for KVM keyboard ports

Same as the `vial` keymap, plus `MOUSE_SHARED_EP = no`.

## Why

Some KVM switches (tested: UGREEN 50744 / CM200, 2-port HDMI) only forward
data from HID **boot** interfaces on their dedicated keyboard/mouse ports, which
are the ports that support hotkey switching. Stock QMK puts the mouse on the shared endpoint
together with media keys and NKRO (report IDs, non-boot interface), so mouse
keys are silently dropped by the KVM.

With `MOUSE_SHARED_EP = no`, QMK exposes the mouse on its own interface with
`bInterfaceSubClass = 1 (Boot)` / `bInterfaceProtocol = 2 (Mouse)`, which the KVM
forwards. Mouse keys then work through the KVM keyboard port.

| Interface | Content |
|-----------|---------|
| 0 | Boot keyboard |
| 1 | Vial raw HID |
| 2 | Boot mouse (new) |
| 3 | Shared EP: system, consumer, NKRO |

## Known limitations on the KVM keyboard port

- Media keys (interface 3) are still dropped.
- Vial/raw HID (interface 1) is not reachable; configure the keyboard on a
  regular USB port.

## Build

    make silakka54:vial_kvm

Flash the resulting `.uf2` to **both** halves (RP2040 bootloader → `RPI-RP2`).
