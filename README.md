# renpho-tape-ble

Read the RENPHO smart tape measure **RF-BMF01** locally over Bluetooth LE, without a RENPHO
cloud account, and send the values to [SparkyFitness](https://github.com/CodeWithCJ/SparkyFitness)
via `POST /api/health-data`.

> Status: protocol documented, client not written yet.

## BLE protocol

Captured on 2026-11-10 with an Android HCI snoop log while the official RENPHO app
(`com.renpho.health`) was connected to an RF-BMF01. Three known lengths were measured against a
folding rule: 50.0 cm, 80.0 cm and 100.0 cm.

### GATT

| Handle | UUID | Properties | Used by the app |
| --- | --- | --- | --- |
| 0x000C–0xFFFF | service `0783b03e-8535-b5a0-7140-a304d2495cb7` | – | – |
| 0x000E | `0783b03e-8535-b5a0-7140-a304d2495cb8` | notify | measurement |
| 0x000F | CCCD of 0x000E | – | app writes `01 00` (enable notifications) |
| 0x0011 | `0783b03e-8535-b5a0-7140-a304d2495cba` | write | never written in the capture |

In the capture the app did not pair, did not send any command and did not write to 0x0011. It
connects, enables notifications on 0x000E and only listens. After each measurement the link
ended with HCI disconnect reason `0x08` (connection timeout), and the app reconnected for the
next one.

### Payload

ASCII, 20 bytes per notification, about 10 notifications per second:

```text
*LLLLL;00000;0001XY\n
```

| Field | Meaning | Observed |
| --- | --- | --- |
| `LLLLL` | length in 1/100 cm | `05000` at 50.0 cm, `08000` at 80.0 cm, `10000` at 100.0 cm; last digit always `0` |
| `00000` | unknown | constant in all 892 notifications |
| `0001` | unknown | constant in all 892 notifications |
| `X` | `P` while measuring; `S` for about 0.4 s once per accepted value | one `S` burst at each of 50, 80 and 100 cm |
| `Y` | `M` metric, `I` inch (display unit) | switched to `I` once; the length field stayed `10000` |

Not verified yet: what the two constant fields mean, whether the length field stays metric in
inch mode beyond the single sample above, and how battery level is reported (no standard battery
service was seen).

## License

MIT
