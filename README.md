# VPN-Site-To-Site-Fortigate

# VPN-Site-To-Site-IPSec-IKEv2-FortiGate-GNS3

**Estudiante:** Juan Francisco Burgos Hiciano  
**Matrícula:** 2023-1981  
**Asignatura:** Seguridad en Redes

📹 Video: https://youtu.be/XXXXXXXXXXXXXXX

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
