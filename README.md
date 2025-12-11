# Lab 1 — IEEE 802.15.4 Fundamentals (PHY/MAC/Sniffing)

**Autor:** Felipe Idárraga Quintero 

**Nombre de la Práctica:** Laboratorio 1 802.15.4 

**Curso:** Desarrollo de Sistemas IoT

**Departamento:** Departamento de Ingeniería Electrica, Electronica y Computacion

---

## Objetivos
- Configurar dos dispositivos ESP32-C6 para comunicación punto a punto usando IEEE 802.15.4.
- Designar un dispositivo A como coordinador (modo recepción) y un dispositivo B como nodo (modo transmisión).
- Enviar tramas de datos desde el nodo B hacia el coordinador A y observar la recepción mediante la interfaz CLI.
- Interpretar de forma básica las tramas recibidas (tipo, dirección, contenido) sin profundizar aún en toda la estructura del estándar.

## Contexto
Este laboratorio se centra en lograr la primera comunicación práctica entre dos dispositivos usando IEEE 802.15.4 sobre ESP32-C6.
A partir del ejemplo CLI incluido en ESP-IDF, se configura un nodo como coordinador receptor y otro como nodo transmisor, permitiendo enviar y recibir datos en un canal 2.4 GHz.
El objetivo no es todavía analizar todos los tipos de tramas del estándar, sino comprender cómo configurar los parámetros básicos (canal, PAN ID, direcciones) y verificar que la comunicación entre A (coordinador) y B (nodo) funciona correctamente.

## Setup del Proyecto

### 1. Crear proyecto desde ejemplo ESP-IDF

Usa la extensión ESP-IDF en VS Code:
1. Presiona `Ctrl+Shift+P` para abrir la paleta de comandos.
2. Busca y ejecuta `ESP-IDF: Show Examples` seleccionando la versión del ESP-IDF.
3. Selecciona `ieee802154/ieee802154_cli` (IEEE802.15.4 Command Line Example).
4. Seleccione la carpeta para crear el proyecto.

### 2. Explorar el código del ejemplo IEEE 802.15.4

El ejemplo `ieee802154/ieee802154_cli` incluye código para comunicación básica IEEE 802.15.4. Examina los archivos principales en el directorio del proyecto creado:

- `main/esp_ieee802154_cli.c`: Punto de entrada del ejemplo CLI
- `components/cmd_ieee802154/ieee802154_cmd.c`: Implementación de comandos CLI

### 3. Build y flash del proyecto

El ejemplo usa configuraciones por defecto adecuadas para IEEE 802.15.4, incluyendo parámetros físicos como desviación de frecuencia según el estándar. No se requieren cambios en sdkconfig.

Usa la barra de herramientas ESP-IDF en VS Code:
1. Haz clic en **ESP-IDF: Establecer Objetivo** y selecciona `esp32c6`.
2. Haz clic en **ESP-IDF: Construir Proyecto**.
3. Conecta la ESP32-C6 y haz clic en **ESP-IDF: Flashear Dispositivo**.
4. Haz clic en **ESP-IDF: Monitorear Dispositivo**.

### 4. Explorar parámetros IEEE 802.15.4

Una vez en la consola del dispositivo, usa los comandos CLI del ejemplo (todos los comandos están documentados en `help`):

```bash
# Ver ayuda completa
help

# Configurar canal (11-26)
channel -s 15

# Ver canal actual
channel -g

# Configurar potencia de transmisión (-80 a -10 dBm)
txpower -s 10

# Ver potencia actual
txpower -g

# Configurar PAN ID
panid 0x1234

# Ver PAN ID
panid -g

# Configurar dirección corta
shortaddr 0x0001

# Ver dirección corta
shortaddr -g

# Configurar dirección extendida
extaddr 0xaa 0xbb 0xcc 0xdd 0x00 0x11 0x22 0x33


# Ver dirección extendida
extaddr -g
```

![Imagen de WhatsApp 2025-12-11 a las 08 32 12_1323bc7c](https://github.com/user-attachments/assets/2403e7a0-a905-42a7-a2e0-bc3ae9071612)

### 5. Comunicación entre dispositivos

**Configurar Dispositivo A (Coordinador):**
```bash
# Configurar PAN ID y direcciones
panid 0x1234
shortaddr 0x0001
channel -s 15

# Entrar en modo recepción
rx -r 1
```

![Imagen de WhatsApp 2025-12-11 a las 08 37 11_734c87e5](https://github.com/user-attachments/assets/4cb72bf9-ab72-4ff5-b51c-f95984c4daff)


**Configurar Dispositivo B (Nodo):**
```bash
# Configurar misma PAN ID, dirección diferente
panid 0x1234
shortaddr 0x0002
channel -s 15

# Transmitir datos al dispositivo A
tx 0x00 0x01 0x02 0x03  # Datos de ejemplo
```

![Imagen de WhatsApp 2025-12-11 a las 08 34 53_a7ed9ec1](https://github.com/user-attachments/assets/cfd27dc8-84d3-4a15-8fe1-f1d26c741a9b)

## Comparación entre Datos Transmitidos y Datos Capturados

En el dispositivo B (nodo) se envía una trama con el comando `tx 0x00 0x01 0x02 0x03`, es decir, 4 bytes de datos (`00 01 02 03`), y el log confirma la transmisión con el mensaje **“Tx Done 4 bytes”**.

En el dispositivo A (coordinador), que está en modo recepción (`rx -r 1`), se observa la llegada de esa trama con los mensajes **“rx sfd done”** y **“Rx Done 4 bytes”**, mostrando los bytes recibidos `00 01 a9 0b 00 00 00 00`.

La diferencia entre los valores enviados y los vistos en A se debe a que la pila IEEE 802.15.4 agrega campos de cabecera y control a la trama, por lo que el coordinador no muestra únicamente la carga útil enviada, sino una combinación de encabezado + datos.
