---
layout: post
title:  "Creacion Automatizada de Enlaces Simbolicos de Roms, desde un Disco externo a carpeta de roms en SteamDeck"
summary: "Este script en Bash crea enlaces simbólicos para juegos en una carpeta específica de EmulationStation en Steam Deck."
author: mayc1030
date: '2024-12-24 0:00:00'
category: ['steamdeck','emulacion']
thumbnail: /assets/img/posts/poststeamdeckp1.png
keywords:  steamdeck, linux, valve
usemathjax: true
permalink: /blog/steamdeck/p2
---

**Pasos del script:**
1. Solicitar el nombre de la carpeta:
El script pide al usuario que ingrese el nombre de la carpeta donde están almacenados los juegos.

2. Verificar que el nombre no esté vacío:
Si el usuario no ingresa un nombre, el script muestra un mensaje de error y se detiene.

3. Definir las rutas de origen y destino:
 - La carpeta de origen corresponde a la ubicación de las ROMs en la microSD.
 - La carpeta de destino es donde EmulationStation busca los juegos.

4. Crear un archivo .sh con los enlaces simbólicos:
Se genera un archivo de script que contiene comandos para crear enlaces simbólicos de los juegos desde la microSD hacia la carpeta de EmulationStation.

5. Ejecutar el archivo generado:
El script ejecuta automáticamente el archivo .sh, creando los enlaces simbólicos sin necesidad de mover archivos.

#### Contenido en archivo .sh:

```sh
#!/bin/bash

# Solicitar al usuario el texto a reemplazar
read -p "Ingrese el nombre de la carpeta a utilizar: -> " texto

# Verificar si la variable texto está vacía
if [ -z "$texto" ]; then
  echo "El nombre de la carpeta no puede estar vacío. Saliendo..."
  exit 1
fi

# Ruta a la carpeta de origen
origen="/run/media/deck/gamesd/steamapps/games/roms/$texto"
# Ruta a la carpeta de destino
destino="/home/deck/Emulation/roms/$texto"
# Nombre del archivo de salida
archivo_salida="enlaces_juegos_$texto.sh"

# Crear el archivo de salida y darle permisos de ejecución
> "$archivo_salida"
chmod +x "$archivo_salida"

# Agregar el shebang al archivo de salida
echo "#!/bin/bash" >> "$archivo_salida"

# Recorrer todos los archivos en la carpeta de origen
for archivo in "$origen"/*; do
  # Obtener el nombre del archivo sin la ruta
  nombre=$(basename "$archivo")

  # Crear la línea de enlace simbólico
  linea="ln -s \"$origen/$nombre\" \"$destino/$nombre\""

  # Agregar la línea al archivo de salida
  echo "$linea" >> "$archivo_salida"
done

echo "Archivo $archivo_salida creado con éxito."

# Ejecutar el archivo de salida
./"$archivo_salida"

echo "YA PUEDE CERRAR LA CONSOLA MANUALMENTE"
```





