# Solución al laboratorio RockWP

|                   | Detalle maquina original                        |
| ----------------- | ----------------------------------------------- |
| Autor             | Jaime Galvez Martinez                                  |
| Dificultad        | Media                                        |
| Fecha de creación | 22/12/2025                                     |
| Fecha del writeup | 25/12/2025                                      |
| Maquina original  | RockWP                                         |

El laboratorio presenta una dificultad media, ya que requiere conocimientos sólidos de enumeración, ataques de fuerza bruta, servicios web y sistemas Linux. Sin embargo, no incluye explotación avanzada ni técnicas de alta complejidad, basándose principalmente en malas configuraciones y malas prácticas de seguridad encadenadas.


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

Este criterio es correcto, ya que WordPress, tras una autenticación válida, responde con un código HTTP 302, redirigiendo al usuario al panel de administración.

Resultados del ataque

Tras ejecutar el ataque, Hydra logra identificar la contraseña válida asociada al usuario <strong>peter_wp</strong>, confirmando que el sistema es vulnerable a ataques de fuerza bruta contra el formulario de autenticación.

<img width="1474" height="259" alt="Captura de pantalla de 2025-12-25 18-33-17" src="https://github.com/user-attachments/assets/1fd8cebc-187f-4631-a9f3-bb0490f3f7f4" />

---------------------------------------------------------------------------------------------------
El éxito del ataque evidencia:

La ausencia de limitación de intentos de login.

La inexistencia de mecanismos de protección como CAPTCHA o bloqueo por IP.

El uso de una contraseña débil basada en un diccionario común, incluso aunque esté invertida.

### Paso 4: Acceso al servicio SSH mediante reutilización de credenciales

Tras obtener credenciales válidas del usuario peter_wp en el panel de administración de WordPress, se procede a comprobar si dichas credenciales han sido reutilizadas en otros servicios del sistema. Este tipo de práctica insegura es habitual y supone un riesgo significativo para la seguridad.

Durante la fase de reconocimiento inicial, se identificó la existencia de un servicio SSH accesible en el puerto 2220, por lo que se intenta el acceso remoto utilizando el mismo usuario y contraseña obtenidos previamente.

### Acceso al servicio SSH

El acceso se realiza mediante el cliente SSH estándar, indicando el puerto no estándar configurado en el servicio:

```bash
ssh peter_wp@192.168.1.10 -p 2220
```

Al introducir la contraseña obtenida durante el ataque de fuerza bruta contra WordPress, el sistema permite el acceso correctamente, proporcionando una shell interactiva en el sistema operativo de la máquina objetivo.

<img width="702" height="547" alt="Captura de pantalla de 2025-12-25 18-47-18" src="https://github.com/user-attachments/assets/7d16554d-8452-4c6f-8d8f-95258e408b17" />


Resultados del acceso

El acceso exitoso al servicio SSH confirma que:

Las credenciales han sido reutilizadas entre diferentes servicios (WordPress y SSH).

No existe una separación adecuada de credenciales entre la aplicación web y el sistema operativo.

El servicio SSH no implementa medidas adicionales de seguridad, como autenticación basada en claves o restricciones por IP.

### Impacto de la vulnerabilidad

La reutilización de contraseñas permite a un atacante:

Obtener acceso directo al sistema operativo sin necesidad de explotar vulnerabilidades adicionales.

Ejecutar comandos con los privilegios del usuario comprometido.

Facilitar ataques posteriores, como la escalada de privilegios hasta obtener control total del sistema.

### Paso 5: Escalada de privilegios mediante configuración insegura de sudo

Una vez obtenido acceso al sistema a través del servicio SSH con el usuario peter_wp, se procede a comprobar si el usuario dispone de privilegios elevados o permisos mal configurados que permitan una escalada de privilegios.

Como primera comprobación, se ejecuta el siguiente comando:

```bash
sudo -i
```

De forma inmediata, el sistema concede acceso a una shell con privilegios de superusuario (root) sin solicitar ninguna confirmación adicional ni contraseña distinta a la del usuario.

Resultados de la escalada de privilegios

La ejecución exitosa de sudo -i indica que el usuario peter_wp:

Pertenece al grupo <strong>sudo</strong> o:

- Tiene permisos configurados en el archivo /etc/sudoers para ejecutar comandos como <strong>root</strong> sin restricciones.

- Esto se confirma al observar que el prompt cambia a <strong>root@</strong>, lo que demuestra que se ha obtenido control total del sistema.

### Paso 6: Obtención de persistencia mediante clave pública SSH

Tras haber conseguido acceso al sistema con privilegios de superusuario (root), el atacante puede implementar mecanismos de persistencia, con el objetivo de mantener el acceso al sistema incluso aunque se cambien contraseñas o se reinicie el servicio.

Una de las técnicas más comunes y efectivas consiste en añadir una clave pública SSH al sistema comprometido.

Obtención y uso de la clave pública SSH

<strong>Desde la máquina atacante</strong>, se dispone de un par de claves SSH previamente generado. En caso de no existir, puede generarse mediante:

```bash
ssh-keygen
```

<img width="622" height="417" alt="Captura de pantalla de 2025-12-25 21-19-04" src="https://github.com/user-attachments/assets/8d521d89-4912-4862-bdd9-f53dc097e189" />

Una vez obtenida la clave pública <strong>id_rsa.pub</strong>,  se añade dicha clave al archivo de claves autorizadas del usuario wp_admin.

Dentro del archivo authorized_keys, se introduce la clave pública SSH del atacante.

<img width="1553" height="187" alt="Captura de pantalla de 2025-12-25 22-06-06" src="https://github.com/user-attachments/assets/5cc35100-d5d9-4b40-b474-efb80a7d2a83" />

Posteriormente, se ajustan los permisos correctos para evitar problemas de acceso:

```bash
chmod 700 /root/.ssh
chmod 600 /root/.ssh/authorized_keys
```

### Verificación de la persistencia

Una vez configurada la clave pública, el atacante puede acceder directamente al sistema como root mediante SSH, sin necesidad de introducir contraseña:

```bash
ssh -i .ssh/id_ed25519 peter_wp@192.168.1.10 -p 2220

```

El acceso exitoso sin solicitud de contraseña confirma que la autenticación por clave SSH ha sido configurada correctamente para el usuario peter_wp, estableciendo así un mecanismo adicional de persistencia.

Impacto de seguridad

Esta configuración permite a un atacante:

Acceder al sistema sin conocer la contraseña del usuario.

Mantener acceso persistente incluso si se cambia la contraseña.

Dificultar la detección del compromiso si no se auditan los archivos authorized_keys.

Este paso refuerza la gravedad del compromiso, ya que el atacante puede utilizar mecanismos legítimos del sistema para mantener el acceso de forma permanente.

### Paso 8: Acceso no autorizado al servicio MySQL sin contraseña

Tras haber obtenido acceso al sistema y comprobar diferentes servicios instalados, se procede a verificar la seguridad del servidor de bases de datos MySQL, ya que este tipo de servicios suelen contener información sensible.

Durante la comprobación, se detecta que el servicio MySQL permite el acceso sin solicitar contraseña, lo que indica una configuración gravemente insegura.

## Acceso al servicio MySQL

Desde la máquina comprometida, se intenta acceder al servicio MySQL utilizando el cliente estándar, sin proporcionar contraseña:

```bash
mysql -u root
```

El acceso se realiza con éxito, obteniendo una consola interactiva de MySQL con privilegios de administrador de base de datos (root), sin que se solicite ningún tipo de autenticación.

Esto confirma que el usuario root de MySQL:

No tiene contraseña configurada,

Está configurado para permitir acceso local sin autenticación.

## Resultados del acceso

Una vez dentro del servicio MySQL, el atacante puede:

Listar todas las bases de datos del sistema.

Acceder a tablas que contienen información sensible.

Modificar, borrar o crear datos.

Crear nuevos usuarios con privilegios elevados.

Este acceso supone el compromiso total de la capa de datos del sistema.

Conclusiones finales del laboratorio

En este laboratorio se ha llevado a cabo el análisis y explotación de una máquina vulnerable, demostrando cómo una serie de malas prácticas de seguridad pueden encadenarse hasta provocar el compromiso total del sistema.

A lo largo del laboratorio se ha seguido una metodología estructurada, comenzando por la fase de reconocimiento y finalizando con la obtención de persistencia y el acceso a servicios críticos sin autenticación.

Resumen de la cadena de ataque

Reconocimiento y escaneo de puertos, identificando servicios expuestos (Apache, WordPress y SSH).

Enumeración de WordPress, obteniendo usuarios válidos.

Ataque de fuerza bruta contra el usuario peter_wp utilizando Hydra y un diccionario invertido.

Acceso al sistema mediante SSH reutilizando las credenciales obtenidas.

Escalada de privilegios directa a root debido a una configuración insegura de sudo.

Obtención de persistencia mediante la instalación de claves públicas SSH.

Acceso persistente sin contraseña al sistema mediante autenticación por clave.

Acceso al servicio MySQL sin contraseña, comprometiendo completamente la base de datos.

### Fin del laboratorio, Esperamos que hayas disfrutado y aprendido;)

Esta secuencia demuestra cómo una vulnerabilidad inicial en una aplicación web puede derivar rápidamente en un compromiso completo de la infraestructura cuando no se aplican medidas básicas de seguridad.

### Vulnerabilidades principales detectadas

Contraseñas débiles y reutilizadas.

Ausencia de protección frente a ataques de fuerza bruta.

Configuración insegura del servicio SSH.

Permisos incorrectos en la configuración de sudo.

Falta de control de accesos mediante claves SSH.

Servicio MySQL accesible sin autenticación.

Falta de políticas de hardening y auditoría.

# Laboratorio: 
# https://cyberforge-labs.byethost7.com/  
# https://cyberforge-labs.a0001.net 
# https://cyberforge-labs.sytes.net
