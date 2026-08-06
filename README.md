# Binarios de ejecución (Box64 + Wine) para PCRuntimeParaAndroid

Binarios compilados para el kit de ejecución del launcher: **Box64** (emulador
x86_64 sobre ARM64) y **Wine** (compilado para x86_64, ejecutado bajo Box64
dentro de un rootfs glibc). Son los binarios que la app espera en el rootfs
para crear el prefijo Wine y lanzar juegos.

## Contenido

| Fichero            | Arquitectura    | Descripción |
|--------------------|-----------------|-------------|
| `box64`            | ARM64 (aarch64) | Emulador Box64 v0.4.4 (toolchain aarch64). |
| `wine`             | x86-64          | Lanzador de Wine 11.0 (ELF x86-64; lo ejecuta Box64). |
| `wineboot`         | symlink → wine  | Crea e inicializa el prefijo (`wineboot -i`). |
| `runtime.tar.gz`   | empaquetado     | Árbol `usr/` completo (bin + `lib/wine`) listo para volcar a un rootfs. |
| `runtime.sha256`   | texto           | SHA-256 de `runtime.tar.gz`. |

Además del ejecutable `box64`, el paquete incluye las librerías de Wine en
`usr/lib/wine/x86_64-unix` y `usr/lib/wine/x86_64-windows`.

## Compatibilidad

- **glibc del rootfs**: mínimo **2.39** (Ubuntu 24.04 o posterior).
- **Arquitectura**: el binario `box64` es **ARM64** y el `wine` es **x86-64**
  (x86-64 bajo Box64). No sirven en dispositivos x86_64 nativos.
- **PRoot**: el launcher sigue necesitando el binario **PRoot arm64** por
  separado (no se incluye aquí).

## Instalación en el rootfs

```bash
# Volcar el árbol usr/ dentro del rootfs
tar -xzf runtime.tar.gz -C /ruta/al/rootfs
chmod 755 /ruta/al/rootfs/usr/bin/box64 /ruta/al/rootfs/usr/bin/wine /ruta/al/rootfs/usr/bin/wineboot
```

O con el script del kit:

```bash
./install-to-rootfs.sh /storage/emulated/0/pcrt/runtime/rootfs
```

Resultados esperados (la app los detecta automáticamente):

```
/…/rootfs/usr/bin/box64
/…/rootfs/usr/bin/wine
/…/rootfs/usr/bin/wineboot -> wine
/…/rootfs/usr/lib/wine/…
```

## Sumas de verificación (SHA-256)

| Artefacto          | SHA-256 |
|--------------------|---------|
| `box64`            | `60c5364faa33a3c82b6cd9ecb305b918499b7ae8cd034e55ffb419624d0dbffb` |
| `wine`             | `32d79f607e0e9bddbb744516080c0691ce68ebe69f10b73f1e0b22a7fbb8bad0` |
| `runtime.tar.gz`   | `ffeded95e68d917418b8f2eb8b9034baf924c061274a0df475c123f9fd4905b7` |

> `runtime.sha256` del paquete es la fuente oficial de la suma del tar.gz.

## Cómo se compilaron (y cómo reconstruirlos)

- **Box64 v0.4.4**: compilación cruzada a ARM64 con `aarch64-linux-gnu-gcc`
  y CMake/Ninja sobre Ubuntu 24.04 (glibc 2.39).
- **Wine 11.0**: compilación **nativa x86_64** en Ubuntu 24.04
  (`./configure --prefix=/usr --enable-win64 && make && make install`).
- Para reconstruir desde cero: revisa el kit de compilación en
  `Dockerfile` + `build.sh` de esta carpeta (requiere Linux/Docker).

## Repos

- Box64: https://github.com/ptitSeb/box64 (tag v0.4.4)
- Wine: fuente desde el espejo usado por la app:
  https://github.com/wine-mirror/wine (tag wine-11.0)

## Licencia

- Box64: MIT.
- Wine: LGPL-2.1-or-later (Wine Public License / LGPL para la mayoría de partes).
