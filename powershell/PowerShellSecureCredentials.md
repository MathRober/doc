
# Métodos para cifrar credenciales en PowerShell

En este documento se describen diferentes métodos para cifrar contraseñas que se usarán en la ejecución de scripts de powershell.

## Cifrado de credenciales con ConvertTo-SecureString

La forma más básica de cifrar credenciales para su uso en PowerShell es a través del cmdlet ConvertTo-SecureString. Veamos el siguiente ejemplo:

~~~powershell

$CredencialCifrada = 'C:\Store\CredencialCifrada.txt'
Read-Host "Introduce la contraseña a cifrar: " -AsSecureString  |
           ConvertFrom-SecureString |
           Out-File $CredencialCifrada

~~~

El contenido del archivo CredencialCifrada.txt contendrá algo similar a esto:

01000000d08c9ddf0115d1118c7a00c04fc297eb010000002ff2a97f44617e4e952acca6b78311470000000002000000000003660000c000000010000000f62b86fd6b3b15237ca47722bc0609030000000004800000a0000000100000008c04059c53a6dc71500c592a96e672b808000000b35fa7cfa4a1605c140000006db9ecd60d3ac37c14159e48600fc9705013f9c3

Esta cadena contiene la contraseña cifrada. Para poder descifrar la contraseña haremos lo siguiente:

~~~powershell

$Password = Get-Content $CredencialCifrada | ConvertTo-SecureString
$Usuario = 'Dominio\Usuario'
$CredencialObj = -[PSCredential]::New($Usuario,$Password)

~~~

ES decir, leemos el archivo con la contraseña cifrada, la convertimos a una cadena segura y a continuación, creamos un objeto PSCredential para poder usarlo en futuras llamadas.

**Ventajas de este método:** Muy facil de configurar

**Desventajas de este método:** Solo funciona en la máquina y en el perfil de usuario que se configure.

***

## Módulo Credential Manager

Existe un módulo en la Powershell Gallery llamado **Credential Manager** que nos permite almacenar credenciales en el Administrador de Credenciales de Windows.

~~~powershell

Install-Module CredentialManager
Import-Module CredentialManager
Get-Command -Module CredentialManager

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-StoredCredential                               2.0        CredentialManager
Cmdlet          Get-StrongPassword                                 2.0        CredentialManager
Cmdlet          New-StoredCredential                               2.0        CredentialManager
Cmdlet          Remove-StoredCredential                            2.0        CredentialManager

~~~

Para guardar una credencial en el almacen de credenciales de Windows, ejecutaremos los siguiente:

~~~powershell

$datosStoredCredential = @{
    Target    = 'Servidor'
    UserName  = "Taskscheduleruser"
    Password  = 'Palabradepaso'
    Persist   = 'LocalMachine'
    Type      = 'Generic'
}

New-StoredCredential @datosStoredCredential

~~~

Para acceder al almacen de credenciales de Windows para obtener una credencial guardada, haremos lo siguiente:

~~~powershell

$servidor = 'Servidor'
$cred = Get-StoredCredential -Target $servidor
$cred

UserName                              Password
--------                              --------
Taskscheduleruser System.Security.SecureString

~~~

**Notas:** Si se elige el parámetro Persist como Enterprise, se puede acceder a las credenciales desde otras máquinas a través de la red, pero siempre con el mismo usuario. Si se elige el parámetro Persist como LocalMachine solo funciona en la máquina donde se ha configurado.

**Ventajas de este método:** Muy facil de configurar.  

**Desventajas de este método:** Solo funciona en el perfil de usuario que se configure. Solo funciona en Windows.

***

## Certificado Autofirmado


















