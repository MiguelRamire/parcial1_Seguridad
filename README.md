# Informe de Laboratorio: Primer Parcial de Seguridad
### Extracción y Documentación de Archivos Ocultos Mediante Metadatos y Esteganografía

**Autor:** Miguel Ángel Ramírez Velásquez
**Institución:** Universidad Pontificia Bolivariana
**Asignatura:** Seguridad en Tecnologías de la Información

---

## Tabla de Contenidos

- [Resumen Ejecutivo](#-resumen-ejecutivo)
- [Entorno y Herramientas](#️-entorno-y-herramientas-utilizadas)
- [Bitácora de Procedimiento](#-bitácora-de-procedimiento-y-comandos)
- [Tabla Consolidada de Resultados](#-tabla-consolidada-de-resultados)
- [Conclusiones](#-conclusiones)

---

## Resumen Ejecutivo

Este documento recoge la bitácora detallada y secuencial de los comandos ejecutados en la terminal para resolver la primera etapa del parcial. El proceso consistió en clonar el repositorio de trabajo, inspeccionar de forma masiva los metadatos de un conjunto de 40 imágenes para localizar campos manipulados, correlacionar esas pistas mediante huellas criptográficas SHA-1 y, finalmente, extraer los cuatro archivos ocultos protegidos mediante criptografía asimétrica (`.gpg`) utilizando Steghide.

---

## Entorno y Herramientas Utilizadas

| Herramienta | Función | Versión |
|---|---|---|
| Ubuntu 22.04.5 LTS (WSL2) | Sistema operativo base | GNU/Linux x86_64 |
| Git | Control de versiones / descarga | — |
| ExifTool | Análisis de metadatos | v12.40 |
| sha1sum | Verificación e integridad | — |
| Steghide | Herramienta esteganográfica | v0.5.1 |

---

## Bitácora de Procedimiento y Comandos

### Paso 1: Obtención y ubicación del repositorio de práctica

Se inició clonando el repositorio para la práctica y accediendo al directorio de trabajo donde se encuentran los recursos multimedia.

```bash
git clone https://github.com/MiguelRamire/parcial1_Seguridad.git
cd parcial1_Seguridad/
ls
cd Embeb/
ls
```

### Paso 2: Instalación de ExifTool

Dado que el sistema no contaba de forma nativa con la herramienta de lectura de metadatos, se instaló el paquete correspondiente mediante el gestor de paquetes de Ubuntu.

```bash
sudo apt install libimage-exiftool-perl
```

### Paso 3: Inspección masiva de las imágenes

Se ejecutó una lectura general sobre todas las imágenes del directorio actual con `exiftool`. Esta revisión permitió examinar la estructura individual de cada archivo y constatar la presencia de campos con alteraciones o datos inusuales en los metadatos.

```bash
exiftool *
```

### Paso 4: Filtrado selectivo de campos y hallazgo de pistas

Para aislar la información relevante, se combinó `exiftool` con `grep`, enfocándose en los términos clave `indice`, `clue` y `traccia`.

```bash
exiftool -filename -Artist -LensModel -Make -Model *.jpg | grep -E "(indice|clue|traccia|===)"
```

**Resultados y pistas obtenidas:**

| N° | Imagen Origen | Atributo EXIF | Pista → Ciudad → Hash SHA-1 |
|---|---|---|---|
| 1 | `stock-photo-a-breathtaking-...-2402643087.jpg` | Artist | `indice` → paris → `caa42f77ee964644b75c9a1fd04b989a10e3e034` |
| 2 | `stock-photo-palacio-de-bellas-...-1630774060.jpg` | Lens Model | `clue` → medellin → `51f1a248718623fc231e2f3302312d0433fe31dd` |
| 3 | `stock-photo-rainbow-bridge-...-2588345801.jpg` | Make | `clue` → canberra → `a74ce8f61b0650cbb026fe441ed34f0f52800af4` |
| 4 | `stock-photo-seoul-south-korea-...-2595923107.jpg` | Camera Model Name | `traccia` → roma → `c39634afb6925ebbf1d2cd38495d108030b31c1b` |

### Paso 5: Correlación de hashes SHA-1 con los archivos objetivo

Para identificar inequívocamente qué archivos del directorio correspondían a cada hash, se calculó el SHA-1 de todas las imágenes y se filtró el resultado:

```bash
sha1sum *.jpg | grep -E "(caa42f77ee964644b75c9a1fd04b989a10e3e034|51f1a248718623fc231e2f3302312d0433fe31dd|a74ce8f61b0650cbb026fe441ed34f0f52800af4|c39634afb6925ebbf1d2cd38495d108030b31c1b)"
```

**Correspondencias identificadas:**

- **Canberra:** `1557659-2593x1725-desktop-hd-canberra-wallpaper-image.jpg`
- **Paris:** `wallpapersden.com_dubai-united-arab-emirates-city_1920x11752.jpg`
- **Medellin:** `wallpapersden.com_toronto-hd-canada_3000x2250.jpg`
- **Roma:** `wp5018679-london-4k-wallpapers.jpg`

### Paso 5.1: Análisis crítico de correspondencia entre pistas e imágenes portadoras

Un aspecto fundamental identificado durante el análisis forense fue la desconexión intencional entre los nombres de las imágenes y las contraseñas asociadas, lo que evidencia la sofisticación del reto planteado:

- La pista **"paris"** (Artist) no provenía de una imagen sobre París, sino de una foto de **Dubai**.
- La pista **"medellin"** (Lens Model) no provenía de Medellín, sino de **Toronto**.
- La pista **"roma"** (Camera Model Name) no provenía de Roma, sino de **Londres**.
- La pista **"canberra"** (Make) sí provino de la imagen de Canberra, siendo la única excepción.

La estructura del reto reveló una estrategia de defensa en profundidad donde la metainformación descriptiva difería de la identidad visual real de cada imagen. Solo mediante el análisis estricto de los hashes criptográficos SHA-1 se pudo establecer la correspondencia correcta entre pistas y archivos.

### Paso 6: Instalación de Steghide

Dado que las imágenes portadoras ocultaban información mediante esteganografía, se procedió a instalar la herramienta `steghide` en el sistema.

```bash
sudo apt install steghide
```

### Paso 7: Extracción esteganográfica de los 4 archivos ocultos

Utilizando las contraseñas obtenidas en las pistas de los metadatos, se ejecutó la extracción sobre cada una de las imágenes portadoras:

```bash
# Extracción archivo D (Canberra)
steghide extract -sf 1557659-2593x1725-desktop-hd-canberra-wallpaper-image.jpg -p "canberra"

# Extracción archivo A (Paris)
steghide extract -sf wallpapersden.com_dubai-united-arab-emirates-city_1920x11752.jpg -p "paris"

# Extracción archivo C (Medellín)
steghide extract -sf wallpapersden.com_toronto-hd-canada_3000x2250.jpg -p "medellin"

# Extracción archivo B (Roma)
steghide extract -sf wp5018679-london-4k-wallpapers.jpg -p "roma"
```

### Paso 8: Organización y control de versiones de los archivos extraídos

Una vez obtenidos los cuatro archivos protegidos (`A.pdf.gpg`, `B.pdf.gpg`, `C.pdf.gpg` y `D.pdf.gpg`), se estructuraron en una carpeta independiente para su control de versiones:

```bash
mkdir ArchivosOcultos
cd Embeb/
mv *.gpg ../ArchivosOcultos/
cd ..
cd ArchivosOcultos/
ls
```

### Paso 9: Configuración de identidad y registro local de cambios (commit)

Se configuró localmente el nombre de usuario y correo institucional, registrando los archivos en el control de versiones local de Git:

```bash
git add .
git config user.name "Miguel Ramirez"
git config user.email "miguel.ramirezv@upb.edu.co"
git commit -m "Extracción y adición de los 4 archivos ocultos del parcial"
```

### Paso 10: Autenticación por token (PAT) y sincronización remota (push)

Para subir los cambios al repositorio remoto en GitHub, se actualizó el usuario y se autenticó la sesión utilizando un Personal Access Token (PAT) clásico con permisos sobre el repositorio (`repo`):

```bash
git config user.name "MiguelRamire"
git push origin main
```

---

## Tabla Consolidada de Resultados

| Archivo Extraído | Imagen Portadora de Origen | Passphrase (Pista) |
|---|---|---|
| **A.pdf.gpg** | `wallpapersden.com_dubai-united-arab-emirates-city_1920x11752.jpg` | `paris` |
| **B.pdf.gpg** | `wp5018679-london-4k-wallpapers.jpg` | `roma` |
| **C.pdf.gpg** | `wallpapersden.com_toronto-hd-canada_3000x2250.jpg` | `medellin` |
| **D.pdf.gpg** | `1557659-2593x1725-desktop-hd-canberra-wallpaper-image.jpg` | `canberra` |

---

## Conclusiones

- La revisión masiva con `exiftool *` permitió identificar comportamientos anómalos incrustados directamente en los atributos de las imágenes que simulaban ser colecciones de stock legítimas.
- El filtrado mediante expresiones regulares estructuró de manera limpia la ruta de resolución del reto, permitiendo aislar únicamente la información relevante sin ruido de metadatos estándar.
- El análisis crítico de correspondencia reveló que la ofuscación intencional entre nombres de archivo y contraseñas fue un mecanismo de seguridad deliberado, demostrando que la confianza ciega en metainformación descriptiva puede constituir una vulnerabilidad analítica.
- La correlación entre los hashes SHA-1 extraídos y calculados permitió establecer una correspondencia precisa e inequívoca entre las pistas y los archivos portadores, validando la supremacía de las técnicas criptográficas sobre los indicadores nominales.
- Mediante el uso de `steghide` y las claves derivadas del análisis forense previo, se cumplió al 100% con el objetivo del parcial: extraer los cuatro archivos ocultos protegidos por criptografía asimétrica, documentando rigurosamente cada comando ejecutado y los hallazgos metodológicos asociados.
