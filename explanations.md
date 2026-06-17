fuer vlan und so ipv6 address schema:

2001:db8: [Standort] : [VLAN] :: [Gerät]

| Teil      | Bedeutung              |
| --------- | ---------------------- |
| 2001:db8: | Unser PI-Space         |
| 1         | Standort Hamburg       |
| 10        | VLAN 10                |
| ::1       | immer Gateway (Switch) |
| ::10      | erstes Endgerät        |
| ::11      | zweites Endgerät       |

### Beispiel Hamburg:
```text
2001:db8:1:10::1   → Gateway VLAN 10 (Switch)
2001:db8:1:10::10  → Client-HH-01
2001:db8:1:10::11  → Client-HH-02
2001:db8:1:20::1   → Gateway VLAN 20 (Switch)
2001:db8:1:20::10  → Server-HH-01
```