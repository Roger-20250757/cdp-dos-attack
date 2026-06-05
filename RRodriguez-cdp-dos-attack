# Ataque DoS mediante protocolo CDP

**Autor:** Roger Rodriguez  
**Matrícula:** 20250757  
**Fecha:** Junio 2026  

---

## Objetivo del laboratorio

Demostrar cómo un atacante puede realizar un ataque de Denegación de Servicio (DoS) 
contra dispositivos Cisco mediante el protocolo CDP (Cisco Discovery Protocol), 
inundando la tabla de vecinos CDP del switch con entradas falsas hasta agotar 
sus recursos de memoria y CPU.

---

## Objetivo del script

El script `cdp_dos.py` genera y envía 1000 paquetes CDP falsos con:
- MACs de origen aleatorias
- Device IDs aleatorios
- Información de plataforma y versión falsificada

Esto provoca que el switch almacene cientos de vecinos CDP falsos en su tabla,
consumiendo memoria y degradando el rendimiento del dispositivo.

---

## Parámetros usados

| Parámetro | Valor | Descripción |
|---|---|---|
| `INTERFAZ` | `eth0` | Interfaz de red del atacante |
| `PAQUETES` | `1000` | Cantidad de paquetes CDP falsos |
| `DELAY` | `0.01` | Tiempo entre paquetes (segundos) |
| `DST MAC` | `01:00:0c:cc:cc:cc` | Multicast CDP de Cisco |

---

## Requisitos para utilizar la herramienta

### Software
- Kali Linux (o cualquier distribución Linux)
- Python 3.x
- Librería Scapy

### Instalación de dependencias
```bash
sudo apt update && sudo apt install python3-scapy -y
```

### Verificar instalación
```bash
python3 -c "import scapy; print('Scapy OK')"
```

### Permisos
El script requiere privilegios de root para enviar paquetes raw:
```bash
sudo python3 cdp_dos.py
```

---

## Documentación del funcionamiento del script

### Flujo del ataque

1. Se genera una MAC de origen aleatoria para cada paquete
2. Se construye un frame Ethernet con destino al multicast CDP `01:00:0c:cc:cc:cc`
3. Se agrega el header LLC con DSAP/SSAP `0xAA`
4. Se agrega el header SNAP con OUI Cisco `0x00000C` y protocolo CDP `0x2000`
5. Se construye el payload CDP con TLVs:
   - **Device ID TLV (type 0x0001):** nombre aleatorio del dispositivo
   - **Port ID TLV (type 0x0003):** puerto FastEthernet aleatorio
   - **Capabilities TLV (type 0x0004):** capacidades del dispositivo
6. El paquete se envía por la interfaz `eth0`
7. El switch registra cada paquete como un nuevo vecino CDP

### Diagrama del ataque

```
ATTACKER (Kali)          SW-ACCESS-1          SW-CORE
20.25.7.100              
      |                       |                   |
      |--- CDP falso #1 ----->|                   |
      |--- CDP falso #2 ----->|                   |
      |--- CDP falso #3 ----->|                   |
      |        ...            |                   |
      |--- CDP falso #1000 -->|                   |
      |                       |                   |
      |              Tabla CDP llena              |
      |              CPU/Memoria agotada          |
```

---

## Documentación de la red

### Topología

```
                    Router-GW
                   20.25.7.1/24
                        |
                    SW-CORE
                   /         \
            SW-ACCESS-1    SW-ACCESS-2
            /       \            \
        ATTACKER    PC1          PC2
```

### Interfaces y direccionamiento

| Dispositivo | Interfaz | IP | VLAN |
|---|---|---|---|
| Router-GW | e0/0 | 20.25.7.1/24 | 1 |
| SW-CORE | e0/0 | — | Trunk |
| SW-CORE | e0/1 | — | Trunk |
| SW-CORE | e0/2 | — | Trunk |
| SW-ACCESS-1 | e0/0 | — | Trunk |
| SW-ACCESS-1 | e0/1 | — | VLAN 1 (ATTACKER) |
| SW-ACCESS-1 | e0/2 | — | VLAN 1 (PC1) |
| SW-ACCESS-2 | e0/0 | — | Trunk |
| SW-ACCESS-2 | e0/1 | — | VLAN 1 (PC2) |
| ATTACKER | eth0 | 20.25.7.100/24 | 1 |
| PC1 | eth0 | 20.25.7.10/24 | 1 |
| PC2 | eth0 | 20.25.7.20/24 | 1 |

### VLANs configuradas

| VLAN | Nombre | Uso |
|---|---|---|
| 1 | default | Red principal |
| 10 | SERVERS | Servidores |
| 20 | CLIENTS | Clientes |
| 99 | MGMT | Gestión |

---

## Ejecución del ataque

```bash
# Clonar el repositorio
git clone https://github.com/tu-usuario/cdp-dos-attack

# Entrar al directorio
cd cdp-dos-attack

# Ejecutar el script
sudo python3 cdp_dos.py
```

### Resultado esperado
```
==================================================
 CDP DoS Attack
 Autor    : Roger Rodriguez
 Matricula: 20250757
 Interfaz : eth0
 Paquetes : 1000
==================================================
[*] Iniciando ataque...
[+] Paquetes enviados: 100/1000
[+] Paquetes enviados: 200/1000
...
[+] Paquetes enviados: 1000/1000
==================================================
[*] Ataque finalizado
[+] Enviados : 1000
[-] Errores  : 0
==================================================
```

### Verificar impacto en el switch
```
SW-CORE# show cdp neighbors
SW-CORE# show cdp neighbors detail
```

---

## Contra-medida

### Descripción
Deshabilitar CDP en todos los dispositivos de la red para evitar que el switch
procese paquetes CDP maliciosos.

### Implementación

En **todos los switches y router**:
```
enable
configure terminal
no cdp run
interface e0/0
 no cdp enable
interface e0/1
 no cdp enable
interface e0/2
 no cdp enable
end
write memory
```

### Verificación
```
SW-CORE# show cdp
% CDP is not enabled
```

### ¿Por qué funciona?
Al deshabilitar CDP globalmente con `no cdp run`, el switch deja de procesar
cualquier paquete CDP entrante, ignorando completamente los paquetes maliciosos
del atacante.

---

## Capturas de pantalla

| Captura | Descripción |
|---|---|
| <img width="896" height="514" alt="image" src="https://github.com/user-attachments/assets/4606384c-ed12-44b5-b26f-c68c6ecc1714" /> | Topología en EVE-NG |
| <img width="765" height="796" alt="image" src="https://github.com/user-attachments/assets/c7d4905b-8b78-4341-8aec-2a1eb960419e" />  |Script corriendo con 1000 paquetes enviados |
|<img width="280" height="76" alt="image" src="https://github.com/user-attachments/assets/5dcc988b-34fe-4029-a360-10f72113a738" /> | CDP deshabilitado en SW-CORE |

---

## Referencias

- [Cisco CDP Protocol](https://www.cisco.com/c/en/us/td/docs/ios/12_2/configfun/configuration/guide/ffun_c/fcf006.html)
- [Scapy Documentation](https://scapy.readthedocs.io/)
- [CVE CDP Vulnerabilities](https://nvd.nist.gov/vuln/search/results?query=cisco+cdp)
