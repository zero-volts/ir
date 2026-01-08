# Raspberry Pi IR Integracion de modulos IR

Este documento registra **todo el proceso real** seguido para lograr **transmisión y recepción infrarroja (IR)** en una Raspberry Pi,

> ⚠️ Importante: este NO es un tutorial. Aquí se documenta la forma que a mi me funciono la conexion y pruebas de los modulos con la raspberry
---

## Paquetes a instalar

Para hacer pruebas se debe isntalar el paquete `v4l-utils` el cual contiene `ir-ctl`

---

## Actualizar config.txt

Para que podamos usar los GPIO es necesario modificar el archivo 
`/boot/firmware/config.txt`

y agregar lo siguiente:
```ini
dtoverlay=gpio-ir,gpio_pin=18       #--> Receptor
dtoverlay=gpio-ir-tx,gpio_pin=23    #--> Emisor
```

### Concepto clave: Device Tree y dtoverlay
> Los dtoverlay permiten mapear pines GPIO físicos a drivers del kernel, los cuales exponen interfaces de software para acceder al hardware.

Luego de estos cambios reiniciar el sistema.

---

## Verificación de dispositivos

```bash
ls -l /dev/lirc*
```

Resultado esperado:
- `/dev/lirc0` → transmisor
- `/dev/lirc1` → receptor

---

## Hardware

### Receptor IR
- Módulo genérico 38kHz (VS1838, KY-022)
- Alimentación: **5V**
- OUT → GPIO18

<p align="center">
  <img src="assets/modules.png" alt="Logo" width="300"/>
</p>


## Conexión modulos

<p align="center">
  <img src="assets/raspberry-pinout.jpg" alt="Logo" width="300"/>
</p>

**Emisor**:
* VCC -> PIN 2
* GND -> PIN 6
* DAT -> PIN 16 -> GPIO 23

**Receptor**:
* VCC -> PIN 3
* GND -> PIN 6
* DAT -> PIN 12 -> GPIO 18


## Recepción y Emision IR – Modo RAW

Para poder recibir y probar en la raspbery si la conexio nesta correcta se debe ejecutar el sigueinte comando en la terminal
```bash
sudo ir-ctl -r -d /dev/lirc1
```

### Explicación del comando
- `ir-ctl` : herramienta de control IR
- `-r` : modo receive (recibir)
- `-d /dev/lirc1` : dispositivo LIRC del receptor

El ejecutar el comando se queda a la escucha y al momento de apuntar y presionar el control
hacia el receptor deberia verse algo como esto:
```
+1290 -403 +1269 -405 +383 -1291 +1271 -403 +1271 -403 +407 -1267 +407 -1269 +425 -1248 ...
```

Cuando queremos grabar una señal debemos hacer lo mismo pero simplemente redireccionando la salida a un archivo

```bash
sudo ir-ctl -r -d /dev/lirc1 > power.raw
```
---

Para transmitir la señal guardada lo hacemos de esta forma
```bash
sudo ir-ctl -d /dev/lirc0 -s power_tx.raw
```

## Loopback (TX → RX) para confirmar emisión

Se realizó una prueba de “loopback”:
- LED emisor apuntando al receptor a **1–2 cm**

Terminal A (RX):
```bash
sudo ir-ctl -r -d /dev/lirc1
```

Terminal B (TX):
```bash
sudo ir-ctl -d /dev/lirc0 -s tx_burst.raw
```