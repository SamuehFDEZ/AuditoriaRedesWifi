# 🎭 Ataque Evil Twin contra WPA2-PSK
### Práctica de Seguridad en Redes Inalámbricas

<img width="700" height="auto" alt="image" src="https://github.com/user-attachments/assets/db590a4b-6933-44a6-8bd6-8ec3fdf27ecf" />

<br><br>
> ⚠️ **AVISO LEGAL**: Este documento tiene fines **exclusivamente educativos**. La realización de estos ataques sin autorización explícita sobre redes ajenas es **ilegal** y está penada por la ley. Realizar únicamente en entornos controlados de laboratorio con autorización.
 
---
 
## 📋 Índice
 
1. [¿Qué es un ataque Evil Twin?](#qué-es-un-ataque-evil-twin)
2. [¿Cómo funciona?](#cómo-funciona)
3. [Requisitos previos](#requisitos-previos)
4. [Herramientas necesarias](#herramientas-necesarias)
5. [Instalación de Airgeddon](#instalación-de-airgeddon)
6. [Fases del ataque paso a paso](#fases-del-ataque-paso-a-paso)
7. [Análisis de resultados](#análisis-de-resultados)
8. [Contramedidas y mitigación](#contramedidas-y-mitigación)
---
 
## ¿Qué es un ataque Evil Twin?
 
Un ataque **Evil Twin** (gemelo malvado) consiste en crear un **punto de acceso WiFi falso** que imita exactamente a una red legítima. El atacante replica el SSID (nombre de red) y otros parámetros de la red objetivo para engañar a los usuarios y conseguir que se conecten al AP malicioso.
 
En el contexto de **WPA2-PSK** (Pre-Shared Key), el objetivo específico es:
 
- Forzar a la víctima a desconectarse de la red legítima
- Conseguir que se conecte al AP falso
- Presentar un **portal cautivo** que solicite la contraseña WiFi
- Capturar y verificar la contraseña introducida
---
 
## ¿Cómo funciona?
 
```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO DEL ATAQUE                         │
└─────────────────────────────────────────────────────────────┘
 
  [Red Legítima]          [Atacante]           [Víctima]
  SSID: MiWiFi            SSID: MiWiFi
  Canal: 6                Canal: 11
       │                       │                    │
       │    1. Reconocimiento  │                    │
       │◄──────────────────────│                    │
       │                       │                    │
       │    2. Crea AP Gemelo  │                    │
       │       (mismo SSID)    │                    │
       │                       │                    │
       │    3. Envía deauth    │◄───────────────────│
       │◄──────────────────────│    Conectada a     │
       │    frames a víctima   │    red legítima    │
       │                       │                    │
       │                       │  4. Víctima se     │
       │                       │  desconecta y      │
       │                       │  ve dos redes      │
       │                       │◄───────────────────│
       │                       │  5. Se conecta     │
       │                       │  al gemelo         │
       │                       │                    │
       │              [Portal Cautivo]               │
       │         "Introduce la contraseña WiFi"      │
       │                       │───────────────────►│
       │                       │                    │
       │               6. Captura y verifica         │
       │                  la contraseña              │
```
 
---
 
## Requisitos previos
 
### Hardware
- 💻 Ordenador con **Kali Linux** (o distribución similar orientada a pentesting)
- 📡 Adaptador de red WiFi compatible con **modo monitor** y **packet injection**
  - Recomendados: Alfa AWUS036ACH, Alfa AWUS036NHA, TP-Link TL-WN722N (v1)
### Software
- Sistema operativo: **Kali Linux** (recomendado) / Parrot OS
- Privilegios de **root**
- Conexión a internet para instalar dependencias
### Conocimientos previos
- Conceptos básicos de redes WiFi (SSID, BSSID, canales, cifrado)
- Familiaridad con la terminal de Linux
- Comprensión del protocolo WPA2-PSK
---
 
## Herramientas necesarias
 
| Herramienta | Función | Incluida en Airgeddon |
|-------------|---------|----------------------|
| **Airgeddon** | Framework principal del ataque | ✅ (gestiona todo) |
| **aircrack-ng** | Suite de auditoría WiFi | ✅ Dependencia |
| **hostapd-wpe** | Creación del AP falso | ✅ Dependencia |
| **dnsmasq** | Servidor DHCP/DNS para el AP | ✅ Dependencia |
| **lighttpd / apache2** | Servidor web del portal cautivo | ✅ Dependencia |
| **isc-dhcp-server** | Servidor DHCP alternativo | ✅ Dependencia |
| **macchanger** | Cambio de dirección MAC | ✅ Dependencia |
 
---
 
## Instalación de Airgeddon
 
### Opción 1: Clonar desde GitHub
 
```bash
# Clonar el repositorio
git clone https://github.com/v1s1t0r1sh3r3/airgeddon.git
 
# Acceder al directorio
cd airgeddon
 
# Dar permisos de ejecución
chmod +x airgeddon.sh
 
# Ejecutar como root
sudo bash airgeddon.sh
```
 
### Opción 2: Instalación en Kali Linux (paquete)
 
```bash
# Actualizar repositorios
sudo apt update
 
# Instalar airgeddon
sudo apt install airgeddon -y
 
# Ejecutar
sudo airgeddon
```
 
### Verificar dependencias
 
Al iniciar Airgeddon por primera vez, el propio script verifica automáticamente todas las dependencias y ofrece instalar las que falten:
 
```
[*] Checking tools...
 airmon-ng ........ OK
 airodump-ng ...... OK
 aireplay-ng ...... OK
 hostapd .......... OK
 dnsmasq .......... OK
 lighttpd ......... OK
 ...
```
 
---
 
## Fases del ataque paso a paso
 
### 🔍 FASE 1 — Reconocimiento y configuración inicial
 
**Paso 1.1 — Iniciar Airgeddon**
 
```bash
sudo bash airgeddon.sh
```
 
Al arrancar, Airgeddon muestra un menú de bienvenida y solicita seleccionar la interfaz de red a utilizar.
 
**Paso 1.2 — Seleccionar la interfaz WiFi**
 
```
Select an option: [número de tu interfaz, ej: wlan0]
```
 
**Paso 1.3 — Poner la interfaz en modo monitor**
 
Desde el menú principal, seleccionar:
 
```
> 2) Put interface in monitor mode
```
 
El adaptador pasará a llamarse `wlan0mon`.
 
---
 
### 📡 FASE 2 — Escaneo de redes objetivo
 
**Paso 2.1 — Acceder al menú Evil Twin**
 
En el menú principal seleccionar:
 
```
> 7) Evil Twin attacks menu
```
A continuación, seleccionar la opción del portal cautivo:
```
> 9) Evil Twin portal cautivo
```

**Paso 2.2 — Escanear redes disponibles**
 
```
> 1) Escaneo de interfazes (modo monitor requerido)
```
 
Airgeddon lanzará `airodump-ng` y mostrará todas las redes WiFi cercanas. Esperar **al menos 20-30 segundos** para capturar suficiente información, es importante tener en cuenta que cuanto más tiempo pase, más canales se obtendrán y se podrán escanear redes wifi con número de canales elevados (columna CH).
 
```
 BSSID              PWR  Beacons  #Data  CH  MB   ENC   ESSID
 AA:BB:CC:DD:EE:FF  -45      120    230   6  130  WPA2  MiRedWiFi
 11:22:33:44:55:66  -72       45     12  11   54  WPA2  OtraRed
```
 
**Paso 2.3 — Seleccionar la red objetivo**
 
Pulsar `Ctrl+C` para detener el escaneo y seleccionar el número correspondiente a la red objetivo. Airgeddon guardará automáticamente:
- BSSID (MAC del AP)
- SSID (nombre de la red)
- Canal
---
 
### 💥 FASE 3 — Ataque de desautenticación (Deauth)
 
**Paso 3.1 — Seleccionar el tipo de ataque Evil Twin**
 
```
> 9) Evil Twin AP attack with captive portal
```
 
Esta opción crea un AP gemelo con portal cautivo que solicita la contraseña WPA.
 
**Paso 3.2 — Configurar el ataque de deautenticación para la obtención del handshake**
 
Airgeddon permite elegir el método de desautenticación:
 
```
Do you want to use deauth / disassoc mdk attacks? [y/n]: n
```
En este caso le decimos que no ya que se requeriría de otra interfaz aparte de la `wlan0mon`
Seleccionar el tipo:
```
> 2) aireplay-ng deauth
```
 
> 💡 **Recomendación**: `mdk4` es más efectivo para expulsar a todos los clientes de la red objetivo, aunque basta con aireplay-ng para esta práctica.

**Paso 3.3 — Seleccionar la existencia de handshake**
 
```
Do you have a handshake file for being used? Y/n
> n
> ...
```
Decimos que no para obtener el fichero

**Paso 3.4 — Seleccionar timeout del ataque de deautenticación**
 
```
Select the timeout for capturing the handshake [20]:
> 
```
Podemos no escribir nada y airggedon tomará los 20 segundos propuestos de timeout
---

**Paso 3.5 —Obtención del handshake**
 
```
 CH 7                           [WPA HANDSHAKE: AA:BB:CC:DD:EE:FF]
 BSSID              PWR  Beacons  #Data  CH  MB   ENC   ESSID
 AA:BB:CC:DD:EE:FF  -45      120    230   6  130  WPA2  MiRedWiFi
 11:22:33:44:55:66  -72       45     12  11   54  WPA2  OtraRed
```
Airggedon nos avisará de la captura del handshake con el mensaje WPA HANDSHAKE: BSSID, luego nos pedirá donde guardar el fichero de extensión cap
---

 
### 🌐 FASE 4 — Creación del AP gemelo y portal cautivo
 
**Paso 4.1 — Airgeddon levanta automáticamente:**
 
- Un AP falso con el mismo **SSID** que la red objetivo
- Un servidor **DHCP** para asignar IPs a los clientes
- Un servidor **DNS** que redirige todo el tráfico
- Un **servidor web** con el portal cautivo
- El proceso de **deautenticación** contra la red legítima
La víctima verá en su dispositivo dos redes con el mismo nombre, la señal de la red original debilitada por el ataque deauth, y al conectarse al AP gemelo, será redirigida al portal cautivo.
 
**Paso 4.2 — Portal cautivo**
 
El portal cautivo muestra una página que imita la interfaz del router o una pantalla genérica solicitando la contraseña WiFi:
 
```
┌────────────────────────────────────┐
│  🔒 Red: MiRedWiFi                 │
│                                    │
│  Su router necesita verificación.  │
│  Introduzca la contraseña WiFi:    │
│                                    │
│  [________________________]        │
│                                    │
│         [ Conectar ]               │
└────────────────────────────────────┘
```
 
---
 
### ✅ FASE 5 — Captura y verificación de la contraseña
 
**Paso 5.1 — Verificación automática**
 
Cuando la víctima introduce la contraseña, Airgeddon la verifica automáticamente contra el **handshake WPA2** previamente capturado (o en tiempo real):
 
```
[*] Password verification in progress...
[+] Password found: M1Contr4señ4Segur4
[*] Saving password to file: /root/evil_twin_result.txt
```
 
> Airgeddon verifica la contraseña usando `aircrack-ng` internamente antes de mostrarla como válida, evitando falsos positivos.
 
**Paso 5.2 — Resultado**
 
Una vez verificada la contraseña:
- El portal cautivo muestra un mensaje de éxito a la víctima
- La víctima es redirigida o reconectada
- Airgeddon guarda la contraseña en un fichero de log
---
 
## Análisis de resultados
 
### Ficheros generados
 
| Fichero | Contenido |
|---------|-----------|
| `/root/evil_twin_result.txt` | Contraseña capturada y verificada |
| `/tmp/hostapd.conf` | Configuración del AP malicioso |
| `/tmp/dnsmasq.conf` | Configuración DNS/DHCP |
| Capturas `.cap` | Handshakes WPA2 capturados |
 
### Indicadores de éxito del ataque
 
```
✅ AP gemelo activo en el mismo canal
✅ Víctima conectada al AP gemelo
✅ Víctima ha introducido la contraseña
✅ Contraseña verificada correctamente
```
 
---
 
## Contramedidas y mitigación
 
Desde el punto de vista **defensivo**, estas son las medidas para protegerse contra ataques Evil Twin:
 
### Para usuarios
- ✅ Verificar el certificado del portal cautivo si aparece uno inesperado
- ✅ Desconfiar de portales que soliciten la contraseña WiFi (los routers legítimos no lo hacen así)
- ✅ Usar VPN en redes públicas o desconocidas
- ✅ Activar alertas de red en el dispositivo
### Para administradores de red
- ✅ Usar **WPA2-Enterprise** (802.1X) en lugar de PSK en entornos corporativos
- ✅ Implementar sistemas **WIDS** (Wireless Intrusion Detection System)
- ✅ Monitorizar la presencia de APs con el mismo SSID en la red
- ✅ Configurar clientes para validar el BSSID conocido del AP legítimo
- ✅ Usar **802.11w** (Management Frame Protection) para proteger frames de gestión contra deauth
---
 
## 📚 Referencias
 
- [Airgeddon - GitHub Oficial](https://github.com/v1s1t0r1sh3r3/airgeddon)
- [Aircrack-ng Suite](https://www.aircrack-ng.org/)
- [IEEE 802.11 Standard](https://standards.ieee.org/ieee/802.11/)
- [OWASP - Wireless Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
---
 
<div align="center">
**Práctica realizada con fines educativos en entorno de laboratorio controlado**
 
*Seguridad en Redes Inalámbricas — Evil Twin WPA2-PSK*
 
</div>
