# Introducción a Docker

# UD 05. Caso práctico 01

# - Wordpress + MariaDB

# Paso 1: con el siguiente comando crearemos una red para nuestros contenedores :

‘docker network create redwp ‘



# Paso 2: Crearemos el contenedor de MariaDB :

‘docker run -name saramariadb --network redwp -v
/home/sara/mariadbdata:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=sara21 -e
MARIADB_USER=sara -e MARIADB_PASSWORD=sara
MARIADB_DATABASE=saradocker -d mariadb:10.6 ``` [cite: 45, 46]’

# Paso 3: crearemos el contenedor de WordPress

‘docker run -name sarawp-network redwp -p 8080:80 -d wordpress
``` [cite: 64] ‘

# Paso 4: Instalación y configuración web:

abrir navegador y entrar a ‘http://localhost:8080’

dentro del wordpress mete los siguientes datos:

**Nombre de la base de datos:** ‘saradocker’

- **Nombre de usuario:** ‘sara’.
- **Contraseña:** ‘sara’
- **Servidor de la base de datos:** ‘saramariadb’
-

## Paso 5: Migración de MariaDB

Para actualizar la versión de la base de datos sin perder información:

1. **Parar y eliminar el contenedor actual:**
    paramos contenedor
    ‘docker stop saramariadb’
    borramos el contendor


‘docker rm saramariadb’
2.comprobaremos que los datos siguen seguros en
‘/home/sara/mariadbdata’

3. y por ultimo lanzaremos el nuevo contenedor con un comando parecido
    ‘docker run -name saramariadb2 --network redwp -v
    /home/sara/mariadbdata:/var/lib/mysql -e MARIADB_ROOT_PASSWORD=sara21 -e
    MARIADB_USER=sara -e MARIADB_PASSWORD=sara
    MARIADB_DATABASE=saradocker -d mariadb:10.6 ``` [cite: 45, 46]’
