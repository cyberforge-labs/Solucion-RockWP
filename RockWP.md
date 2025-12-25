# Solición al laboratorio RockWP

|                   | Detalle maquina original                        |
| ----------------- | ----------------------------------------------- |
| Autor             | Jaime Galvez Martinez                                  |
| Dificultad        | Media                                        |
| Fecha de creación | 22/12/2025                                     |
| Fecha del writeup | 25/12/2025                                      |
| Maquina original  | RockWP                                         |


### Paso 1: Reconocimiento - Escaneo de Puertos

 nmap -p- -sV 192.168.1.10

<img width="624" height="366" alt="reconocimientoRockWP" src="https://github.com/user-attachments/assets/b8d21fcf-9869-4f9b-981f-f4916d39018e" />

Este comando permite:

-p-: escanear todos los puertos TCP (1–65535).

-sV: detectar la versión de los servicios que se están ejecutando.

192.168.1.10: dirección IP de la máquina vulnerable.

Como resultado del escaneo, se identifican los siguientes servicios activos:

Puerto 53/TCP: Servidor DNS (De Momento, no nos intersa)

Puerto 80/TCP: Servidor web Apache (HTTP).

Puerto 443/TCP: Servidor web Apache con HTTPS.

Puerto 2220/TCP: Servicio SSH, configurado en un puerto no estándar.

<img width="1280" height="826" alt="Captura de pantalla de 2025-12-25 16-30-20" src="https://github.com/user-attachments/assets/d769fdc0-47d0-44be-a1bf-099be600dfc4" />

A continuación, se accede al servicio web a través del navegador utilizando la dirección IP de la máquina objetivo. Al analizar el contenido del sitio web, se observa que la aplicación utilizada es un WordPress, lo cual se identifica por la estructura del sitio, los recursos cargados y las rutas típicas del CMS.

La detección de WordPress resulta especialmente relevante desde el punto de vista de la seguridad, ya que se trata de una plataforma ampliamente utilizada y, si no está correctamente configurada o actualizada, puede presentar vulnerabilidades conocidas explotables.
