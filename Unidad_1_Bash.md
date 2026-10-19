# Unidad 1. Bash para el manejo de datos genómicos

**Curso:** Ciencias Genómicas, semestre 27-1, ENCB-IPN.  
**Profesor:** Dr. Juan Alfredo Hernández García.  
**Sesiones vinculadas:** 29 de octubre y 5 de noviembre de 2026.  
**Regresar a:** [página principal](README.md).

## Objetivos y productos

Al terminar esta práctica podrás navegar por directorios, inspeccionar archivos FASTA, buscar patrones, generar tablas y ejecutar un script sencillo.

Entregarás un script comentado, una tabla resumen y una bitácora con los comandos utilizados, los resultados y su interpretación. La práctica principal funciona **sin conexión a internet**; la descarga de NCBI es una actividad adicional.

## 1. Terminal, shell y Bash

La **terminal** permite interactuar con el sistema mediante texto. El **shell** interpreta comandos. **Bash** es uno de los shells disponibles; macOS puede abrir otro shell de forma predeterminada.

### Reconocer y abrir la terminal

Localiza la aplicación de terminal en el equipo. El icono y la apariencia pueden variar con el sistema operativo.

![Icono de la aplicación Terminal utilizado en el material original del curso](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/01terminal.png)

*Figura 1. Icono de referencia para reconocer la aplicación de terminal.*

Al abrirla, identifica el texto que aparece antes de escribir un comando:

![Ventana de terminal con el usuario, el equipo y el prompt del ejemplo original](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/02terminal.png)

*Figura 2. Captura original de la terminal. Los nombres del usuario y del equipo corresponden al ejemplo del profesor.*

Para que los ejemplos se ejecuten en Bash, abre una terminal e inicia:

```bash
bash
printf 'Version de Bash: %s\n' "$BASH_VERSION"
pwd
```

El prompt es el texto que muestra la terminal antes de cada comando. Un símbolo `$` no identifica de forma inequívoca el shell ni sus permisos. En esta guía los bloques de Bash contienen **solo comandos copiables**, sin el prompt. Los resultados esperados aparecen por separado.

Un comando tiene esta estructura:

```text
programa opciones argumentos
```

Por ejemplo, `head -n 4 archivo.fasta` muestra las cuatro primeras líneas del archivo indicado.

### Primeros comandos y lectura de resultados

Desde la terminal, ejecuta:

```bash
date
printf 'Hola, mundo\n'
```

![Captura original con la ejecución de date y echo y sus resultados en la terminal](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/03terminal.png)

*Figura 3. Ejemplo original de comandos y resultados. La fecha y el aspecto del prompt dependen del equipo; cada resultado aparece después del comando que lo genera.*

**Para discutir en clase:** distingue el comando escrito, sus argumentos, la salida y el siguiente prompt.

La siguiente imagen también forma parte de los recursos originales y permite comparar la sintaxis entre lenguajes:

![Ejemplo original de la instrucción print para mostrar hello world en Python](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/hello_world.png)

*Figura 4. La instrucción de esta imagen corresponde a Python. En los ejercicios de Bash de esta unidad utilizamos comandos como `printf`; cada bloque debe ejecutarse con el intérprete correspondiente.*

## 2. Crear un espacio de práctica

El explorador gráfico y la terminal permiten trabajar con el mismo sistema de archivos. Observa la organización en carpetas y archivos de la captura original:

![Explorador Finder con las carpetas y archivos de los ejercicios originales de la unidad](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/finder.png)

*Figura 5. Vista gráfica de archivos y directorios. Los nombres y las rutas de la captura pertenecen al ejemplo original; la práctica siguiente crea un espacio de trabajo propio.*

Ejecuta este bloque desde una carpeta donde puedas trabajar. Crea una carpeta nueva con `mktemp` para evitar mezclar ejercicios de distintas ejecuciones.

```bash
cg_practica=$(mktemp -d "$PWD/cg_bash_27_1.XXXXXX")
cd "$cg_practica"
mkdir -p 00_datos 01_resultados scripts notas
pwd
ls
```

La ruta mostrada por `pwd` será la raíz de esta práctica. Los bloques posteriores, salvo que se indique otra cosa, se ejecutan desde esa raíz. La variable `cg_practica` permite regresar con `cd "$cg_practica"` mientras siga abierta esta sesión de Bash.

| Comando | Uso |
| --- | --- |
| `pwd` | Mostrar el directorio de trabajo actual. |
| `ls` / `ls -lh` | Listar archivos; añadir tamaños y otros detalles. |
| `cd directorio` | Cambiar al directorio indicado. |
| `cd ..` | Subir un nivel. |
| `cd ~` | Ir al directorio personal del usuario. |
| `mkdir -p ruta` | Crear directorios y los niveles intermedios necesarios. |

Una **ruta absoluta** parte de `/`, la raíz del sistema. Una **ruta relativa** se interpreta desde el directorio de trabajo actual. El directorio personal del usuario y la raíz `/` son conceptos diferentes.

```bash
cd 00_datos
pwd
cd ..
pwd
```

**Pregunta:** ¿por qué `cd 00_datos` puede funcionar desde una carpeta y fallar desde otra?

## 3. Generar datos didácticos

Las siguientes secuencias son **artificiales** y sirven exclusivamente para practicar el manejo de archivos. Las etiquetas de organismo no constituyen identificaciones taxonómicas ni datos experimentales.

```bash
cat > 00_datos/demo_a.fasta <<'FASTA'
>demo_01 organismo=Geosmithia_sp. origen=sintetico
ACGTAC
GTACGT
>demo_02 organismo=Geosmithia_sp. origen=sintetico
GGCATTAACCGGTTAA
FASTA

cat > 00_datos/demo_b.fasta <<'FASTA'
>demo_03 organismo=Bacillus_sp. origen=sintetico
TTGGCCAA
>demo_04 organismo=Bacillus_sp. origen=sintetico
ATGCATGCAT
ATGCATGCAT
FASTA
```

Un FASTA contiene registros que comienzan con `>` y una cabecera, seguidos de una o más líneas de secuencia. **Una línea de secuencia no equivale necesariamente a un registro completo.**

## 4. Consultar y contar

```bash
head -n 5 00_datos/demo_a.fasta
tail -n 3 00_datos/demo_b.fasta
wc -l 00_datos/demo_a.fasta
grep -c '^>' 00_datos/demo_a.fasta
grep '^>' 00_datos/demo_a.fasta
```

Resultados de referencia:

- `demo_a.fasta` contiene **5 líneas**, **2 secuencias** y **28 nucleótidos**.
- `demo_b.fasta` contiene **5 líneas**, **2 secuencias** y **28 nucleótidos**.
- El conjunto contiene **4 secuencias** y **56 nucleótidos**.

El patrón `^>` identifica cabeceras al inicio de una línea. Contar líneas y dividir entre dos daría un resultado incorrecto en estos archivos multilínea.

Para explorar archivos grandes se puede utilizar `less archivo.fasta`; la tecla `q` cierra el visor. En FASTQ, contar líneas y dividir entre cuatro solo es válido si los registros tienen exactamente cuatro líneas y el archivo está íntegro; esa cuenta no valida la sincronización entre R1 y R2.

## 5. Comodines y expresiones regulares

Los **comodines del shell** seleccionan nombres de archivos. Las **expresiones regulares** describen patrones dentro del texto. Aunque comparten algunos símbolos, sus reglas son diferentes.

### Seleccionar archivos

```bash
printf '%s\n' 00_datos/*.fasta
printf '%s\n' 00_datos/demo_?.fasta
printf '%s\n' 00_datos/demo_[ab].fasta
```

| Comodín | Significado en un nombre de archivo |
| --- | --- |
| `*` | Cero o más caracteres. |
| `?` | Un carácter. |
| `[ab]` | Un carácter del conjunto indicado. |

Las llaves, como en `{1..3}`, realizan una expansión de Bash; no buscan archivos existentes.

### Buscar dentro de archivos

```bash
grep '^>' 00_datos/demo_a.fasta
grep -i 'geosmithia' 00_datos/demo_a.fasta
grep -l 'Geosmithia' 00_datos/*.fasta
grep -E '^>demo_0[12][[:space:]]' 00_datos/demo_a.fasta
```

| Opción o patrón | Interpretación |
| --- | --- |
| `grep -c` | Contar líneas que coinciden, no todas las ocurrencias del patrón. |
| `grep -l` | Mostrar los nombres de los archivos con coincidencias. |
| `grep -i` | Ignorar diferencias entre mayúsculas y minúsculas. |
| `grep -E` | Utilizar expresiones regulares extendidas. |
| `^` / `$` | Inicio/final de una línea. |
| `[[:space:]]` | Carácter de espacio en blanco. |
| `[[:alnum:]_]` | Letra, dígito o guion bajo. |

Los caracteres `<` y `>` sin escape no son anclas generales de inicio/final de palabra. Para estos ejercicios se utilizan patrones de línea y clases de caracteres disponibles tanto en GNU/Linux como en macOS.

Buscar un motivo directamente con `grep` no detecta coincidencias que atraviesen un salto de línea. Para análisis biológicos se necesitan secuencias completas y criterios explícitos sobre hebra, ambigüedades y solapamiento de coincidencias.

## 6. Redirección y tuberías

```bash
grep '^>' 00_datos/demo_a.fasta > 01_resultados/cabeceras.txt
grep '^>' 00_datos/demo_b.fasta >> 01_resultados/cabeceras.txt
cat 01_resultados/cabeceras.txt
grep '^>' 00_datos/demo_a.fasta | sed 's/^>//' | cut -d ' ' -f 1
```

- `>` crea un archivo o reemplaza su contenido.
- `>>` añade contenido al final.
- La tubería `|` conecta la salida de un comando con la entrada del siguiente.

![Ilustración original de Mario y tuberías utilizada para introducir el operador pipe](https://img.youtube.com/vi/uMCCxuGIGtw/hqdefault.jpg)

*Figura 6. Recurso visual original para introducir las tuberías. En Bash, `|` permite que la salida de un programa sea procesada por el siguiente.*

El último comando debe mostrar `demo_01` y `demo_02`, cada identificador en una línea. Si repites todo el bloque, el primer comando reinicia el archivo de cabeceras y el segundo añade las restantes, sin acumular duplicados de ejecuciones anteriores.

## 7. Copiar, mover y eliminar archivos de práctica

```bash
cp 00_datos/demo_a.fasta 01_resultados/copia_demo_a.fasta
mv 01_resultados/copia_demo_a.fasta 01_resultados/copia_para_revisar.fasta
ls -lh 01_resultados
rm -i 01_resultados/copia_para_revisar.fasta
```

`cp` copia; `mv` mueve o renombra; `rm` elimina. El modificador `-i` solicita confirmación antes de borrar. Practica únicamente con las copias generadas aquí. En la terminal, `rm` normalmente no envía los archivos a la papelera.

`touch archivo.txt` crea un archivo vacío si no existe; si ya existe, actualiza sus marcas de tiempo sin vaciar su contenido.

## 8. Archivar y comprimir

`tar` agrupa archivos en un archivo contenedor. La opción `-z` utiliza gzip para comprimirlo; la relación de compresión depende del contenido.

```bash
tar -czf 01_resultados/datos_demo.tar.gz 00_datos
tar -tzf 01_resultados/datos_demo.tar.gz
mkdir -p 01_resultados/extraidos
tar -xzf 01_resultados/datos_demo.tar.gz -C 01_resultados/extraidos
diff 00_datos/demo_a.fasta 01_resultados/extraidos/00_datos/demo_a.fasta
```

| Opción | Función |
| --- | --- |
| `-c` | Crear el archivo contenedor. |
| `-t` | Listar su contenido. |
| `-x` | Extraer. |
| `-z` | Usar gzip. |
| `-f` | Indicar el nombre del archivo contenedor. |
| `-C` | Cambiar al directorio de destino para la operación. |

Si `diff` no produce salida y finaliza correctamente, las dos copias comparadas son idénticas.

![Viñeta original de XKCD sobre la dificultad de recordar las opciones de tar](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/tar_xkcd.png)

*Figura 7. Viñeta conservada del material original. El objetivo de la práctica es comprender las opciones y consultar la ayuda cuando sea necesario.*

## 9. Variables y bucles

Una variable guarda un valor. No se escriben espacios alrededor del signo `=` en una asignación de Bash. Las comillas dobles permiten expandir variables y conservar los espacios contenidos en una ruta.

```bash
cg_etapa='revision_inicial'
printf 'Etapa: %s\n' "$cg_etapa"

for cg_fasta in 00_datos/*.fasta; do
    printf 'Archivo: %s\n' "$cg_fasta"
    grep -c '^>' "$cg_fasta"
done
```

En este ejercicio existen archivos que coinciden con el patrón. En un script general, también debe comprobarse qué hacer si no hay coincidencias.

### Arreglos de Bash: ampliación opcional

```bash
cg_archivos=(00_datos/demo_a.fasta 00_datos/demo_b.fasta)
for cg_fasta in "${cg_archivos[@]}"; do
    printf '%s\n' "$cg_fasta"
done
```

La expansión `"${cg_archivos[@]}"` conserva cada elemento por separado. Para recorrer archivos no se recomienda analizar la salida de `ls` ni usar `for archivo in $(cat lista.txt)` cuando los nombres pueden contener espacios.

## 10. Contar nucleótidos en FASTA multilínea

Ejecuta este bloque desde la raíz de la práctica:

```bash
awk '
BEGIN { OFS="\t"; print "secuencia", "longitud_nt" }
/^>/ {
    if (id != "") print id, longitud
    id = substr($0, 2)
    sub(/[[:space:]].*/, "", id)
    longitud = 0
    next
}
{
    gsub(/[[:space:]]/, "")
    longitud += length($0)
}
END { if (id != "") print id, longitud }
' 00_datos/demo_a.fasta 00_datos/demo_b.fasta > 01_resultados/longitudes.tsv

cat 01_resultados/longitudes.tsv
```

Resultado esperado, con columnas separadas por tabuladores:

```text
secuencia	longitud_nt
demo_01	12
demo_02	16
demo_03	8
demo_04	20
```

Este ejemplo suma las líneas de cada registro. Supone archivos FASTA estructuralmente válidos; no comprueba por sí solo el alfabeto biológico ni la calidad de las secuencias.

## 11. Escribir un script reutilizable

Un script es un archivo de texto con instrucciones. El directorio de trabajo normalmente se **hereda del proceso que lo ejecuta**; no cambia automáticamente al directorio donde se guarda el script. En el siguiente ejemplo, las rutas se reciben como argumentos.

Guarda el bloque completo:

```bash
cat > scripts/resumir_fasta.sh <<'BASH'
#!/usr/bin/env bash
set -euo pipefail

if [ "$#" -ne 2 ]; then
    printf 'Uso: bash %s DIRECTORIO_FASTA SALIDA_TSV\n' "$0" >&2
    exit 2
fi

cg_entrada=$1
cg_salida=$2

if [ ! -d "$cg_entrada" ]; then
    printf 'No existe el directorio: %s\n' "$cg_entrada" >&2
    exit 1
fi

shopt -s nullglob
cg_archivos=("$cg_entrada"/*.fasta)

if [ "${#cg_archivos[@]}" -eq 0 ]; then
    printf 'No se encontraron archivos .fasta en %s\n' "$cg_entrada" >&2
    exit 1
fi

for cg_fasta in "${cg_archivos[@]}"; do
    if ! grep -q '^>' "$cg_fasta"; then
        printf 'El archivo no contiene cabeceras FASTA: %s\n' "$cg_fasta" >&2
        exit 1
    fi
done

mkdir -p "$(dirname "$cg_salida")"
printf 'archivo\tsecuencias\tbases_nt\n' > "$cg_salida"

for cg_fasta in "${cg_archivos[@]}"; do
    cg_resumen=$(awk '
        /^>/ { secuencias++; next }
        { gsub(/[[:space:]]/, ""); bases += length($0) }
        END { printf "%d\t%d", secuencias, bases }
    ' "$cg_fasta")
    printf '%s\t%s\n' "$(basename "$cg_fasta")" "$cg_resumen" >> "$cg_salida"
done

printf 'Resumen guardado en %s\n' "$cg_salida"
BASH
```

El script resume archivos con extensión `.fasta` y reemplaza la tabla de salida en cada ejecución. `set -euo pipefail` ayuda a detectar errores de ejecución, variables no definidas y fallos en tuberías; no demuestra que un resultado sea biológicamente correcto.

### Comprobar la sintaxis y ejecutar

```bash
bash -n scripts/resumir_fasta.sh
bash scripts/resumir_fasta.sh 00_datos 01_resultados/resumen_fasta.tsv
cat 01_resultados/resumen_fasta.tsv
```

Resultado esperado:

```text
archivo	secuencias	bases_nt
demo_a.fasta	2	28
demo_b.fasta	2	28
```

Para ejecutarlo directamente, concede el permiso y después ejecútalo en una segunda línea:

```bash
chmod +x scripts/resumir_fasta.sh
./scripts/resumir_fasta.sh 00_datos 01_resultados/resumen_fasta.tsv
```

El permiso de ejecución no es necesario cuando se invoca explícitamente con `bash archivo.sh` y el archivo puede leerse. La línea `#!` indica el intérprete para la ejecución directa.

### Reconocer los permisos en una lista de archivos

![Esquema original de los permisos de un archivo Unix: tipo, propietario, grupo y otros usuarios](https://raw.githubusercontent.com/AliciaMstt/BioinfinvRepro/master/Unidad1/permissions.png)

*Figura 8. Esquema de permisos utilizado en el material de origen. La imagen se recuperó del repositorio de Alicia Mastretta Yanes, porque `permissions.png` no estaba incluido en el árbol de IntroGenomics revisado.*

En una salida de `ls -l`, identifica el tipo de archivo y los permisos de lectura (`r`), escritura (`w`) y ejecución (`x`) para propietario, grupo y otros usuarios. Compara el resultado antes y después de `chmod +x`:

```bash
ls -l scripts/resumir_fasta.sh
```

**Pregunta:** ¿qué cambia al dar permiso de ejecución y por qué ese permiso no demuestra que el script sea correcto?

## 12. Descargar secuencias públicas de NCBI

Actividad adicional con conexión. Se conservan las accesiones del ejemplo original de *Geosmithia*, ahora con versiones explícitas:

| Accesión | Registro consultado el 19/10/2026 |
| --- | --- |
| AM181431.2 | Geosmithia sp. CCF3557. |
| AM947671.1 | Geosmithia cnesini, cepa MK 1820. |
| KF808310.1 | Geosmithia sp. 21 NL-2014, cepa MK1665. |

Estos registros corresponden a regiones ribosomales con ITS y segmentos asociados; no son genomas completos. Para estudios taxonómicos debe verificarse además el estatus del material y la correspondencia de los marcadores.

![Captura original de un editor de texto con el script para descargar secuencias de Geosmithia](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/get_seq.png)

*Figura 9. Captura original del script de descarga: permite reconocer comentarios, comandos y rutas. Se conserva como apoyo visual; el bloque de ejecución actualizado aparece a continuación. La captura utiliza Atom, un editor histórico del material.*

Desde la raíz de la práctica:

```bash
curl --fail --location --silent --show-error \
    --retry 2 --connect-timeout 20 --max-time 90 \
    'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nucleotide&rettype=fasta&retmode=text&id=AM181431.2,AM947671.1,KF808310.1' \
    --output 00_datos/Geosmithia_NCBI.fasta

grep '^>' 00_datos/Geosmithia_NCBI.fasta
grep -c '^>' 00_datos/Geosmithia_NCBI.fasta
```

La descarga consultada durante esta revisión devolvió tres registros. Si la conexión falla o no se obtienen tres cabeceras, revisar el archivo antes de continuar. Los archivos públicos añadidos cambiarán los totales del script anterior: los valores de referencia de la práctica principal corresponden solamente a `demo_a.fasta` y `demo_b.fasta`.

Para descargar código o texto desde GitHub se utiliza el contenido **raw**, no una página `/edit/`. Consultar el [manual de curl](https://curl.se/docs/manpage.html) y la [documentación de E-utilities](https://www.ncbi.nlm.nih.gov/books/NBK25499/).

## 13. Actividades de entrega

1. Explica por qué contar líneas y dividir entre dos no permite contar los registros de estos FASTA.
2. Genera una lista con los cuatro identificadores sintéticos, sin el símbolo `>` ni sus descripciones.
3. Entrega `resumir_fasta.sh` y la tabla de resultados. Comprueba que el script produce los mismos conteos cuando una ruta contiene espacios.
4. Modifica una copia de un FASTA sintético para distribuir su secuencia en más líneas. Demuestra que la longitud total se conserva.
5. En la actividad de NCBI, documenta las accesiones y la fecha de consulta. Explica por qué un identificador de secuencia no prueba por sí solo la identidad taxonómica.

La bitácora deberá registrar la pregunta del ejercicio, el procedimiento, el resultado y una explicación breve. No es necesario adjuntar registros extensos de la terminal si no aportan evidencia adicional.

## 14. Resolución de problemas

| Mensaje o situación | Comprobación |
| --- | --- |
| `command not found` | Revisar el nombre del programa y comprobar su instalación con `command -v programa`. |
| `No such file or directory` | Consultar `pwd`, listar la carpeta y revisar mayúsculas, extensión y ruta. |
| Un nombre con espacios se divide | Rodear la expansión de la variable con comillas dobles. |
| El script funciona en una terminal y falla en otra | Iniciar Bash explícitamente y verificar el entorno. |
| `Permission denied` al ejecutar directamente | Revisar permisos o invocar el archivo legible mediante `bash`. |
| La tabla tiene valores inesperados tras descargar NCBI | Comprobar qué archivos `.fasta` se incluyeron; el script procesa todos los del directorio indicado. |

## Documentación y créditos

- [Manual de Bash](https://www.gnu.org/software/bash/manual/bash.html).
- [Manual de grep](https://www.gnu.org/software/grep/manual/grep.html).
- [Manual de GNU tar](https://www.gnu.org/software/tar/manual/tar.html).
- [Manual de curl](https://curl.se/docs/manpage.html).
- Material introductorio original del curso, que reconoce adaptaciones de recursos de [Alicia Mastretta Yanes](https://github.com/AliciaMstt/BioinfinvRepro).

Las imágenes del repositorio original se conservan en sus secciones correspondientes, con texto alternativo y pies de figura. Los enlaces completos permiten visualizarlas desde este Markdown y al incorporarlo al repositorio. Las capturas históricas acompañan la explicación; los bloques de código indican los comandos de la práctica actualizada.

![Imagen de cierre To be continued conservada de los recursos originales del curso](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/to_be_continued.png)

*Figura 10. Imagen original de cierre de la unidad.*

**Actualización:** 19 de octubre de 2026.
