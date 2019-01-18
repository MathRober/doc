# PowerShell SSH

Este documento describe cómo configurar los sistemas operativos Linux y Windows para poder hacer uso de SSH en las conexiones remotas de PowerShell. Para la realización de este documento he usado la distribución Ubuntu 18.04 en el caso de Linux y Windows 10 Pro en el caso de Windows.

***

## Configuración de Linux

Instalamos la distribución de Linux Ubuntu 18.04 y actualizamos a la última versión de los paquete disponibles.

```bash

sudo apt update
sudo apt upgrade

```

A continuación instalamos PowerShell:

```bash

# Descargamos las claves GPG del repositorio de Microsoft
sudo wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb

# Las registramos
sudo dpkg -i packages-microsoft-prod.deb

# Actualizamos la lista de productos
sudo apt-get update

# Instalamos PowerShell
sudo apt-get install -y powershell

```

En caso de que haya una nueva versión de PowerShell disponible, podemos actualizarla con la siguiente instrucción:

 ```bash

sudo apt upgrade powershell

```

***

## Configuración de Windows

En esta sección vamos a ver cómo instalar OpenSSH sobre Windows 10. Se puede instalar de manera separada la parte cliente y la parte servidor, pero en esta caso voy a instalar las dos cosas a la vez.

//TO DO  
INSTALACION CLIENTE
INSTALACIONSERVIDOR
RUTA DEL ARCHIVO DE CONFIGURACION
SERVICIOS
CONFIGURACION DEL SERVIDOR
FIREWALL

 Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

 Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0   

choco install openssh -params '"/SSHServerFeature /KeyBasedAuthenticationFeature"' -y  # Server ssh en windows 10

Install-Module -Force OpenSSHUtilsnetsh advfirewall firewall add rule name="SSHD Port" dir=in action=allow protocol=TCP localport=22
***

## SSH en Linux

Linux ya viene con OpenSSH instalado y configurado. OpenSSH tiene dos partes:

- Parte cliente. Programa ssh
- Parte servidor. Programa sshd

Podemos ver que OpenSSH está instalado consultando a apt:

```bash

~$ dpkg --list openssh\*

Name                   Version
+++-======================-================
ii  openssh-client         1:7.6p1-4ubuntu0
ii  openssh-server         1:7.6p1-4ubuntu0
ii  openssh-sftp-server    1:7.6p1-4ubuntu0

```

La parte cliente de ls -la OpenSSH es el comando ssh ubicado en /usr/bin:

```bash

~$ which sshd
/usr/bin/ssh

```

**Nota.** En ese mismo directorio hay varias herramientas de OpenSSH.

La parte servidor está ubicada en /usr/sbin:

```bash

~$ which sshd
/usr/sbin/sshd

```

y los archivos de configuración están ubicados en /etc/ssh. Como el archivo de configuración de la parte servidor lo tendremos que modifcar, previamente haremos una copia de seguridad del archivo original.

```bash

# Hacemos una copia del archivo de configuración inicial
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.original
# Evitamos que s pueda editar
sudo chmod a-w /etc/ssh/sshd_config.original
# editamos el archivo sshd_config para personalizarlo
vi /etc/ssh/sshd_config

```

La parte servidor está configurada como un servicio y lo podemos gestionar con el comando systemctl. Por ejemplo, para reiniciar el servicio haríamos lo siguiente:

```bash

sudo systemctl restart sshd.service

```

***

### Prueba de conexión desde otra máquina

Para verificar que la parte servidor está funcionando correctamente, realizamos una conexión ssh desde otra máquina:

```bash

ssh unix@ubuntu1804b.mshome.net  #usuario@servidor

```

**Nota.** Estoy usando Hyper-V para las máquinas virtuales con Linux y usando el sitch por defecto, por lo que está habilitado DHCP y las máquinas registran el nombre en una zona especial llamada mshome.net.

Una vez que tenemos correctamente configurado SSH en Linux, vamos a ver cómo usarlo desde dentro del Shell de PowerShell.

***

## SSH en Linux desde PowerShell

Vamos a ver cómo hacer uso de SSH desde el shell de PowerShell. El sistema operativo ya está preparado para usar SSH por lo que PowerShell podrá hacer uso de él. Para abrir PowerShell, ejecutamos el comando pwsh:

```bash

~$ pwsh
PowerShell 6.1.2
Copyright (c) Microsoft Corporation. All rights reserved.

https://aka.ms/pscore6-docs
Type 'help' to get help.

PS /home/unix>

```

y lanzamos ssh exactamente igual que desde bash. Por ejemplo, para conectarnos a otro equipo Linux:

```powershell

ssh unix@ubuntu1804c.mshome.net

```

Por defecto la conexión se hace al shell bash de la máquina remota. En este shell podemos ejecutar pwsh y ya tendremos acceso a PowerShell. Sin embargo, ¿podemos modificar el archivo de configuración del servidor ssh para que por defecto nos conectemos al shell PowerShell????.

Para permitir que los cmdlet de PowerShell se conecten por ssh, deberemos modificar el archivo sshd_config para agregar un nuevo Subsistema.

```bash

# editamos /etc/ssh/sshd_config agregando esta línea
Subsystem    powershell /usr/bin/pwsh -sshs -NoLogo -NoProfile
#Reiniciamos el servicio sshd
sudo systemctl restart sshd.service

```

Ahora desde PowerShell, podemos hacer cosas como esta:

```powershell

Invoke-Command -HostName ubuntu1804c.mshome.net -ScriptBlock { Get-ChildItem}

```

Podemos crear sesiones:

```powershell

PS /home/unix> $s = New-PSSession -HostName ubuntu1804c.mshome.net -UserName unix
unix@ubuntu1804c.mshome.net s password:
PS /home/unix> $s

 Id Name            Transport ComputerName    ComputerType    State
 -- ----            --------- ------------    ------------    ----
  2 Runspace1       SSH       ubuntu1804c.... RemoteMachine   Opened

PS /home/unix> Invoke-Command -Session $s -ScriptBlock {Get-Process | Select-Object -First 1}

 NPM(K)    PM(M)      WS(M)     CPU(s)      Id  SI ProcessName                    PSComputerName
 ------    -----      -----     ------      --  -- -----------                    --------------
      0     0.00       2.24       0.00    1663 655 (sd-pam)                       ubuntu1804c.mshome.net

```

Y también sesiones interactivas:

```powershell

PS /home/unix> Enter-PSSession -Session $s
[ubuntu1804c.mshome.net]: PS /home/unix>

```

***

## SSH en Windows desde Powershell Core

Desde una máquina con Windows 10 o Windows Server que tenga instalado un cliente de OpenSSH, podremos acceder a las máquinas Linux:

```cmd
#Desde una consola de línea de comandos cmd:

ssh unix@ubuntu1804c.mshome.net

```

Y desde powerShell (versión core), exactamente igual que hemos hecho antes:

```powershell

Invoke-Command -HostName ubuntu1804c.mshome.net -ScriptBlock { Get-ChildItem} -Username unix

```

```powershell

PS C:\temporal> $PSVersionTable | Select-Object PSEdition, OS

PSEdition OS
--------- --
Core      Microsoft Windows 10.0.17763


PS C:\temporal> Invoke-Command -HostName ubuntu1804c.mshome.net -ScriptBlock { $PSVersionTable | Select-Object PSEdition, OS } -Username unix
unix@ubuntu1804c.mshome.net s password:

PSEdition      : Core
OS             : Linux 4.15.0-43-generic #46-Ubuntu SMP Thu Dec 6 14:45:28 UTC 2018
PSComputerName : ubuntu1804c.mshome.net

```

***

## Acceso a SSH mediante claves asimétricas

ssh-keygen
Your identification has been saved in C:\Users\Roberto_Seoane/.ssh/id_rsa.
Your public key has been saved in C:\Users\Roberto_Seoane/.ssh/id_rsa.pub.

cat ~/.ssh/id_rsa.pub | ssh kubeadm@172.17.214.225 "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"




$s1 = New-Pssession -Hostname  ubuntu1804c.mshome.net -Username unix
$s2 = New-Pssession -ComputerName  windows10.mshome.net -Username user1

$sesiones = Get-PSsession

Invoke-Command -Session $sesiones -ScriptBlock {Get-Process pw*}



<#
https://github.com/PowerShell/Win32-OpenSSH/releases
Descomprimir en c:\windows\system32\OpenSSH
Agregar la ruta anteriro al path del sistema

.\install-sshd.ps1
.\ssh-keygen.exe -A
.\FixUserFilePermissions.ps1
.\FixHostFilePermissions.ps1
mklink /D c:\pwsh "C:\Program Files\PowerShell\6"

modificar el ssh_config a

PasswordAuthentication yes
Subsystem    powershell c:/pwsh/pwsh.exe -sshs -NoLogo -NoProfile


ssh administrator@DominioE@192.168.11.111

#>

##https://hostadvice.com/how-to/how-to-install-an-openssh-server-client-on-a-windows-2016-server/
##Get-WindowsCapability -Online | ? Name -like 'OpenSSH*'
#Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
#Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0



 are other SSH commands besides the client ssh. Each has its own page.
ssh-keygen - creates a key pair for public key authentication
ssh-copy-id - configures a public key as authorized on a server
ssh-agent - agent to hold private key for single sign-on
ssh-add - tool to add a key to the agent
scp - file transfer client with RCP-like command interface
sftp - file transfer client with FTP-like command interface
sshd - OpenSSH serv



