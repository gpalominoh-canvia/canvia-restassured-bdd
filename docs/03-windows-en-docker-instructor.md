# Windows en Docker (demo del instructor)

> Versión para imprimir: [PDF con formato Canvia](pdf/manual-entorno-docker-restassured.pdf).

Levanta un **Windows 11 recién instalado** dentro de Docker para mostrar en clase la instalación completa,
paso a paso, sobre una máquina limpia. Usa la imagen [`dockurr/windows`](https://github.com/dockur/windows),
que descarga Windows desde Microsoft, lo instala solo y lo muestra en el navegador.

Archivos en [`docker/windows/`](../docker/windows/).

| Perfil | Para qué | Qué trae |
|---|---|---|
| `limpio` | Demo en vivo: "todos síganme" | Windows sin nada instalado |
| `listo` | Tener una máquina de respaldo ya configurada | JDK 17, Maven 3.9, Git, VS Code + extensiones y el repo en el escritorio |

## Alternativa rápida: Windows Sandbox

Si tu equipo no expone `/dev/kvm` (pasa en PCs **AMD con la seguridad VBS / integridad de memoria** de Windows
activada), usa **Windows Sandbox**, incluido en Windows 10/11 Pro: un Windows limpio que abre en segundos
y se borra al cerrarlo.

1. Actívalo una vez (PowerShell **como administrador**) y reinicia la PC:

   ```powershell
   Enable-WindowsOptionalFeature -Online -FeatureName Containers-DisposableClientVM -All
   ```

2. Crea un archivo `demo-windows-limpio.wsb` que comparta una carpeta con los instaladores:

   ```xml
   <Configuration>
     <MemoryInMB>8192</MemoryInMB>
     <MappedFolders>
       <MappedFolder>
         <HostFolder>C:\cursos\instaladores</HostFolder>
         <SandboxFolder>C:\Users\WDAGUtilityAccount\Desktop\instaladores</SandboxFolder>
         <ReadOnly>true</ReadOnly>
       </MappedFolder>
     </MappedFolders>
   </Configuration>
   ```

3. Doble clic en el `.wsb` y sigue la [instalación paso a paso](01-instalacion-windows-paso-a-paso.md).
   Sandbox **no trae `winget`**: usa los instaladores `.msi` / `.exe` de la carpeta compartida
   (los mismos enlaces de descarga del paso a paso).

## Requisitos del equipo anfitrión

- Windows 11 (o Linux) con **Docker Desktop** y backend **WSL 2**.
- Virtualización activada en BIOS. Docker Desktop en Windows 11 expone `/dev/kvm` a los contenedores;
  compruébalo con:

  ```powershell
  docker run --rm --device=/dev/kvm alpine ls -l /dev/kvm
  ```

  Si responde *no such file or directory*, activa la virtualización anidada de WSL. Crea (o edita)
  `%USERPROFILE%\.wslconfig` con:

  ```ini
  [wsl2]
  nestedVirtualization=true
  ```

  Cierra Docker Desktop, ejecuta `wsl --shutdown`, vuelve a abrir Docker Desktop y repite la prueba.
  Si sigue sin aparecer, revisa que la virtualización (Intel VT-x / AMD SVM) esté activada en la BIOS.
- 8 GB de RAM libres (la VM usa 6 GB) y **70 GB de disco** libres.
- Buena conexión: la primera vez descarga la ISO de Windows (~6 GB).

## Levantar el Windows limpio

```powershell
cd docker\windows
.\iniciar-windows.bat            # equivale a: docker compose --profile limpio up -d
```

Abre <http://localhost:8006>. Verás la instalación de Windows avanzando sola (20–40 min la primera vez).
Cuando aparezca el escritorio, entra con:

| Usuario | Clave |
|---|---|
| `alumno` | `canvia` |

**Prepáralo antes de la clase.** La instalación se hace una sola vez; el disco queda en el volumen
`disco-limpio`. El día de la clase, `iniciar-windows.bat` arranca en 1–2 minutos.

### Truco: congelar el Windows limpio

Para repetir la demo varias veces desde cero, guarda una copia del disco recién instalado y restáurala:

```powershell
docker compose --profile limpio stop
docker run --rm -v canvia-windows-restassured_disco-limpio:/d -v ${PWD}:/b alpine tar czf /b/windows-limpio.tgz -C /d .
# ... tras la demo, para volver al estado limpio:
docker run --rm -v canvia-windows-restassured_disco-limpio:/d -v ${PWD}:/b alpine sh -c "rm -rf /d/* && tar xzf /b/windows-limpio.tgz -C /d"
```

## Guion de la demo

Con el Windows limpio abierto en <http://localhost:8006>, sigue
[Instalación en Windows, paso a paso](01-instalacion-windows-paso-a-paso.md) desde el Paso 0.

- Comparte la pestaña del navegador en la videollamada; los alumnos repiten cada comando en su PC.
- Haz la verificación de cada paso (`java -version`, `mvn -v`...) **antes** de avanzar y pide que levanten la mano
  quienes no ven lo mismo.
- Para pegar comandos: el visor web permite pegar con el portapapeles del panel lateral. Para una experiencia
  más fluida usa Escritorio remoto: `mstsc` → `localhost:3389`, usuario `alumno`, clave `canvia`.

### Pasar archivos al Windows

La carpeta del repositorio se comparte con Windows: en el escritorio aparece **Shared**
(o abre `\\host.lan\Data` en el Explorador). Ahí están el PDF del manual, el proyecto y esta guía.

## Levantar el Windows listo (autoconfigurado)

```powershell
cd docker\windows
.\iniciar-windows.bat listo
```

Al terminar la instalación, Windows ejecuta [`oem/install.bat`](../docker/windows/oem/install.bat), que instala
JDK 17, Maven, Git y VS Code ([`oem/configurar.ps1`](../docker/windows/oem/configurar.ps1)). En el primer inicio
de sesión se instalan las extensiones y se copia el repo al escritorio
([`oem/primer-inicio.ps1`](../docker/windows/oem/primer-inicio.ps1)).

Registro de la instalación: `C:\OEM\configurar.log` dentro del Windows.

> El script solo corre durante la **primera instalación**. Si cambias los `.ps1`, borra el disco
> (`docker compose --profile listo down -v`) y vuelve a levantarlo.

## Apagar y limpiar

| Acción | Comando (en `docker\windows`) |
|---|---|
| Apagar (conserva el disco) | `.\detener-windows.bat` |
| Ver el avance o errores | `docker compose --profile limpio logs -f` |
| Borrar los discos de Windows | `docker compose --profile limpio --profile listo down -v` |

## Ajustes

Todo está en [`docker/windows/docker-compose.yml`](../docker/windows/docker-compose.yml):

| Variable | Valor | Nota |
|---|---|---|
| `VERSION` | `11` | `10` para Windows 10 |
| `LANGUAGE` | `Spanish` | idioma de Windows |
| `KEYBOARD` | `es-ES` | `es-MX` para teclado latinoamericano |
| `RAM_SIZE` / `CPU_CORES` | `6G` / `4` | bájalos si el equipo tiene poca memoria |
| `DISK_SIZE` | `64G` | tamaño máximo; crece según uso |

> Windows corre sin activar (modo evaluación), suficiente para una demo. No uses esta VM para trabajo real.

---

Anterior: [Entorno listo en Docker](02-entorno-docker-alumnos.md) · [Índice](README.md) · Siguiente: [Git y GitHub](04-git-y-github.md)
