# VPN-Site-To-Site-Fortigate

**Estudiante:** Juan Francisco Burgos Hiciano  
**Matrícula:** 2023-1981  
**Asignatura:** Seguridad en Redes

📹 Video: (https://youtu.be/e-4W9xGcuUI)

## Descripción

Este laboratorio documenta el proceso de implementación y verificación de una VPN Site-To-Site basada en **IPsec IKEv2** entre dos firewalls FortiGate simulados en GNS3, conectados a través de un Router ISP.

La solución utiliza:

- IPsec para cifrado y protección del tráfico entre sedes.
- IKEv2 con clave precompartida (PSK).
- Dos FortiGate como gateways VPN en cada sitio.
- Cisco IOS como router ISP de tránsito.
- Webterm (Firefox) como hosts de prueba en cada LAN.

## Objetivo

Establecer una conexión VPN Site-To-Site segura entre dos redes LAN separadas geográficamente, permitiendo la comunicación extremo a extremo mediante un túnel IPsec cifrado, y verificar la conectividad mediante traceroute.

## Topología

- webterm-1 (Host LAN A)
- IOL1 (Switch LAN A)
- FortiGate5.6.1-1 (Gateway VPN Sede A)
- Router ISP (Tránsito WAN)
- FortiGate5.6.1-2 (Gateway VPN Sede B)
- IOL2 (Switch LAN B)
- webterm-2 (Host LAN B)

![Topología VPN Site-To-Site](https://github.com/TUUSUARIO/VPN-Site-To-Site/blob/main/VPN_Site-To-Site_Fortigate.png)

## Direccionamiento

| Dispositivo | Interfaz | Dirección IP | Red |
|------------|----------|--------------|-----|
| FortiGate5.6.1-1 | port1 (WAN) | 10.0.0.1/30 | 10.0.0.0/30 |
| FortiGate5.6.1-1 | port2 (LAN) | 192.168.1.1/24 | 192.168.1.0/24 |
| Router ISP | e0/1 | 10.0.0.2/30 | 10.0.0.0/30 |
| Router ISP | e0/2 | 10.0.1.1/30 | 10.0.1.0/30 |
| FortiGate5.6.1-2 | port1 (WAN) | 10.0.1.2/30 | 10.0.1.0/30 |
| FortiGate5.6.1-2 | port2 (LAN) | 192.168.2.1/24 | 192.168.2.0/24 |
| webterm-1 | eth0 | 192.168.1.10/24 | 192.168.1.0/24 |
| webterm-2 | eth0 | 192.168.2.10/24 | 192.168.2.0/24 |

## Parámetros de Seguridad

### IKEv2 (Fase 1)

- Cifrado: AES-256
- Integridad: SHA-256
- Autenticación: Pre-Shared Key (PSK)
- Grupo Diffie-Hellman: 14
- Lifetime: 86400 segundos

### IPsec (Fase 2)

- ESP-AES-256
- ESP-SHA256-HMAC
- Modo Tunnel

## Configuración

### Router ISP

```cisco
enable
configure terminal
hostname ISP

interface e0/1
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface e0/2
 ip address 10.0.1.1 255.255.255.252
 no shutdown

end
write memory
```

### FortiGate5.6.1-1 (Sede A)

#### Interfaces

config system interface
edit "port1"
set mode static
set ip 10.0.0.1 255.255.255.252
set allowaccess ping https ssh
next
edit "port2"
set mode static
set ip 192.168.1.1 255.255.255.0
set allowaccess ping https ssh
next
end

#### Fase 1 — IKE

config vpn ipsec phase1-interface
edit "VPN-to-FGT2"
set interface "port1"
set remote-gw 10.0.1.2
set psksecret "Passw0rd123!"
set ike-version 2
set proposal aes256-sha256
next
end

#### Fase 2 — IPsec

config vpn ipsec phase2-interface
edit "VPN-to-FGT2-P2"
set phase1name "VPN-to-FGT2"
set src-subnet 192.168.1.0/24
set dst-subnet 192.168.2.0/24
next
end

#### Rutas Estáticas

config router static
edit 1
set dst 10.0.1.0 255.255.255.252
set gateway 10.0.0.2
set device "port1"
next
edit 2
set dst 192.168.2.0 255.255.255.0
set device "VPN-to-FGT2"
next
end

#### Políticas de Firewall

config firewall policy
edit 1
set name "LAN1-to-VPN"
set srcintf "port2"
set dstintf "VPN-to-FGT2"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
next
edit 2
set name "VPN-to-LAN1"
set srcintf "VPN-to-FGT2"
set dstintf "port2"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
next
end

### FortiGate5.6.1-2 (Sede B)

#### Interfaces

config system interface
edit "port1"
set mode static
set ip 10.0.1.2 255.255.255.252
set allowaccess ping https ssh
next
edit "port2"
set mode static
set ip 192.168.2.1 255.255.255.0
set allowaccess ping https ssh
next
end

#### Fase 1 — IKE

config vpn ipsec phase1-interface
edit "VPN-to-FGT1"
set interface "port1"
set remote-gw 10.0.0.1
set psksecret "Passw0rd123!"
set ike-version 2
set proposal aes256-sha256
next
end

#### Fase 2 — IPsec

config vpn ipsec phase2-interface
edit "VPN-to-FGT1-P2"
set phase1name "VPN-to-FGT1"
set src-subnet 192.168.2.0/24
set dst-subnet 192.168.1.0/24
next
end

#### Rutas Estáticas

config router static
edit 1
set dst 10.0.0.0 255.255.255.252
set gateway 10.0.1.1
set device "port1"
next
edit 2
set dst 192.168.1.0 255.255.255.0
set device "VPN-to-FGT1"
next
end

#### Políticas de Firewall

config firewall policy
edit 1
set name "LAN2-to-VPN"
set srcintf "port2"
set dstintf "VPN-to-FGT1"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
next
edit 2
set name "VPN-to-LAN2"
set srcintf "VPN-to-FGT1"
set dstintf "port2"
set srcaddr "all"
set dstaddr "all"
set action accept
set schedule "always"
set service "ALL"
next
end

### Hosts (webterm-1 y webterm-2)

```bash
# webterm-1
ip addr add 192.168.1.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.1.1

# webterm-2
ip addr add 192.168.2.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.2.1
```

## Problemas Encontrados

### 1. KVM no disponible en GNS3

**Síntoma:**
KVM acceleration cannot be used (/dev/kvm doesn't exist)

**Causa:**
Hyper-V activo en Windows bloqueaba la virtualización anidada en VMware.

**Solución:**
```powershell
bcdedit /set hypervisorlaunchtype off
Disable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
Disable-WindowsOptionalFeature -Online -FeatureName HypervisorPlatform
```
O alternativamente, deshabilitar KVM en GNS3:
```ini
# ~/.config/GNS3/gns3_server.conf
[Qemu]
enable_kvm = false
```

### 2. Error al asignar IP en webterm

**Síntoma:**
Object "192.168.2.10/24" is unknown, try "ip help"

**Causa:**
Sintaxis incorrecta — los webterm corren bash Linux, no VPCS.

**Solución:**
```bash
ip addr add 192.168.2.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.2.1
```

### 3. Destination Host Unreachable desde webterm-2

**Síntoma:**
From 10.0.0.1 icmp_seq=3 Destination Host Unreachable

**Causa:**
webterm-1 no tenía IP configurada. El FortiGate recibía el paquete del túnel pero no podía entregarlo al host destino.

**Solución:**
```bash
ip addr add 192.168.1.10/24 dev eth0
ip link set eth0 up
ip route add default via 192.168.1.1
```

## Comandos de Verificación

### FortiGate (ambos)
diagnose vpn ike gateway list
get vpn ipsec tunnel summary
get router info routing-table all
show firewall policy
execute ping 192.168.1.10
execute ping 192.168.2.10

### Hosts (webterm)

```bash
ip addr show eth0
ip route show
ping 192.168.1.1
ping 192.168.2.10
traceroute 192.168.2.10
```

## Resultado Obtenido

El túnel VPN Site-To-Site logró establecerse correctamente:

- IKEv2 Fase 1 exitosa.
- IPsec Fase 2 exitosa.
- Rutas estáticas correctamente instaladas en ambos FortiGate.
- Políticas de firewall bidireccionales aplicadas.
- Comunicación exitosa entre webterm-1 y webterm-2.

```text
selectors(total,up): 1/1
rx(pkt,err): 10/0
tx(pkt,err): 10/0
```

## Estado Final

✅ Interfaces WAN y LAN configuradas  
✅ Túnel IKEv2 Fase 1 establecido  
✅ Túnel IPsec Fase 2 establecido  
✅ Rutas estáticas correctas en ambos FortiGate  
✅ Políticas de firewall bidireccionales aplicadas  
✅ Comunicación entre LANs verificada  
✅ Traceroute exitoso entre webterm-1 y webterm-2  

## Conclusión

La VPN Site-To-Site con IPsec IKEv2 fue implementada exitosamente en GNS3 entre dos firewalls FortiGate, tras resolver problemas relacionados con virtualización (KVM), configuración de hosts Linux y diagnóstico de rutas. El túnel cifrado permite la comunicación segura entre las redes 192.168.1.0/24 y 192.168.2.0/24 sin que el Router ISP sea visible en el traceroute, confirmando el correcto encapsulamiento ESP del tráfico.
