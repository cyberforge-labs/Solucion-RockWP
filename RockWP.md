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

### Paso 2: Enumeración de WordPress

Tras identificar que el servicio web corresponde a un WordPress, se procede a realizar una fase de enumeración, cuyo objetivo es obtener información detallada sobre la instalación del CMS, como usuarios, temas, plugins y posibles vectores de ataque.

Para ello, se emplea la herramienta WPScan, especializada en el análisis de seguridad de sitios WordPress.

Identificación de la versión y componentes

Se ejecuta el siguiente comando contra el sitio web:
wpscan --url http://192.168.1.10 --enumerate u,vt,vp

<img width="936" height="191" alt="Captura de pantalla de 2025-12-25 16-49-51" src="https://github.com/user-attachments/assets/7ab5c3ec-3738-4977-b6f7-09df60a92506" />

Donde:

--url: especifica la URL del sitio WordPress objetivo.

--enumerate u: enumera los usuarios registrados.

--enumerate vt: enumera los temas vulnerables.

--enumerate vp: enumera los plugins vulnerables.

Como resultado del escaneo, se obtiene información relevante sobre la instalación de WordPress, obtenemos informacion como que existe un usuario valido en el sistema llamado peter_wp, La enumeración de usuarios es especialmente crítica, ya que facilita ataques de fuerza bruta o credential stuffing contra el panel de administración.

Además, se detecta la presencia de componentes potencialmente vulnerables, lo que indica que el sistema no sigue buenas prácticas de actualización y endurecimiento de seguridad.

Acceso a rutas comunes de WordPress

De forma complementaria, se realiza una comprobación manual accediendo a rutas típicas de WordPress como:

/wp-admin

/wp-login.php

/wp-content/

/wp-includes/

El acceso a estas rutas confirma nuevamente el uso de WordPress y permite verificar si existen restricciones de acceso o mecanismos de protección adicionales, como autenticación reforzada o limitación de intentos de inicio de sesión.

## Paso 3: Ataque de fuerza bruta contra WordPress

Tras la fase de enumeración, se ha identificado la existencia de un usuario válido en WordPress llamado peter_wp. A partir de esta información, se procede a realizar un ataque de fuerza bruta contra el panel de autenticación de WordPress con el objetivo de obtener credenciales válidas.

Preparación del diccionario

Para el ataque se utiliza el diccionario rockyou.txt, ampliamente empleado en auditorías de seguridad. No obstante, en este laboratorio el diccionario se emplea invertido, con el fin de simular un escenario en el que el usuario ha utilizado una contraseña basada en una palabra común pero escrita al revés, una práctica insegura relativamente frecuente.

El diccionario invertido se genera mediante el siguiente comando:
tac /usr/share/wordlist/rockyou.txt > /tmp/rockyou_invertido.txt

De este modo, se obtiene un nuevo diccionario que contiene las mismas palabras que rockyou.txt, pero en orden inverso.

# Ejecución del ataque de fuerza bruta.

Una vez identificado el usuario válido <strong>peter_wp</strong> y preparado el diccionario rockyou invertido, se procede a ejecutar el ataque de fuerza bruta contra el formulario de autenticación de WordPress utilizando la herramienta Hydra, ampliamente empleada para ataques de autenticación en servicios web.

En WordPress, el inicio de sesión se realiza a través del archivo wp-login.php, por lo que el ataque se dirige contra dicho formulario.

El comando utilizado es el siguiente:
```bash
hydra -l peter_wp -P /tmp/rockyou-reversed.txt 192.168.1.10 http-post-form \
"/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&redirect_to=http%3A%2F%2F192.168.1.10%2F:S=302"
```

Donde:

-l peter_wp: especifica el usuario objetivo.

-P /tmp/rockyou-reversed.txt: indica el diccionario de contraseñas invertido utilizado en el ataque.

192.168.1.10: dirección IP del servidor WordPress.

http-post-form: módulo de Hydra para ataques contra formularios web.

/wp-login.php: ruta del formulario de inicio de sesión.

log=^USER^: campo del formulario correspondiente al nombre de usuario.

pwd=^PASS^: campo del formulario correspondiente a la contraseña.

wp-submit=Log In: parámetro enviado al pulsar el botón de inicio de sesión.

redirect_to=...: redirección utilizada por WordPress tras el login.

S=302: indica a Hydra que una respuesta HTTP 302 (redirección) se considera un inicio de sesión exitoso.
