---
layout: post
title:  "Gestión Automática de ROMs en Steam Deck: Creación y Limpieza de Enlaces Simbólicos con Bash"
summary: "Estos dos scripts están diseñados para gestionar enlaces simbólicos de ROMs en la Steam Deck para EmulationStation. Permiten organizar juegos almacenados en discos externos y sincronizarlos con la carpeta de EmulationStation sin necesidad de mover archivos."
author: mayc1030
date: '2024-12-24 0:00:00'
category: ['steamdeck','emulacion']
thumbnail: /assets/img/posts/poststeamdeck.png
keywords:  steamdeck, linux, valve
usemathjax: true
permalink: /blog/steamdeck/p3
---

#### Resumen del funcionamiento
1. El script principal verifica qué discos están montados y elige el adecuado.
2. Llama al script auxiliar, que se encarga de crear los enlaces simbólicos.
3. Elimina enlaces simbólicos inválidos que apunten a discos no montados.
4. Sincroniza datos en la nube con EmuDeck.
5. Ejecuta EmulationStation para que reconozca los juegos correctamente.
Este sistema permite mantener organizadas las ROMs en discos externos y acceder a ellas en EmulationStation sin necesidad de copiarlas manualmente.


#### Primer script (crear_enlaces.sh - Script principal)
Este script automatiza la creación y limpieza de enlaces simbólicos para múltiples sistemas de juegos.

1. Selección de sistemas de juego
 - Define una lista de consolas compatibles `(Dreamcast, GameCube, GBA, PS2, etc.)`.
 - Si no se proporciona un argumento, el script procesa todas las consolas de la lista.
2. Verificación de parámetros
 - Si se proporciona una consola específica como argumento, verifica que sea válida.
 - Si el nombre de la carpeta no coincide con los sistemas disponibles, muestra un error y termina la ejecución.
3. Detección del disco de origen, define rutas de origen en dos posibles ubicaciones de almacenamiento externo:
 - `/run/media/deck/mayck/SteamLibrary/Emulation/roms`
 - `/run/media/deck/gamesd/steamapps/games/roms`
 - Verifica cuál de estos discos está montado y elige el que esté disponible.
 - Si un disco está montado, ejecuta un script secundario (crear_enlaces.sh) con la carpeta correspondiente.
4. Eliminación de enlaces simbólicos inválidos
 - Escanea la carpeta de enlaces de EmulationStation y elimina enlaces simbólicos que apunten a discos no montados.
 - Esto evita enlaces rotos cuando los discos externos no están conectados.
5. Sincronización en la nube y ejecución de EmulationStation-DE
 - Ejecuta funciones de EmuDeck para sincronizar configuraciones desde la nube.
 - Finalmente, ejecuta EmulationStation-DE `($ESDE_toolPath)` con los parámetros recibidos.


**Contenido en archivo .sh:**

```sh
#!/bin/bash

# Verificar si se recibieron los parámetros necesarios
[ -z "$1" ] || [ -z "$2" ] && echo "Error: Se requieren dos parámetros." && exit 1

# Definir variables
ORIGEN="$1/$2"
DESTINO="/home/deck/Emulation/roms/$2"

# Verificar si la carpeta de origen existe
[ ! -d "$ORIGEN" ] && echo "La carpeta de origen $ORIGEN no existe. Saliendo..." && exit 1

# Crear la carpeta de destino si no existe
mkdir -p "$DESTINO"

# Crear enlaces simbólicos solo si no existen
NUEVOS_ENLACES=0

for ARCHIVO in "$ORIGEN"/*; do
  [ -e "$ARCHIVO" ] || continue  # Saltar si no hay archivos
  ENLACE="$DESTINO/$(basename "$ARCHIVO")"
  [ -L "$ENLACE" ] || ln -s "$ARCHIVO" "$ENLACE" && NUEVOS_ENLACES=1
done

# Mensaje final
[ "$NUEVOS_ENLACES" -eq 1 ] && echo "Enlaces creados en $DESTINO." || echo "Todos los enlaces ya existen."
```

#### Segundo script (crear_enlaces.sh - Script auxiliar)
Este script se encarga de crear los enlaces simbólicos en la carpeta de EmulationStation para un sistema específico.

1. Verificación de parámetros, recibe dos argumentos: 
 - ***Ruta de origen*** (ubicación de las ROMs en el disco externo).
 - ***Sistema de juego*** (nombre de la carpeta de destino en EmulationStation).
 - Si falta alguno de estos valores, muestra un error y termina la ejecución.
2. Creación de la carpeta de destino
 - Si la carpeta de destino en EmulationStation no existe, la crea.
3. Creación de enlaces simbólicos
 - Recorre todos los archivos en la carpeta de origen y crea un enlace simbólico en la carpeta de destino.
 - Si un enlace ya existe, lo omite para evitar duplicados.
 - Cuenta cuántos enlaces nuevos fueron creados.
4. Mensaje final
 - Si se crearon enlaces nuevos, muestra un mensaje de confirmación.
 - Si todos los enlaces ya existían, indica que no fue necesario hacer cambios.


**Contenido en archivo .sh:**

```sh
#!/bin/bash

# Opciones disponibles para TEXTO
TEXTO_OPCIONES=("dreamcast" "gamecube" "gb" "gba" "gbc" "n64" "nds" "nes" "ps2" "ps3" "psp" "psvita" "psx" "switch" "xbox" "xbox360")

# Si no se pasa un argumento, se asignan todas las opciones
if [ -z "$1" ]; then
  TEXTO_OPCIONES_SELECCIONADAS=("${TEXTO_OPCIONES[@]}")
else
  TEXTO_OPCIONES_SELECCIONADAS=("$1")
fi

# Recorrer todas las opciones seleccionadas
for TEXTO in "${TEXTO_OPCIONES_SELECCIONADAS[@]}"; do
  # Verificar si el valor de TEXTO es válido
  if [[ ! " ${TEXTO_OPCIONES[@]} " =~ " ${TEXTO} " ]]; then
    echo "Error: TEXTO debe ser uno de los siguientes: ${TEXTO_OPCIONES[@]}"
    exit 1
  fi

  SYMLINK_FOLDER="/home/deck/Emulation/roms/$TEXTO"
  DISK_MOUNTS=("/run/media/deck/mayck" "/run/media/deck/gamesd")
  ORIGEN_ROMS=("/run/media/deck/mayck/SteamLibrary/Emulation/roms" "/run/media/deck/gamesd/steamapps/games/roms")

  # Verificar si la carpeta de enlaces simbólicos existe
  if [ ! -d "$SYMLINK_FOLDER" ]; then
    echo "Carpeta $SYMLINK_FOLDER no existe. Omitiendo..."
    continue
  fi

  DISK_FOUND=false

  # Recorrer las rutas de montaje y verificar si alguna está activa
  for i in "${!DISK_MOUNTS[@]}"; do
    if mount | grep -q "${DISK_MOUNTS[$i]}"; then
      DISK_FOUND=true
      echo "Usando origen: ${ORIGEN_ROMS[$i]}"
      bash /home/deck/Emulation/tools/launchers/es-de/crear_enlaces.sh "${ORIGEN_ROMS[$i]}" "$TEXTO"
    fi
  done

  # Verificar y eliminar enlaces simbólicos que no pertenezcan a discos montados
  echo "Verificando enlaces simbólicos en $SYMLINK_FOLDER..."

  # Usar `find` para obtener todos los enlaces simbólicos y verificar sus destinos
  find "$SYMLINK_FOLDER" -type l | while read -r symlink; do
    target=$(readlink -f "$symlink")

    # Verificar si el destino del enlace pertenece a un disco montado
    LINK_VALID=false
    for mount in "${DISK_MOUNTS[@]}"; do
      if [[ "$target" == "$mount"* ]]; then
        LINK_VALID=true
        break
      fi
    done

    # Si el enlace simbólico no es válido, eliminarlo
    if [ "$LINK_VALID" = false ]; then
      echo "Eliminando $symlink (apunta a $target, disco no montado)"
      rm "$symlink"
    fi
  done

  echo "Proceso completado para $TEXTO."
done



source $HOME/.config/EmuDeck/backend/functions/all.sh
cloud_sync_downloadEmuAll && cloud_sync_startService
"$ESDE_toolPath" "${@}"
rm -rf "$savesPath/.gaming"
```




