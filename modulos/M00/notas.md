# M00 · Notas

Onboarding: laboratorio, Git y método. Noviembre de 2026.

## Qué hice

- Hice un **backup completo** de mi notebook con Windows 10, que ya no recibía soporte normal: imagen del sistema, archivos personales, exportación de las distribuciones de WSL, lista de programas (`winget export`) y drivers (`Export-WindowsDriver`).
- Verifiqué la compatibilidad con Windows 11 (procesador, UEFI, Secure Boot y TPM 2.0) e hice una **instalación limpia de Windows 11 Pro**.
- Reinstalé solo los programas necesarios a partir de una lista depurada (de 87 a 39 paquetes), y los drivers del chipset desde mi backup.
- Restauré **Ubuntu 24.04 en WSL2** desde el backup, limité su memoria con `.wslconfig` y conecté Docker Desktop.
- Generé un **par de claves SSH ed25519** protegido con passphrase, lo registré en GitHub para autenticación y para firma, y configuré Git para firmar todos los commits.
- Publiqué este repositorio, con su primer commit firmado.
- Descargué dos imágenes (`hello-world` y `alpine`) y ejecuté contenedores a partir de ellas. Uno lo ejecuté con `--user 1000:1000`, para que el proceso no corriera como root.

## Qué aprendí

- **Antes de cualquier cambio, un backup verificado.** Un backup que nunca se probó no es un backup.
- **Imagen vs contenedor**: la imagen es la clase y el contenedor es el objeto. De una misma imagen se crean muchos contenedores.
- **Identificar recursos por atributos estables**: los números de disco y las letras de unidad cambian; la etiqueta de un volumen, no. Con las claves SSH pasa lo mismo: se identifican por su huella.
- **Leer los errores con calma**: el código de error de WSL decía exactamente qué faltaba. Y en una cadena de errores, el que hay que resolver es el primero.
- **Ciclo de vida del software**: Windows 10, .NET 8 y 9, Java 6 y Flash ya no tienen soporte. Con `ubuntu-distro-info` puedo consultar cuánto soporte le queda a mi Ubuntu (hasta mayo de 2029).
- Encontré un disco interno de 1 TB **sin ninguna partición**: lo particioné, le di formato NTFS y lo dejé como `L:` LABS para las máquinas virtuales.

## Qué falló y cómo lo resolví

| Problema | Causa | Solución |
| --- | --- | --- |
| Olvidé la passphrase de mi clave SSH | Una passphrase no se puede recuperar | **Rotación**: generé una clave nueva, la registré en GitHub y borré la vieja |
| `winget export` no encontraba `D:\backups` | El disco externo estaba desconectado | Lo conecté y repetí el comando |
| Después de reinstalar, el disco de backups apareció como `E:` | Windows reasignó las letras de unidad | Reasigné las letras identificando cada disco por su etiqueta (`Set-Partition`) |
| `wsl --import` falló con `HCS_E_SERVICE_NOT_AVAILABLE` | Faltaba activar la Plataforma de máquina virtual | La activé con `dism` y reinicié |
| Configuración duplicada en `/etc/wsl.conf` | Agregué una sección que ya existía | Respaldé el archivo y borré las líneas duplicadas con `sed` |
| `git remote add` devolvió `not a git repository` | No había ejecutado `git init` | Primero `git init`, después `git remote add` |
| Ejecuté comandos de Linux en PowerShell | Confundí las terminales | Ahora miro el prompt: `PS` adelante es PowerShell; `$` al final es Linux |

## Hallazgos de seguridad que encontré

- **Sistema operativo sin soporte**: Windows 10 ya no recibía actualizaciones normales. Lo resolví con la instalación limpia de Windows 11.
- **Activador no oficial**: mi Windows Home figuraba activado por el canal KMS. KMS es un mecanismo legítimo, pero para empresas; en un equipo hogareño indica un activador de origen desconocido, que corre como administrador y puede dejar persistencia. La instalación limpia lo eliminó.
- **Software sin soporte**: en mi lista de programas había Java 6, Flash, versiones de .NET vencidas y "actualizadores de drivers" de terceros. No los reinstalé.
- **Contenedores como root por defecto**: `docker run --rm alpine id` devolvió `uid=0(root)`. Con `--user 1000:1000`, el mismo contenedor corre sin privilegios.
- **Control de integridad funcionando**: winget rechazó el instalador de Chrome porque su hash no coincidía con el del catálogo. No salteé la verificación.
- **Un gestor de paquetes no garantiza la última versión**: winget me instaló Nmap 7.80, de 2019.
- **Pendiente**: mi disco de backups no está cifrado, y contiene exportaciones de WSL con claves SSH.
- **El principio que aplica a todo**: mínimo privilegio. Ningún usuario, proceso o contenedor debería tener más permisos de los que necesita.

## Pendientes

- [ ] Cifrar el disco de backups con BitLocker To Go
- [ ] Reemplazar Nmap 7.80 por la última versión de nmap.org
- [ ] Rotar la clave SSH de la PC de escritorio y revisar la del servidor Traccar
