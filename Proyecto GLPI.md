ENTREGA EL 19/02 -> Sube el PROYECTO FINAL
DockerHub a GitLab (NOVEDAD)
Primero actualizaremos el sistema:

“sudo apt update && sudo apt upgrade -y”

Crearemos la estructura de carpetas para organizar los volúmenes.

Bash

“mkdir -p ~/proyecto-glpi/config ~/proyecto-glpi/db_data”
<img width="1366" height="768" alt="Captura_p3" src="https://github.com/user-attachments/assets/a6652203-c5b6-4dec-9971-ac21b58601d8" />


entraremos dentro

“cd ~/proyecto-glpi”

crearemos el fichero docker_compose.yml:

“sudo nano docker_compose.yml”


y meteremos lo siguiente y guardaremos:

<img width="1363" height="768" alt="Captura_p2" src="https://github.com/user-attachments/assets/0b70d2d6-1037-41ff-b990-b9ae6b516e27" />

arrancamos el proyecto
“docker compose up -d ”

comprobamos que esta activo
“docker-compose ps”

despues de levantarlo entramos con lo siguiente al navegador

“http://localhost:8081/install/install.php”

<img width="1366" height="768" alt="Captura_p4" src="https://github.com/user-attachments/assets/cc87c9fc-e3be-443f-9b73-da30ba17e63d" />

y seguimos la guia de instalacion dandole click a este boton
y nos sale lo siguiente:
<img width="1366" height="768" alt="Captura_p5" src="https://github.com/user-attachments/assets/3b3faf98-1a64-4f6e-bc16-d5937929ae9b" />

metemos las contraseñas:
<img width="1366" height="768" alt="Captura_p6" src="https://github.com/user-attachments/assets/9b4fa0e8-d85e-48ad-badf-b139f8498d27" />

y despues de elegir la base de datos ya tendremos acceso
