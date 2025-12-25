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

Con esto vemos los puertos abiertos, y que esta corriendo en cada puerto: Obtenemos que, Hay un servidor web APACHE en el puerto 80 y 443, y un servidor SSH en el puerto 2220.

