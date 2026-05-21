# Proyecto para hacer en NAVIDAD

```
El proyecto consta de hacer un active directory pero como mi portátil tiene limitaciones
hardware (8 gb) de ram he decidido hacel la instalación desde virtual box en vez de hiperV
Una vez instalado nos saldrá la contraseña y después esta pantalla y nos saldrá esto
```
## PONERLE NOMBRE

```
Selecionamos la opcion 2 para ponerle el nombre :
```
y despues le cambiamos el nombre:


una vez alla reinicado hacemos lo siguiente:

## ADAPTADOR DE RED

Escribe el número **8** (Configuración de red) y presiona **Enter**.

- Te mostrará tu tarjeta de red con un número de índice (normalmente es el **1** o el 0 ). Escribe
    ese número de índice y dale a **Enter**.
- Ahora elige la opción **1** (Establecer dirección del adaptador de red) y presiona **Enter**.
- Te preguntará si es DHCP o Estática. Escribe **E** (Estática) y presiona **Enter**.
- Introduce los datos uno a uno cuando te los pida:
- **Dirección IP:** 10.10.10.
- **Máscara de subred:** 255.255.255.
- **Puerta de enlace predeterminada:** 10.0.2.2 (esta es la que usa VirtualBox para darte
salida a internet).
- Volverás al submenú de red. Ahora elige la opción **2** (Establecer servidores DNS) y presiona
    **Enter**.
- **DNS Preferido:** Escribe 127.0.0.1 y dale a Enter.
- **DNS Alternativo:** Escribe 8.8.8.8 y dale a Enter.
- Escribe el número **4** (Volver al menú principal) y presiona **Enter**.

La configuracion debe de quedar tal que asi:


## configuracion active directory:

- Ahora, ejecuta este segundo comando para crear tu dominio tal y como lo pide el ejercicio:

PowerShell

Install-ADDSForest -DomainName "tailwindtraders.internal"
-SafeModeAdministratorPassword (convertto-securestring "Pa55w.rdPa55w.rd"
-asplaintext -force) -Force


- Te hará una pregunta de confirmación en la pantalla. Presiona la tecla **S** (o A para Sí a todo)
    y dale a **Enter**.

## CONFIGURAR OPERACIONES DE

## CONTROLADOR DE DOMINIO

**como el ordenador esta falto de recursos modificamos las reglas del proyceto para que
funcione en tus 8 GB de RAM** , por lo que no tenemos la máquina TAILWIND-MBR1 si no que
todo dentro de una única máquina virtual (TAILWIND-DC1) de la siguiente manera.

Entra a powershell y escribe lo siguiente:

Get-ADDomain | Select-Object RIDMaster, InfrastructureMaster, PDCEmulator


### Crear el Sitio de Active Directory ("Sydney") y configurar la

### Subred

En la versión gráfica usarías ventanas, pero en tu versión optimizada para 8 GB de RAM lo

haremos con dos líneas de comandos ultra rápidas de PowerShell.

Asegúrate de seguir en la consola de powershell (letras blancas con PS al principio) y ejecuta lo

siguiente:

#### 2.1 Crear el nuevo sitio llamado "Sydney"

```
1.Copia, escribe y ejecuta este comando para crear el sitio asociado al enlace por defecto:
```
```
PowerShell
New-ADReplicationSite -Name "Sydney"
```
```
2.Pulsa Enter. Si no muestra ningún error rojo, el sitio se ha creado correctamente en la base
de datos de Active Directory.
```
#### 2.2 Crear la subred y asignarla al sitio "Sydney"

```
1.El ejercicio pide mapear la red 172.16.1.0/24 a este nuevo sitio geográfico. Ejecuta
este comando:
```
```
PowerShell
New-ADReplicationSubnet -Name "172.16.1.0/24" -Site "Sydney"
```
##### 2.

#### 2.3 Verificar que todo se creó con éxito

```
1.Para asegurarte de que los datos quedaron guardados tal y como los pide el laboratorio,
ejecuta este comando de verificación:
```
```
PowerShell
Get-ADReplicationSubnet -Filter * | Select-Object Name, Site
```
```
2.Verás en la pantalla una pequeña tabla donde se confirma que la subred 172.16.1.0/
está oficialmente unida al sitio Sydney.
```

## CREAR CARPETAS ORGANIZATIVAS

##### EJECUTA LO SIGUIENTE EN POWER SHELL PARA CREARLAS

```
New-ADOrganizationalUnit -Name "Sydney" -Path "DC=tailwindtraders,DC=internal"
```
```
3.New-ADOrganizationalUnit -Name "Melbourne" -Path "DC=tailwindtraders,DC=internal"
```
```
4.New-ADOrganizationalUnit -Name "Brisbane" -Path "DC=tailwindtraders,DC=internal"
```
#### 2. Crear Usuarios y configurar sus fechas de vencimiento

Primero creamos al contratista de Sydney en su OU, le ponemos la fecha de vencimiento (1 de

enero de 2030) y luego creamos copias de esa plantilla hacia Melbourne y Brisbane.

PowerShell

# Definir la contraseña común
$Pass = ConvertTo-SecureString "Pa55w.rdPa55w.rd" -AsPlainText -Force

# Crear SydneyContractor con vencimiento el 01/01/


New-ADUser -Name "SydneyContractor" -SamAccountName "SydneyContractor"
-UserPrincipalName "SydneyContractor@tailwindtraders.internal" -Path
"OU=Sydney,DC=tailwindtraders,DC=internal" -AccountPassword $Pass
-AccountExpirationDate "01/01/2030" -Enabled $true

# Crear MelbourneContractor en su respectiva OU (emulando la copia)
New-ADUser -Name "MelbourneContractor" -SamAccountName "MelbourneContractor"
-UserPrincipalName "MelbourneContractor@tailwindtraders.internal" -Path
"OU=Melbourne,DC=tailwindtraders,DC=internal" -AccountPassword $Pass
-AccountExpirationDate "01/01/2030" -Enabled $true

# Crear BrisbaneContractor en su respectiva OU
New-ADUser -Name "BrisbaneContractor" -SamAccountName "BrisbaneContra

#### Crear el Grupo de Administradores de Sídney y añadir miembros

El manual pide que el grupo sea de ámbito **Universal** y que agreguemos a SydneyContractor.

PowerShell

# Crear el grupo con ámbito Universal en la OU Sydney
New-ADGroup -Name "Sydney Administrators" -SamAccountName "Sydney_Admins"
-GroupCategory Security -GroupScope Universal -Path
"OU=Sydney,DC=tailwindtraders,DC=internal"

# Añadir a SydneyContractor al grupo recién creado
Add-ADGroupMember -Identity "Sydney_Admins" -Members "SydneyContract”


#### Configurar un usuario como "Usuario Protegido"

Añadimos a SydneyContractor al grupo especial de seguridad nativo de Windows llamado

Protected Users.

PowerShell

Add-ADGroupMember -Identity "Protected Users" -Members "SydneyContractor"

#### 5. Delegar permisos de seguridad de una OU a un Grupo

En Server Core, la delegación de "Restablecer contraseñas" sobre una OU se realiza modificando

las Listas de Control de Acceso (ACL). Ejecuta estas líneas para otorgarle el permiso al grupo
Sydney Administrators:

PowerShell

$OUPath = "AD:\OU=Sydney,DC=tailwindtraders,DC=internal"
$TargetGroup = [System.Security.Principal.NTAccount]
("TAILWINDTRADERS\Sydney_Admins")
$Acl = Get-Acl -Path $OUPath

# GUIDs nativos de Active Directory para restablecer contraseñas
$ResetPasswordGUID = [GUID]"00299570-246d-11d0-a768-00aa006e57a3"
$UserClassGUID = [GUID]"bf967a86-0de6-11d0-a285-00aa003049e2"

$Ace = New-Object
System.DirectoryServices.ActiveDirectoryAccessRule($TargetGroup,
"ExtendedRight", "Allow", $ResetPasswordGUID, "Descendents", $UserClassGUID)
$Acl.AddAccessRule($Ace)
Set-Acl -Path $OUPath -AclObject $Acl


#### 6. Configurar atributo de Ciudad y realizar la Búsqueda

Le asignamos el valor "Sydney" en sus propiedades de dirección y simulamos el filtro de búsqueda
del manual.

PowerShell

# Configurar el atributo Ciudad (City)
Set-ADUser -Identity "SydneyContractor" -City "Sydney"

# Comando de búsqueda para verificar (Equivale al "Buscar ahora" del manual)
Get-ADUser -Filter {City -eq "Sydney"} | Select-Object Name, City


#### 7. Deshabilitar el usuario contratista de Melbourne

PowerShell

Disable-ADAccount -Identity "MelbourneContractor"

#### 8. Restablecer la contraseña del usuario contratista de Brisbane

El manual pide cambiarla a una contraseña diferente: Pa66w.rdPa66w.rd.

PowerShell

$NewPass = ConvertTo-SecureString "Pa66w.rdPa66w.rd" -AsPlainText -Force
Set-ADAccountPassword -Identity "BrisbaneContractor" -NewPassword $N

## 1. Configurar la política de contraseñas del dominio (GPO)

Ejecuta este bloque de comandos para exportar la directiva actual, cambiar el valor a 14 y aplicarla:


PowerShell

# 1. Exportar la directiva actual a un archivo temporal
secedit /export /cfg C:\gpo.cfg

# 2. Cambiar el valor de la longitud mínima a 14 caracteres
(Get-Content C:\gpo.cfg) -replace "MinimumPasswordLength = .*",
"MinimumPasswordLength = 14" | Set-Content C:\gpo.cfg

# 3. Importar y aplicar la nueva configuración en el sistema
secedit /configure /db $env:windir\security\local.sdb /cfg C:
\gpo.cfg /areas SECURITYPOLICY

# 4. Forzar la actualización inmediata de las directivas
gpupdate /force


## 2. Configurar una política de

## contraseñas detallada (Fine-Grained

## Password Policy)

Ejecuta este comando de PowerShell para generarla y enlazarla automáticamente:

PowerShell

# Crear la política detallada con los parámetros del manual
New-ADFineGrainedPasswordPolicy -Name "Domain Admin Password
Policy" `
-Precedence 1 `
-MinPasswordLength 16 `
-ComplexityEnabled $true `
-Description "Politica estricta para administradores" `
-LockoutThreshold 5 `
-LockoutDuration "00:30:00" `
-LockoutObservationWindow "00:30:00"

# Aplicar la política directamente al grupo Domain Admins
Add-ADFineGrainedPasswordPolicySubject -Identity "Domain Admin
Passwo


## 3. Habilitar la papelera de reciclaje de Active Directory

Ejecuta este comando para activarla de raíz:

Enable-ADOptionalFeature -Identity "CN=Recycle Bin Feature,CN=Optional
Features,CN=Directory Service,CN=Configuration,DC=tailwindtraders,DC=internal" -Scope
ForestOrConfigurationSet -Target "tailwindtraders.internal" -Confirm:$false


