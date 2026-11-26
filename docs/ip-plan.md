# IP Plan dan Port Mapping

Dokumen pendukung untuk [README](../README.md). Semua subnet memakai mask `255.255.255.0` (/24).

## 1. VLAN dan subnet

| VLAN | Nama | Network | Gateway | Rentang host | Broadcast |
|------|------|---------|---------|--------------|-----------|
| 10 | HR | 192.168.10.0/24 | 192.168.10.1 | .1 - .254 | 192.168.10.255 |
| 20 | IT | 192.168.20.0/24 | 192.168.20.1 | .1 - .254 | 192.168.20.255 |
| 30 | FINANCE | 192.168.30.0/24 | 192.168.30.1 | .1 - .254 | 192.168.30.255 |
| 40 | GUEST | 192.168.40.0/24 | 192.168.40.1 | .1 - .254 | 192.168.40.255 |
| 99 | MGMT (native) | 192.168.99.0/24 | 192.168.99.1 | .1 - .254 | 192.168.99.255 |

Alokasi alamat pada project ini:

- `.1`: gateway (subinterface R1)
- `.11` dan seterusnya: PC (IP static)
- VLAN 99: `.1` R1, `.2` SW-L1, `.3` SW-L2, `.4` SW-L3

## 2. Subinterface router R1

Semua subinterface berada di `GigabitEthernet0/0`.

| Subinterface | Encapsulation | IP address | Keterangan |
|--------------|---------------|------------|------------|
| G0/0.10 | dot1Q 10 | 192.168.10.1/24 | Gateway HR |
| G0/0.20 | dot1Q 20 | 192.168.20.1/24 | Gateway IT |
| G0/0.30 | dot1Q 30 | 192.168.30.1/24 | Gateway Finance |
| G0/0.40 | dot1Q 40 | 192.168.40.1/24 | Gateway Guest |
| G0/0.99 | dot1Q 99 native | 192.168.99.1/24 | Management (native) |

## 3. PC per lantai

### Lantai 1 (SW-L1)

| PC | Port | VLAN | IP address | Gateway |
|----|------|------|------------|---------|
| PC-IT-L1a | Fa0/1 | 20 IT | 192.168.20.11 | 192.168.20.1 |
| PC-IT-L1b | Fa0/2 | 20 IT | 192.168.20.12 | 192.168.20.1 |
| PC-GUEST-L1a | Fa0/3 | 40 GUEST | 192.168.40.11 | 192.168.40.1 |
| PC-GUEST-L1b | Fa0/4 | 40 GUEST | 192.168.40.12 | 192.168.40.1 |

### Lantai 2 (SW-L2)

| PC | Port | VLAN | IP address | Gateway |
|----|------|------|------------|---------|
| PC-HR-L2a | Fa0/1 | 10 HR | 192.168.10.11 | 192.168.10.1 |
| PC-HR-L2b | Fa0/2 | 10 HR | 192.168.10.12 | 192.168.10.1 |
| PC-FIN-L2a | Fa0/3 | 30 FINANCE | 192.168.30.11 | 192.168.30.1 |
| PC-FIN-L2b | Fa0/4 | 30 FINANCE | 192.168.30.12 | 192.168.30.1 |

### Lantai 3 (SW-L3)

| PC | Port | VLAN | IP address | Gateway |
|----|------|------|------------|---------|
| PC-FIN-L3a | Fa0/1 | 30 FINANCE | 192.168.30.13 | 192.168.30.1 |
| PC-FIN-L3b | Fa0/2 | 30 FINANCE | 192.168.30.14 | 192.168.30.1 |
| PC-IT-L3a | Fa0/3 | 20 IT | 192.168.20.13 | 192.168.20.1 |
| PC-IT-L3b | Fa0/4 | 20 IT | 192.168.20.14 | 192.168.20.1 |
| PC-HR-L3 | Fa0/5 | 10 HR | 192.168.10.13 | 192.168.10.1 |

Total: 13 PC (Lantai 1 = 4, Lantai 2 = 4, Lantai 3 = 5).

## 4. Port access per switch

| Switch | Port | VLAN |
|--------|------|------|
| SW-L1 | Fa0/1-2 | 20 IT |
| SW-L1 | Fa0/3-4 | 40 GUEST |
| SW-L2 | Fa0/1-2 | 10 HR |
| SW-L2 | Fa0/3-4 | 30 FINANCE |
| SW-L3 | Fa0/1-2 | 30 FINANCE |
| SW-L3 | Fa0/3-4 | 20 IT |
| SW-L3 | Fa0/5 | 10 HR |

## 5. Kabel dan trunk

| Dari | Ke | Jenis kabel | Mode | Allowed VLAN | Native VLAN |
|------|----|-------------|------|--------------|-------------|
| R1 G0/0 | SW-L1 Gi0/1 | Copper Straight-Through | Trunk | 10, 20, 30, 40, 99 | 99 |
| SW-L1 Fa0/24 | SW-L2 Fa0/24 | Copper Cross-Over | Trunk | 10, 30, 99 | 99 |
| SW-L1 Fa0/23 | SW-L3 Fa0/24 | Copper Cross-Over | Trunk | 10, 20, 30, 99 | 99 |

Catatan: pada link ke SW-L2 dan SW-L3, daftar `allowed vlan` di sisi SW-L1 memuat semua VLAN (10, 20, 30, 40, 99), sedangkan sisi SW-L2 dan SW-L3 hanya memuat VLAN yang dipakai di lantai tersebut. Kolom Allowed VLAN di tabel menunjukkan daftar di sisi switch lantai.

## 6. Management

| Perangkat | Interface | IP address | Default gateway |
|-----------|-----------|------------|-----------------|
| SW-L1 | Vlan99 | 192.168.99.2/24 | 192.168.99.1 |
| SW-L2 | Vlan99 | 192.168.99.3/24 | 192.168.99.1 |
| SW-L3 | Vlan99 | 192.168.99.4/24 | 192.168.99.1 |
