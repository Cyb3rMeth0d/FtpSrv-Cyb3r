<div align="center">

<img src="ruta/a/tu/logo.png" alt="FtpSv-Cyb3r Logo" width="380">

# ⚡ FtpSv-Cyb3r `V.1a`

**Servidor FTP multihilo de alto rendimiento para payloads ELF en PS4, PS5 y GNU/Linux.**

![Plataforma](https://img.shields.io/badge/Plataforma-PS5%20%7C%20PS4%20%7C%20Linux-0070D1?style=for-the-badge&logo=playstation)
![Versión](https://img.shields.io/badge/Versión-V.1a-00E5FF?style=for-the-badge)
![Puerto](https://img.shields.io/badge/Puerto%20FTP-2121-2ea44f?style=for-the-badge)
![Licencia](https://img.shields.io/badge/Licencia-GPL--3.0-orange?style=for-the-badge)

</div>

---

## 📌 Características principales

* **Multiplataforma:** Compilable como payload ELF para **PlayStation 4** y **PlayStation 5**, además de binario nativo POSIX para **GNU/Linux**.
* **Arquitectura multihilo:** Crea un hilo independiente por cada conexión FTP simultánea.
* **Notificación instantánea:** Al arrancar muestra en pantalla `FTP INICIADO: IP:puerto` por cada interfaz de red activa.
* **Interfaz personalizada:** Incluye banner ASCII exclusivo al iniciar sesión y documentación íntegra en español.
* **Puerto predeterminado `2121`:** Configurable al ejecutar el binario *(nota: el puerto `9021` habitual en los SDK corresponde al envío del payload, no al servidor FTP)*.

> [!WARNING]
> **Aviso de seguridad:** Este servidor **no autentica usuarios ni contraseñas** (`USER` y `PASS` aceptan cualquier valor) y el tráfico FTP viaja sin cifrar. Dado que el payload opera con privilegios elevados sobre el sistema de archivos y procesos, **úsalo exclusivamente en redes locales de confianza** y nunca lo expongas a Internet.

---

## 🚀 Uso rápido

1. Envía el payload (`ftpsrv-ps5.elf` o `ftpsrv-ps4.elf`) a tu consola mediante tu cargador ELF habitual.
2. Comprueba la **IP** y el **puerto** en la notificación emergente (`FTP INICIADO: IP:puerto`).
3. Conecta desde tu cliente FTP favorito (**FileZilla**, **WinSCP**, **Cyberduck**, **lftp**...) dejando usuario y contraseña en blanco.

Si deseas cambiar el puerto por defecto al lanzar el ejecutable:

```bash
./ftpsrv-ps5.elf -p 2121
```
*(Usa `-h` para listar todas las opciones disponibles).*

---

## 🛠️ Comandos soportados

### Comandos FTP estándar
* **Navegación y listado:** `LIST`, `NLST`, `MLSD`, `MLST`, `CWD`, `CDUP`, `PWD`
* **Transferencias:** `RETR`, `STOR`, `APPE`, `REST`
* **Gestión de archivos:** `MKD`, `RMD`, `DELE`, `RNFR`, `RNTO`, `SIZE`, `MDTM`
* **Modos de red:** `PORT`, `PASV`, `EPRT`, `EPSV`
* **Sesión y negociación:** `USER`, `PASS`, `FEAT`, `OPTS`, `TYPE`, `MODE`, `STRU`, `SYST`, `STAT`, `HELP`, `NOOP`, `ABOR`, `ALLO`, `QUIT`
* **Alias heredados:** `XCUP`, `XMKD`, `XPWD`, `XRMD` *(los comandos `XRCP`, `XRSQ`, `XSEM` y `XSEN` se reconocen pero no están implementados)*.

### Comandos personalizados (`SITE`)
Se pueden ejecutar directamente o precedidos del prefijo `SITE`:

| Comando | Descripción | Plataforma |
| :--- | :--- | :---: |
| `MTRW` | Remonta `/preinst`, `/system` y `/system_ex` con permisos de escritura *(en PS4 incluye `/preinst2`)*. | PS4 / PS5 |
| `AUTHID <hex>` | Modifica el `AuthID` del proceso actual. | PS5 |
| `SELF` | Activa/desactiva por sesión el descifrado automático de archivos `SELF` a `ELF` al descargarlos. | PS4 / PS5 |
| `SCHK [0\|1]` | Activa o desactiva la verificación del digest `SELF` *(activada por defecto al conectar)*. | PS4 / PS5 |
| `CHMOD <modo> <ruta>` | Cambia los permisos de un archivo o directorio. | Todas |
| `UMASK [modo]` | Consulta o establece la máscara de permisos. | Todas |
| `SYMLINK <obj> <enlace>` | Crea un enlace simbólico. | Todas |
| `COPY <orig> <dest>` | Copia archivos o directorios de forma recursiva en el servidor. | Todas |
| `CPFR <orig>` / `CPTO <dest>` | Inicia y ejecuta una copia directa en el servidor. | Todas |
| `MOVE <orig> <dest>` | Mueve archivos o directorios en el servidor. | Todas |
| `RMDA <dir>` / `RMDIR` | Elimina un directorio y todo su contenido de forma recursiva. | Todas |
| `LOWER <ruta>` / `UPPER <ruta>` | Convierte recursivamente los nombres a minúsculas o mayúsculas *(verifica colisiones antes de renombrar)*. | Todas |
| `DSIZ <ruta>` | Calcula el tamaño total de un directorio *(devuelve error si se aplica a un archivo)*. | Todas |
| `AVBL <ruta>` | Muestra el espacio libre disponible en bytes. | Todas |
| `XQUOTA` | Muestra los límites y el uso actual de espacio y archivos. | Todas |
| `STOP` | Detiene la tarea en segundo plano que esté en curso. | Todas |
| `KILL` | Cierra y finaliza el proceso del servidor FTP. | Todas |

> [!NOTE]
> Las operaciones pesadas (`COPY`, `MOVE`, `RMDA`, `LOWER`, `UPPER`) se ejecutan en **segundo plano** de una en una. Mientras haya una activa no se podrá lanzar otra, y sus posibles errores posteriores no se reportan al cliente FTP.

### 📂 Sistema virtual `/proc` (Exclusivo PS5)
En PlayStation 5 se expone una vista en tiempo real de los procesos activos bajo `/proc`:
* `LIST /proc` — Lista todos los procesos en ejecución junto a la entrada `_all_`.
* `RETR /proc/_all_` — Descarga la tabla completa de procesos.
* `RETR /proc/<pid>` o `/proc/<pid>_<title-id>_<command>` — Obtiene información detallada de un proceso concreto.
* `DELE /proc/<pid>` — Envía una señal `SIGKILL` para cerrar forzosamente dicho proceso.

---

## ⚙️ Compilación desde el código fuente

Requiere un entorno POSIX (Linux, macOS o WSL) con las herramientas de compilación estándar y el SDK correspondiente.

<details>
<summary><b>🔵 Compilar para PlayStation 5</b> (SDK v0.50)</summary>

Instala el [PS5 Payload SDK](https://github.com/ps5-payload-dev/sdk), exporta su ruta y compila:
```sh
export PS5_PAYLOAD_SDK=/ruta/al/ps5-payload-sdk
make -f Makefile.ps5
```
*Genera el binario `ftpsrv-ps5.elf`.*
</details>

<details>
<summary><b>🔷 Compilar para PlayStation 4</b></summary>

Instala el [PS4 Payload SDK](https://github.com/ps4-payload-dev/sdk), exporta su ruta y compila:
```sh
export PS4_PAYLOAD_SDK=/ruta/al/ps4-payload-sdk
make -f Makefile.ps4
```
*Genera el binario `ftpsrv-ps4.elf`.*
</details>

<details>
<summary><b>🐧 Compilar para GNU/Linux</b></summary>

Con `gcc`/`clang` y soporte para `pthreads`:
```sh
make -f Makefile.linux
```
*Genera el binario `ftpsrv-posix.elf`.*
</details>

---

## 📁 Estructura del repositorio

* **Núcleo FTP:** `srv.c/h`, `cmd.c/h`, `io.c/h`, `self.c/h`, `sha256.c/h`, `notify.c/h`, `log.h`, `main-common.h`
* **Implementación PS5:** `main-prospero.c`, `cmd-prospero.c`, `self-prospero.c`, `cmd-proc.c/h`, `io-freebsd.c`
* **Implementación PS4:** `main-orbis.c`, `cmd-orbis.c`, `self-orbis.c`
* **Implementación Linux:** `main-linux.c`, `cmd-linux.c`, `self-linux.c`, `io-linux.c`
* **Personalización y build:** `logo.h` (banner ASCII), `Makefile.ps5`, `Makefile.ps4`, `Makefile.linux`
* **Tests:** `test/test-lftp.sh`, `test/gen-testdata.sh`

---

## 🤝 Créditos y Licencia

* **Modificación y distribución (`FtpSv-Cyb3r V.1a`):** Adaptado y mantenido por **Cyb3rMeth0d** (banner ASCII personalizado, notificaciones en español y ajustes de esta build).
* **Proyecto base:** Desarrollado originalmente por **John Törnblom** en [ps5-payload-dev/ftpsrv](https://github.com/ps5-payload-dev/ftpsrv).
* **SDKs y herramientas:** Agradecimientos a los equipos de **ps5-payload-dev** y **ps4-payload-dev**.

Distribuido bajo licencia **GNU GPL v3.0 o posterior**. Consulta el archivo [LICENSE](LICENSE) para más detalles.
