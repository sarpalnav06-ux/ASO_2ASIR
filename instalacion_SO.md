# Instalación del Sistema Operativo – Proyecto 2 ASIR 2025/2026

## Planificación de la Infraestructura

### Recursos del servidor físico

| Recurso | Total disponible |
|---------|-----------------|
| RAM | 64 GB |
| Almacenamiento | 250 GB |
| vCPUs | 32 |

### Máquinas Virtuales planificadas

| VM | Sistema Operativo | RAM | vCPU | Disco | IP |
|----|-------------------|-----|------|-------|----|
| VM1 – Servidor Docker | Ubuntu Server 24.04 LTS | 16 GB | 8 | 80 GB | 192.168.1.20 |
| VM2 – Active Directory | Windows Server 2022 | 4 GB | 2 | 50 GB | 192.168.1.10 |

### Contenedores Docker (en VM1)

| Contenedor | Imagen | Puerto |
|------------|--------|--------|
| MySQL | mysql:8 | 3306 |
| PHP + Apache | php:8.2-apache | 80 |
| phpMyAdmin | phpmyadmin | 8080 |
| Grafana | grafana/grafana | 3000 |
| Prometheus | prom/prometheus | 9090 |

---

## VM1 – Ubuntu Server 24.04 LTS

### Paso 1 – Descargar la ISO

Descargar Ubuntu Server 24.04 LTS desde la página oficial:

```
https://ubuntu.com/download/server
```

### Paso 2 – Subir la ISO a Proxmox

1. Acceder a la interfaz web de Proxmox: `https://IP_PROXMOX:8006`
2. Ir a: `local` → `ISO Images` → `Upload`
3. Seleccionar el archivo `.iso` descargado y subir

### Paso 3 – Crear la VM en Proxmox

1. Clic en `Create VM`
2. Rellenar los campos:

| Campo | Valor |
|-------|-------|
| Name | `Ubuntu-Docker` |
| OS | Linux (Ubuntu) |
| ISO | ubuntu-24.04-live-server-amd64.iso |
| CPU | 8 vCPUs |
| RAM | 16384 MB |
| Disco | 80 GB – SCSI |
| Red | vmbr0 |

3. Revisar resumen y hacer clic en `Finish`

### Paso 4 – Instalar Ubuntu Server

1. Arrancar la VM → abrir `Console`
2. Seleccionar idioma: **Spanish** (o English, recomendado para entorno técnico)
3. Actualizar el instalador si lo solicita
4. Configuración de red: DHCP (configuraremos IP estática después)
5. En `Storage configuration` → seleccionar disco completo, sin LVM si se quiere simple
6. Configurar usuario:

| Campo | Valor sugerido |
|-------|---------------|
| Nombre | `adminp2` |
| Nombre del servidor | `vm1-docker` |
| Contraseña | (segura, mínimo 12 caracteres) |

7. **Habilitar OpenSSH** cuando lo pregunte el instalador ✓
8. No instalar snaps adicionales, clic en `Done`
9. Esperar a que finalice la instalación y reiniciar

### Paso 5 – Configurar IP estática

Editar el archivo de red:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Contenido:

```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: false
      addresses:
        - 192.168.1.20/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 192.168.1.10   # DNS → AD Windows Server
          - 8.8.8.8
```

Aplicar cambios:

```bash
sudo netplan apply
```

### Paso 6 – Actualizar el sistema

```bash
sudo apt update && sudo apt upgrade -y
```

### Paso 7 – Instalar Docker y Docker Compose

```bash
# Instalar dependencias
sudo apt install -y ca-certificates curl gnupg

# Añadir clave GPG oficial de Docker
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Añadir repositorio
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Añadir usuario al grupo docker
sudo usermod -aG docker $USER
newgrp docker

# Verificar instalación
docker --version
docker compose version
```

---

## VM2 – Windows Server 2022

### Paso 1 – Descargar la ISO de evaluación

Acceder al portal de evaluación de Microsoft y descargar la ISO:

```
https://www.microsoft.com/es-es/evalcenter/evaluate-windows-server-2022
```

> Rellenar el formulario de registro. La descarga es gratuita (180 días de evaluación).

### Paso 2 – Subir la ISO a Proxmox

1. Proxmox → `local` → `ISO Images` → `Upload`
2. Seleccionar la ISO de Windows Server 2022

### Paso 3 – Crear la VM en Proxmox

1. Clic en `Create VM`
2. Rellenar los campos:

| Campo | Valor |
|-------|-------|
| Name | `WinServer-AD` |
| OS | Windows 2022 |
| ISO | Windows Server 2022 Evaluation |
| CPU | 2 vCPUs |
| RAM | 4096 MB |
| Disco | 50 GB – SCSI |
| Red | vmbr0 |

3. Revisar y hacer clic en `Finish`

### Paso 4 – Instalar Windows Server 2022

1. Arrancar la VM → abrir `Console`
2. Seleccionar idioma y distribución de teclado (Español)
3. Clic en `Instalar ahora`
4. Elegir edición: **Windows Server 2022 Standard (Desktop Experience)**
5. Aceptar términos de licencia
6. Seleccionar `Instalación personalizada`
7. Seleccionar el disco y continuar
8. Esperar la instalación y reinicio automático
9. Establecer contraseña del Administrador (mínimo 12 caracteres, mayúsculas, números y símbolos)

### Paso 5 – Configurar IP estática (dentro de Windows)

1. Abrir `Panel de control` → `Centro de redes` → `Cambiar configuración del adaptador`
2. Clic derecho en el adaptador → `Propiedades`
3. Seleccionar `Protocolo de Internet versión 4 (TCP/IPv4)` → `Propiedades`
4. Configurar:

| Campo | Valor |
|-------|-------|
| Dirección IP | 192.168.1.10 |
| Máscara de subred | 255.255.255.0 |
| Puerta de enlace | 192.168.1.1 |
| DNS preferido | 127.0.0.1 |
| DNS alternativo | 8.8.8.8 |

### Paso 6 – Instalar rol Active Directory

Abrir **PowerShell como Administrador**:

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

### Paso 7 – Promover a controlador de dominio

```powershell
Install-ADDSForest `
  -DomainName "proyecto2.local" `
  -DomainNetbiosName "PROYECTO2" `
  -InstallDns:$true `
  -Force:$true
```

> Se pedirá contraseña DSRM. El servidor se reiniciará automáticamente al finalizar.

### Paso 8 – Crear estructura de OUs y grupos

```powershell
# Unidades Organizativas
New-ADOrganizationalUnit -Name "Proyecto2" -Path "DC=proyecto2,DC=local"
New-ADOrganizationalUnit -Name "Usuarios" -Path "OU=Proyecto2,DC=proyecto2,DC=local"
New-ADOrganizationalUnit -Name "Tecnicos" -Path "OU=Proyecto2,DC=proyecto2,DC=local"
New-ADOrganizationalUnit -Name "Administradores" -Path "OU=Proyecto2,DC=proyecto2,DC=local"
New-ADOrganizationalUnit -Name "Grupos" -Path "OU=Proyecto2,DC=proyecto2,DC=local"

# Grupos de seguridad
New-ADGroup -Name "GRP_Usuarios"  -GroupScope Global -Path "OU=Grupos,OU=Proyecto2,DC=proyecto2,DC=local"
New-ADGroup -Name "GRP_Tecnicos"  -GroupScope Global -Path "OU=Grupos,OU=Proyecto2,DC=proyecto2,DC=local"
New-ADGroup -Name "GRP_Admins"    -GroupScope Global -Path "OU=Grupos,OU=Proyecto2,DC=proyecto2,DC=local"
```

### Paso 9 – Crear usuarios de prueba

```powershell
# Administrador
New-ADUser -Name "admin" -SamAccountName "admin" `
  -UserPrincipalName "admin@proyecto2.local" `
  -Path "OU=Administradores,OU=Proyecto2,DC=proyecto2,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Admin1234!" -AsPlainText -Force) `
  -Enabled $true
Add-ADGroupMember -Identity "GRP_Admins" -Members "admin"

# Técnico
New-ADUser -Name "tecnico1" -SamAccountName "tecnico1" `
  -UserPrincipalName "tecnico1@proyecto2.local" `
  -Path "OU=Tecnicos,OU=Proyecto2,DC=proyecto2,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Tecnico1234!" -AsPlainText -Force) `
  -Enabled $true
Add-ADGroupMember -Identity "GRP_Tecnicos" -Members "tecnico1"

# Usuario normal
New-ADUser -Name "usuario1" -SamAccountName "usuario1" `
  -UserPrincipalName "usuario1@proyecto2.local" `
  -Path "OU=Usuarios,OU=Proyecto2,DC=proyecto2,DC=local" `
  -AccountPassword (ConvertTo-SecureString "Usuario1234!" -AsPlainText -Force) `
  -Enabled $true
Add-ADGroupMember -Identity "GRP_Usuarios" -Members "usuario1"
```

### Paso 10 – Abrir puerto LDAP en el firewall

```powershell
New-NetFirewallRule -DisplayName "LDAP" -Direction Inbound -Port 389 -Protocol TCP -Action Allow
```

---

## Verificación final

| Comprobación | Comando |
|---|---|
| VM1 – Docker instalado | `docker --version` |
| VM1 – Docker Compose instalado | `docker compose version` |
| VM1 – Conectividad con AD | `ping 192.168.1.10` |
| VM2 – AD activo | `Get-ADDomain` (PowerShell) |
| VM2 – LDAP escuchando | `netstat -an \| findstr :389` |

---

*Proyecto Intermodular 2º ASIR – 2025/2026 | proyecto2.local*
