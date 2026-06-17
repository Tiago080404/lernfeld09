```text
LUE ──────── HH
 │  ╲      ╱  │
 │    ╲  ╱    │
 │      ╳     │
 │    ╱  ╲    │
 │  ╱      ╲  │
 B ────────── M
```

### Hamburg

```text
enable
configure terminal
hostname RT-HH-01
ipv6 unicast-routing

interface Serial0/1/0
 ipv6 address fd00::1:1/64
 ipv6 enable
 clock rate 64000
 no shutdown

interface Serial0/1/1
 ipv6 address fd00::2:1/64
 ipv6 enable
 clock rate 64000
 no shutdown

interface Serial0/2/0
 ipv6 address fd00::3:1/64
 ipv6 enable
 clock rate 64000
 no shutdown

interface GigabitEthernet0/0/0
 ipv6 address 2001:db8:1::1/64
 ipv6 enable
 no shutdown

! Direkte Routen
ipv6 route 2001:db8:2::/64 fd00::1:2
ipv6 route 2001:db8:3::/64 fd00::2:2
ipv6 route 2001:db8:4::/64 fd00::3:2


! Und Die ROUTEN ZU DEN VLAN(MUENCHEN)
ipv6 route 2001:db8:4:10::/64 fd00:0:0:3::2

end
write memory
```

### Lübeck

```text
enable
configure terminal
hostname RT-LUE-01
ipv6 unicast-routing

! WAN zu Hamburg
interface Serial0/1/0
 ipv6 address fd00:0:0:1::2/64
 ipv6 enable
 no shutdown

! WAN zu Berlin
interface Serial0/1/1
 ipv6 address fd00:0:0:4::1/64
 ipv6 enable
 clock rate 64000
 no shutdown

! WAN zu München
interface Serial0/2/0
 ipv6 address fd00:0:0:5::1/64
 ipv6 enable
 clock rate 64000
 no shutdown

interface GigabitEthernet0/0/0
 ipv6 address 2001:db8:2::1/64
 ipv6 enable
 no shutdown

ipv6 route 2001:db8:1::/64 fd00::1:1
ipv6 route 2001:db8:3::/64 fd00::4:2
ipv6 route 2001:db8:4::/64 fd00::5:2

end
write memory
```

### Berlin

```text
enable
configure terminal
hostname RT-B-01
ipv6 unicast-routing

! WAN zu Hamburg
interface Serial0/1/0
 ipv6 address fd00:0:0:2::2/64
 ipv6 enable
 no shutdown

! WAN zu Lübeck
interface Serial0/1/1
 ipv6 address fd00:0:0:4::2/64
 ipv6 enable
 no shutdown

! WAN zu München
interface Serial0/2/0
 ipv6 address fd00:0:0:6::1/64
 ipv6 enable
 clock rate 64000
 no shutdown

interface GigabitEthernet0/0/0
 ipv6 address 2001:db8:3::1/64
 ipv6 enable
 no shutdown

ipv6 route 2001:db8:1::/64 fd00::2:1
ipv6 route 2001:db8:2::/64 fd00::4:1
ipv6 route 2001:db8:4::/64 fd00::6:2

end
write memory
```

### München

```text
enable
configure terminal
hostname RT-M-01
ipv6 unicast-routing

! WAN zu Hamburg
interface Serial0/1/0
 ipv6 address fd00:0:0:3::2/64
 ipv6 enable
 no shutdown

! WAN zu Lübeck
interface Serial0/1/1
 ipv6 address fd00:0:0:5::2/64
 ipv6 enable
 no shutdown

! WAN zu Berlin
interface Serial0/2/0
 ipv6 address fd00:0:0:6::2/64
 ipv6 enable
 no shutdown

interface GigabitEthernet0/0/0
 ipv6 address 2001:db8:4::1/64
 ipv6 enable
 no shutdown

! Routen zu anderen Standorten
ipv6 route 2001:db8:1::/64 fd00:0:0:3::1
ipv6 route 2001:db8:2::/64 fd00:0:0:5::1
ipv6 route 2001:db8:3::/64 fd00:0:0:6::1

! Und Die ROUTEN ZU DEN VLAN(HAMBURG)
ipv6 route 2001:db8:1:10::/64 fd00:0:0:3::1
ipv6 route 2001:db8:1:20::/64 fd00:0:0:3::1

end
write memory
```

## SWITCHES

### Hamburg L3-Switch (SW-HH-01)

```text
enable
configure terminal
hostname SW-HH-01
sdm prefer dual-ipv4-and-ipv6 default
end
reload
```

```text
enable
configure terminal

ipv6 unicast-routing

! === VLANs erstellen ===
vlan 10
 name Clients-HH
vlan 20
 name Server-HH

! === VLAN Interfaces (Gateway für Clients/Server) ===
interface vlan 10
 ipv6 address 2001:db8:1:10::1/64
 ipv6 enable
 no shutdown

interface vlan 20
 ipv6 address 2001:db8:1:20::1/64
 ipv6 enable
 no shutdown

! === Uplink zum Router (Trunk) ===
interface GigabitEthernet1/0/1
 no switchport
 ipv6 address 2001:db8:1::2/64
 ipv6 enable
 no shutdown

! === Access Ports (Clients) ===
interface GigabitEthernet1/0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface GigabitEthernet1/0/3
 switchport mode access
 switchport access vlan 10
 no shutdown

! === Access Port VLAN 20 (Server) ===
interface GigabitEthernet1/0/4
 switchport mode access
 switchport access vlan 20
 no shutdown

! === Default Route zum Router ===
ipv6 route ::/0 2001:db8:1::1

end
write memory
```

### Erklärung Uplink

Der Port am Switch der zum Router zeigt — also "nach oben" in Richtung Netzwerk.

```text
[PCs/Server] ──── Switch ──Uplink──► Router ──► Internet/WAN
```

### Lübeck L3-Switch (SW-LUE-01)

```text
enable
configure terminal
hostname SW-LUE-01
sdm prefer dual-ipv4-and-ipv6 default
end
reload
```

```text
enable
configure terminal

ipv6 unicast-routing

! === VLANs erstellen ===
vlan 10
 name Clients-LUE
vlan 20
 name Server-LUE

! === VLAN Interfaces (Gateways) ===
interface vlan 10
 ipv6 address 2001:db8:2:10::1/64
 ipv6 enable
 no shutdown

interface vlan 20
 ipv6 address 2001:db8:2:20::1/64
 ipv6 enable
 no shutdown

! === Uplink zum Router ===
interface GigabitEthernet1/0/1
 no switchport
 ipv6 address 2001:db8:2::2/64
 ipv6 enable
 no shutdown

! === Access Ports (Clients) ===
interface GigabitEthernet1/0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

interface GigabitEthernet1/0/3
 switchport mode access
 switchport access vlan 10
 no shutdown

! === Access Port VLAN 20 (Server) ===
interface GigabitEthernet1/0/4
 switchport mode access
 switchport access vlan 20
 no shutdown

! === Default Route zum Router ===
ipv6 route ::/0 2001:db8:2::1

end
write memory
```

### München L3-Switch (SW-M-01)

```text
enable
configure terminal
hostname SW-M-01
sdm prefer dual-ipv4-and-ipv6 default
end
reload
```

```text
enable
configure terminal

ipv6 unicast-routing

! === VLAN erstellen ===
vlan 10
 name Server-M

! === VLAN Interface (Gateway) ===
interface vlan 10
 ipv6 address 2001:db8:4:10::1/64
 ipv6 enable
 no shutdown

! === Uplink zum Router ===
interface GigabitEthernet1/0/1
 no switchport
 ipv6 address 2001:db8:4::2/64
 ipv6 enable
 no shutdown

! === Access Port VLAN 10 (Server) ===
interface GigabitEthernet1/0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

! === Default Route zum Router ===
ipv6 route ::/0 2001:db8:4::1

end
write memory
```

### Berlin

```text
enable
configure terminal
hostname SW-B-01
sdm prefer dual-ipv4-and-ipv6 default
end
reload
```

```text
enable
configure terminal

ipv6 unicast-routing

! === VLAN erstellen ===
vlan 10
 name Server-B

! === VLAN Interface (Gateway) ===
interface vlan 10
 ipv6 address 2001:db8:3:10::1/64
 ipv6 enable
 no shutdown

! === Uplink zum Router ===
interface GigabitEthernet1/0/1
 no switchport
 ipv6 address 2001:db8:3::2/64
 ipv6 enable
 no shutdown

! === Access Port ===
interface GigabitEthernet1/0/2
 switchport mode access
 switchport access vlan 10
 no shutdown

! === Default Route zum Router ===
ipv6 route ::/0 2001:db8:3::1

end
write memory
```

```text
Was wir geändert haben
Das Problem vorher:

Wir hatten Adressen wie:

fd00::1:1
fd00::1:2
fd00::2:1
fd00::2:2

Diese liegen alle im gleichen Netz fd00::/64 — Cisco erkennt das als Überlappung und lehnt es ab!
Die Lösung:

Wir haben für jede Verbindung ein eigenes /64 Netz gemacht:
Verbindung	Altes Netz ❌	Neues Netz ✅
HH ↔ LUE	fd00::1:x	fd00:0:0:1::/64
HH ↔ B	fd00::2:x	fd00:0:0:2::/64
HH ↔ M	fd00::3:x	fd00:0:0:3::/64
LUE ↔ B	fd00::4:x	fd00:0:0:4::/64
LUE ↔ M	fd00::5:x	fd00:0:0:5::/64
B ↔ M	fd00::6:x	fd00:0:0:6::/64
Warum war das ein Problem?

fd00::1:1  =  fd00:0000:0000:0000:0000:0000:0001:0001
fd00::2:1  =  fd00:0000:0000:0000:0000:0000:0002:0001

    Beide liegen im Netz fd00::/64 — also gleiches Netz, verschiedene Hosts!

fd00:0:0:1::1  =  fd00:0000:0000:0001:0000:0000:0000:0001
fd00:0:0:2::1  =  fd00:0000:0000:0002:0000:0000:0000:0001

    Jetzt sind es verschiedene Netze — kein Konflikt!

Kurz gesagt:

    Die Netzteile /64 waren vorher identisch — jetzt ist jede Verbindung in einem eigenen eindeutigen Subnetz.
```

## Alle Pings

---

### Von RT-HH-01

```text
RT-HH-01#ping ipv6 fd00:0:0:1::2    ! zu RT-LUE Serial
RT-HH-01#ping ipv6 fd00:0:0:2::2    ! zu RT-B Serial
RT-HH-01#ping ipv6 fd00:0:0:3::2    ! zu RT-M Serial
RT-HH-01#ping ipv6 2001:db8:1::2    ! zu SW-HH
RT-HH-01#ping ipv6 2001:db8:2::1    ! zu LUE LAN
RT-HH-01#ping ipv6 2001:db8:3::1    ! zu B LAN
RT-HH-01#ping ipv6 2001:db8:4::1    ! zu M LAN
```

### Von RT-LUE-01

```text
RT-LUE-01#ping ipv6 fd00:0:0:1::1    ! zu RT-HH Serial
RT-LUE-01#ping ipv6 fd00:0:0:4::2    ! zu RT-B Serial
RT-LUE-01#ping ipv6 fd00:0:0:5::2    ! zu RT-M Serial
RT-LUE-01#ping ipv6 2001:db8:2::2    ! zu SW-LUE
RT-LUE-01#ping ipv6 2001:db8:1::1    ! zu HH LAN
RT-LUE-01#ping ipv6 2001:db8:3::1    ! zu B LAN
RT-LUE-01#ping ipv6 2001:db8:4::1    ! zu M LAN
```

### Von RT-B-01

```text
RT-B-01#ping ipv6 fd00:0:0:2::1    ! zu RT-HH Serial
RT-B-01#ping ipv6 fd00:0:0:4::1    ! zu RT-LUE Serial
RT-B-01#ping ipv6 fd00:0:0:6::2    ! zu RT-M Serial
RT-B-01#ping ipv6 2001:db8:3::2    ! zu SW-B
RT-B-01#ping ipv6 2001:db8:1::1    ! zu HH LAN
RT-B-01#ping ipv6 2001:db8:2::1    ! zu LUE LAN
RT-B-01#ping ipv6 2001:db8:4::1    ! zu M LAN
```

### Von RT-M-01

```text
RT-M-01#ping ipv6 fd00:0:0:3::1    ! zu RT-HH Serial
RT-M-01#ping ipv6 fd00:0:0:5::1    ! zu RT-LUE Serial
RT-M-01#ping ipv6 fd00:0:0:6::1    ! zu RT-B Serial
RT-M-01#ping ipv6 2001:db8:4::2    ! zu SW-M
RT-M-01#ping ipv6 2001:db8:1::1    ! zu HH LAN
RT-M-01#ping ipv6 2001:db8:2::1    ! zu LUE LAN
RT-M-01#ping ipv6 2001:db8:3::1    ! zu B LAN
```

### every ping has to work (works day 17.06)

## Aktueller Stand:

```text
✅ Erledigt:

IPv6 unicast-routing auf allen Routern
Transportnetze (ULA fd00:0:0:x::/64)
Statische Routen auf allen Routern
Full-Mesh Serial-Verbindungen
Switch-Konfiguration mit VLANs für HH, LUE, B, M
```

### Statische IPv6 Adressen für alle Geräte

## Hamburg (HH) — 2 VLANs

## Clients und Server einrichten fuer hh:

#### Client-HH-01

```text
IPv6 Address:    2001:db8:1:10::10
Prefix Length:   64
Default Gateway: 2001:db8:1:10::1
```

#### Client-HH-02

```text
IPv6 Address:    2001:db8:1:10::11
Prefix Length:   64
Default Gateway: 2001:db8:1:10::1
```

#### Server-HH-01

```text
IPv6 Address:    2001:db8:1:20::10
Prefix Length:   64
Default Gateway: 2001:db8:1:20::1
```

### Pings von Client 1

```text
ping 2001:db8:1:10::11    ! zu Client-HH-02
ping 2001:db8:1:20::10    ! zu Server-HH-01
ping 2001:db8:1::1        ! zu Router HH
```

### Luebeck (LUE) — 2 VLANs

## Clients und Server einrichten fuer LUE:

#### Client-LUE-01

```text
IPv6 Address:    2001:db8:2:10::10
Prefix Length:   64
Default Gateway: 2001:db8:2:10::1
```

#### Client-LUE-02

```text
IPv6 Address:    2001:db8:2:10::11
Prefix Length:   64
Default Gateway: 2001:db8:2:10::1
```

#### Server-LUE-01

```text
IPv6 Address:    2001:db8:2:20::10
Prefix Length:   64
Default Gateway: 2001:db8:2:20::1
```

```text
ping 2001:db8:2::1 | zu Router LUE
ping 2001:db8:2:10::11    ! zu Client-LUE-02
ping 2001:db8:2:20::10    ! zu Server-LUE-01
```

### Muenchen (MUE) — 1 VLAN

## Server einrichten fuer MUE:

#### Server-M-01

```text
IPv6 Address:    2001:db8:4:10::10
Prefix Length:   64
Default Gateway: 2001:db8:4:10::1
```

```text
ping 2001:db8:4::1        ! zu SW-M-01 Gateway
ping 2001:db8:4:10::1     ! zu VLAN 10 Gateway
ping 2001:db8:1:10::10    ! zu Client-HH-01 Hamburg
```
