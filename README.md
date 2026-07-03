# VPN Site-to-Site IKEv2 con Túnel GRE sobre IPSec

## Descripción

Este laboratorio implementa una **VPN Site-to-Site Punto a Punto** utilizando **IKEv2**, **IPSec** y un **túnel GRE** entre dos routers Cisco.

El objetivo es permitir la comunicación segura entre dos redes LAN remotas encapsulando el tráfico dentro de un túnel GRE protegido mediante IPSec con autenticación IKEv2.

---

# Objetivos

- Implementar una VPN Site-to-Site.
- Configurar un túnel GRE entre dos routers Cisco.
- Proteger el túnel GRE mediante IPSec.
- Utilizar IKEv2 para el intercambio seguro de claves.
- Permitir la comunicación entre dos redes LAN remotas.

---

# Tecnologías utilizadas

- Cisco IOS
- GRE (Generic Routing Encapsulation)
- IPSec
- IKEv2
- AES-256
- SHA-256
- Diffie-Hellman Grupo 14
- Pre-Shared Key (PSK)

---

# Topología

```
                ISP
        -------------------
        |                 |
23.6.1.2/30         23.6.1.5/30
        |                 |
   Peer1               Peer2
23.6.1.1            23.6.1.6
        \           /
         \         /
          =========
          GRE Tunnel
      10.0.0.1/30
      10.0.0.2/30
         =========
        /           \
23.6.2.0/24      23.6.3.0/24
    LAN1             LAN2
```

---

# Direccionamiento IP

## Enlaces WAN

| Dispositivo | Interfaz | Dirección IP |
|------------|----------|--------------|
| Peer1 | e0/0 | 23.6.1.1/30 |
| ISP | e0/0 | 23.6.1.2/30 |
| ISP | e0/1 | 23.6.1.5/30 |
| Peer2 | e0/0 | 23.6.1.6/30 |

---

## Redes LAN

| Dispositivo | Interfaz | Dirección |
|------------|----------|-----------|
| Peer1 | e0/1 | 23.6.2.1/24 |
| Peer2 | e0/1 | 23.6.3.1/24 |

---

## Túnel GRE

| Dispositivo | Interfaz | Dirección |
|------------|----------|-----------|
| Peer1 | Tunnel0 | 10.0.0.1/30 |
| Peer2 | Tunnel0 | 10.0.0.2/30 |

---

## Equipos finales

### LAN Peer1

| Equipo | Dirección | Gateway |
|---------|-----------|----------|
| VPC1 | 23.6.2.10/24 | 23.6.2.1 |

### LAN Peer2

| Equipo | Dirección | Gateway |
|---------|-----------|----------|
| VPC2 | 23.6.3.10/24 | 23.6.3.1 |

---

# Funcionamiento

El laboratorio utiliza dos mecanismos principales:

## GRE

GRE crea un túnel lógico entre ambos routers.

Su función es encapsular cualquier protocolo IP dentro de un único enlace virtual.

Esto permite transportar múltiples tipos de tráfico entre ambas sedes.

---

## IPSec

GRE por sí solo **no cifra la información**.

Por ello el túnel GRE es protegido mediante IPSec utilizando:

- AES-256
- SHA-256
- Diffie-Hellman Grupo 14
- IKEv2
- Clave compartida (PSK)

Todo el tráfico encapsulado viaja cifrado.

---

# Configuración de IKEv2

Se utiliza una configuración compuesta por:

## Proposal

Define:

- Algoritmo de cifrado
- Algoritmo de integridad
- Grupo Diffie-Hellman

```
AES-256
SHA-256
DH Group 14
```

---

## Policy

Asocia la Proposal que será utilizada durante la negociación IKE.

---

## Keyring

Contiene:

- Dirección IP del peer remoto
- Clave compartida (Pre-Shared Key)

```
Cisco123
```

---

## Profile

Define:

- Método de autenticación
- Asociación del Keyring
- Identidad del peer remoto

---

# Configuración IPSec

El laboratorio utiliza:

## Transform Set

```
ESP-AES-256
ESP-SHA256-HMAC
Modo Transport
```

---

## IPSec Profile

Asocia:

- Transform Set
- Perfil IKEv2

Posteriormente este perfil protege el túnel GRE mediante:

```
tunnel protection ipsec profile VPN_PROFILE
```

---

# Rutas Estáticas

## Peer1

```
23.6.3.0/24
↓

10.0.0.2
```

Ruta por defecto:

```
0.0.0.0/0
↓

23.6.1.2
```

---

## Peer2

```
23.6.2.0/24
↓

10.0.0.1
```

Ruta por defecto:

```
0.0.0.0/0
↓

23.6.1.5
```

---

# Flujo del tráfico

```
PC1
↓

Peer1

↓

GRE

↓

IPSec

↓

ISP

↓

IPSec

↓

GRE

↓

Peer2

↓

PC2
```

Todo el tráfico viaja cifrado entre ambos sitios.

---

# Verificación

## Estado de IKEv2

```
show crypto ikev2 sa
```

Debe aparecer:

```
READY
```

---

## Estado de IPSec

```
show crypto ipsec sa
```

Verificar:

- Paquetes cifrados
- Paquetes descifrados
- Encapsulados
- Desencapsulados

---

## Estado del túnel

```
show interface tunnel0
```

Debe indicar:

```
Tunnel0 is up

Line protocol is up
```

---

## Tabla de rutas

```
show ip route
```

Deben aparecer las redes remotas:

```
23.6.2.0/24

23.6.3.0/24
```

---

## Conectividad

Desde VPC1:

```
ping 23.6.3.10
```

Desde VPC2:

```
ping 23.6.2.10
```

Los pings deben responder correctamente.

---

# Seguridad implementada

- IKEv2
- IPSec
- AES-256
- SHA-256
- Diffie-Hellman Grupo 14
- GRE protegido mediante IPSec
- Autenticación mediante Pre-Shared Key

---

# Ventajas de GRE sobre IPSec

- Permite transportar protocolos de enrutamiento.
- Soporta multicast y broadcast.
- Facilita el uso de OSPF, EIGRP y RIP.
- IPSec proporciona el cifrado mientras GRE encapsula el tráfico.

---

# Archivos incluidos

```
README.md
Configuración ISP
Configuración Peer1
Configuración Peer2
Topología
Capturas de validación
```

---

# Resultado esperado

Al finalizar la configuración:

- El túnel GRE debe encontrarse **UP**.
- La negociación IKEv2 debe completarse correctamente.
- IPSec debe cifrar todo el tráfico entre ambos sitios.
- Los equipos de ambas LAN deben comunicarse mediante **ping**.
- Todo el tráfico entre sedes debe viajar de forma segura a través del túnel GRE protegido con IPSec.
