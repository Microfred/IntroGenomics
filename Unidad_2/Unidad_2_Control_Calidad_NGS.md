# Unidad 2. Control de calidad y procesamiento de lecturas NGS

## Illumina short reads y lecturas largas de Oxford Nanopore y PacBio

**Curso:** Ciencias Genómicas, ENCB-IPN, semestre 27-1.  
**Profesor:** Dr. Juan Alfredo Hernández García.  
**Material práctico:** [IntroGenomics](https://github.com/Microfred/IntroGenomics).  
**Ubicación sugerida en el repositorio:** `Unidad_2/Readme_U2.md`.  
**Actualización y consulta de recursos:** 19 de octubre de 2026.

Esta unidad explica cómo evaluar datos de secuenciación, decidir si requieren procesamiento y documentar el efecto de cada intervención. Se conservan **las doce imágenes del material original**, con títulos y pies acordes con el contenido de cada gráfica.

**Ruta de lectura:** [Fundamentos](#fundamentos) · [Herramientas](#herramientas) · [Preparación](#preparacion) · [FastQC e imágenes](#fastqc) · [Illumina y fastp](#illumina) · [MultiQC](#multiqc) · [Long reads](#longreads) · [PacBio HiFi](#hifi) · [Entregables](#entregables) · [Recursos oficiales](#recursos).

## Objetivos de aprendizaje

Al finalizar la unidad podrás:

1. Distinguir diagnóstico de calidad, filtrado, recorte, corrección e integración de informes.
2. Interpretar calidad Phred, contenido de bases, GC, adaptadores, duplicación y distribución de longitudes.
3. Utilizar FastQC, fastp y MultiQC con lecturas cortas de Illumina.
4. Seleccionar herramientas específicas para evaluar y procesar lecturas largas.
5. Comparar datos iniciales y procesados mediante número de lecturas, bases, calidad y longitud.
6. Justificar los parámetros según tecnología, biblioteca, organismo y análisis posterior.

<a id="fundamentos"></a>
## 1. Qué significa controlar la calidad

Los archivos entregados por un servicio de secuenciación pueden contener adaptadores, cebadores, lecturas cortas, bases ambiguas, secuencias de baja calidad o material ajeno al objetivo del estudio. El control de calidad busca identificar estos problemas y valorar su importancia para una pregunta biológica.

| Operación | Qué hace | Ejemplo |
| --- | --- | --- |
| Evaluación o diagnóstico | Describe los datos y señala patrones que requieren interpretación. | FastQC, NanoPlot, SeqKit. |
| Filtrado | Conserva o excluye lecturas según un criterio. | Longitud mínima o calidad por lectura. |
| Recorte, trimming | Retira segmentos de una lectura. | Adaptadores o extremos de baja calidad. |
| Corrección | Modifica bases mediante un modelo o evidencia adicional. | Corrección de bases por solapamiento, si se activa y justifica. |
| Integración de informes | Reúne resultados de herramientas y muestras. | MultiQC. |

**FastQC y MultiQC no limpian las secuencias.** Generan o reúnen diagnósticos. Una mejora visual en un informe tampoco demuestra por sí sola que el conjunto resultante sea adecuado para el análisis.

La secuencia de trabajo será:

1. Identificar los datos, la plataforma y el diseño de biblioteca.
2. Comprobar integridad, formato y correspondencia entre archivos.
3. Evaluar las lecturas originales.
4. Seleccionar y justificar un procesamiento, si se necesita.
5. Evaluar nuevamente y cuantificar lo que se retuvo o descartó.
6. Conservar informes, versiones, parámetros y decisiones.

## 2. FASTQ, calidad Phred y métricas fundamentales

### 2.1 Estructura de un registro FASTQ

En su representación habitual de cuatro líneas, un registro contiene identificador, secuencia, separador y cadena de calidades:

```text
@lectura_demo
ACGTACGT
+
IIIIIIII
```

La secuencia y las calidades deben tener la misma longitud. En Phred+33, el carácter `I` representa Q40. Este ejemplo es artificial y solo ilustra la estructura del archivo.

Contar líneas y dividir entre cuatro solo cuenta correctamente registros FASTQ de cuatro líneas íntegros. No valida por sí solo el formato, las calidades ni la sincronización R1/R2. No deben contarse lecturas buscando todas las líneas que empiezan con `@`, porque ese carácter también puede aparecer en las calidades.

### 2.2 Relación entre calidad y probabilidad de error

La escala Phred expresa una probabilidad de error estimada:

**Q = -10 log10(Perror)**; por tanto, **Perror = 10^(-Q/10)**.

| Calidad | Probabilidad estimada de error por base | Porcentaje de error |
| --- | --- | --- |
| Q10 | 0.1 | 10% |
| Q20 | 0.01 | 1% |
| Q30 | 0.001 | 0.1% |
| Q40 | 0.0001 | 0.01% |

Estos valores describen una predicción del modelo de asignación de bases. La exactitud observada requiere otra evidencia, como comparación con una referencia apropiada. El porcentaje Q30 es la proporción de bases con calidad **al menos Q30**, no el porcentaje de lecturas que carecen de errores.

La media aritmética de valores Phred y la calidad obtenida al transformar la probabilidad media de error no son equivalentes. Para comparar programas, comprobar cómo calculan la calidad por lectura y evitar interpretar promedios distintos como si fueran la misma métrica.

### 2.3 Diferencias entre plataformas

| Característica | Illumina short reads | Oxford Nanopore | PacBio HiFi |
| --- | --- | --- | --- |
| Longitudes | Frecuentemente definidas por los ciclos; R1/R2 en bibliotecas paired-end. | Distribución amplia, relacionada con la muestra y preparación. | Lecturas largas de alta exactitud; distribución dependiente de biblioteca y preparación. |
| Calidad que conviene revisar | Calidad por ciclo, R1 frente a R2, adaptadores y retención de pares. | Calidad por lectura, longitud, rendimiento y, si existe información, evolución de la corrida. | Calidad de consenso, rendimiento y distribución de longitudes. |
| Diagnóstico principal de esta unidad | FastQC, SeqKit, fastp y MultiQC. | NanoPlot y SeqKit; procesamiento seleccionado según necesidad. | Estadísticas y diagnósticos adecuados al producto HiFi. |
| Decisión que debe evitarse | Recortar hasta que todos los módulos aparezcan verdes. | Aplicar automáticamente umbrales y longitud fija de Illumina. | Tratar lecturas HiFi como si fueran subreads crudas de mayor error. |

El número de bases retenidas importa tanto como el porcentaje de lecturas. Para un genoma aislado de tamaño estimado G, una aproximación de cobertura de secuenciación es **C = bases retenidas / G**. En metagenomas, este cociente no representa la cobertura de cada organismo de la comunidad.

<a id="herramientas"></a>
## 3. Selección de herramientas

| Herramienta | Función | Datos o situación principal | Límite de interpretación |
| --- | --- | --- | --- |
| **FastQC** | Diagnóstico por archivo. | Especialmente útil para bibliotecas de lecturas cortas. | Sus indicadores dependen del tipo de biblioteca y de la versión. |
| **fastp** | QC, filtrado y recorte con informes HTML/JSON. | Short reads, incluyendo Illumina paired-end. | No trasladar sus parámetros de short reads a lecturas largas. |
| **MultiQC** | Integración de informes compatibles. | Comparación de muestras, etapas y herramientas. | No ejecuta el procesamiento ni interpreta automáticamente el diseño. |
| **SeqKit** | Lectura y estadísticas de FASTA/FASTQ. | Datos cortos o largos; conteos y longitudes. | Las estadísticas no identifican por sí solas adaptadores o contaminación. |
| **Cutadapt** | Retirada de adaptadores y cebadores especificados. | Bibliotecas con secuencias técnicas conocidas, incluidos amplicones. | Las secuencias y su orientación deben corresponder al protocolo real. |
| **NanoPlot** | Gráficas y estadísticas de lecturas largas. | FASTQ, BAM y otros formatos admitidos. | Filtrar lo mostrado en una gráfica no genera automáticamente un FASTQ limpio. |
| **fastplong** | Preprocesamiento de lecturas largas con informes HTML/JSON. | ONT y otros datos largos compatibles. | Revisar adaptadores, partición de lecturas y opciones de la versión instalada. |
| **Chopper** | Filtrado y estrategias de recorte para long reads. | ONT/PacBio en FASTQ. | Un umbral de longitud o calidad puede excluir moléculas informativas. |
| **Filtlong** | Selección de lecturas largas por longitud y calidad. | Subconjuntos de long reads; puede utilizar evidencia externa. | La selección modifica la distribución de longitudes, bases y representación. |
| **Dorado** | Asignación de bases y procesamiento asociado de ONT. | Señal y productos de Oxford Nanopore. | Revisar qué se hizo antes de recibir los FASTQ; requiere entradas y modelos apropiados. |
| **Cramino** | Evaluación rápida de BAM/CRAM de lecturas largas. | Cuando se dispone de esos formatos y se necesitan sus métricas. | No equivale a un preprocesador general de FASTQ. |

**Herramienta histórica:** NanoStat sigue disponible y sus informes tienen soporte en MultiQC. Su autor indica que ha sido ampliamente sustituido por Cramino y que probablemente no recibirá nuevas actualizaciones. Para las gráficas de esta unidad utilizaremos NanoPlot, que también produce un archivo de estadísticas; no es necesario instalar NanoStat para ejecutar esa ruta.

Las páginas, repositorios y documentación de estas herramientas aparecen al final de la unidad, con la fecha de comprobación.

<a id="preparacion"></a>
## 4. Preparación del proyecto y los entornos

### 4.1 Organización de trabajo

Los comandos suponen una terminal Bash y una carpeta de proyecto propia. Adaptar los nombres de ejemplo a los archivos reales, manteniendo la trazabilidad de sus nombres originales.

```bash
mkdir -p proyecto_qc_ngs
cd proyecto_qc_ngs
mkdir -p 00_raw/illumina 00_raw/longreads 01_qc/illumina/raw 01_qc/illumina/clean
mkdir -p 01_qc/illumina/fastp 01_qc/longreads 02_clean/illumina 02_clean/longreads
mkdir -p 03_multiqc metadatos logs scripts
```

Los bloques posteriores se ejecutan desde esta raíz. Los ejemplos utilizan:

- `00_raw/illumina/S01_R1.fastq.gz` y `S01_R2.fastq.gz`: un par Illumina.
- `00_raw/longreads/ONT01.fastq.gz`: un conjunto de lecturas largas ONT.

Antes de descargar o copiar datos, registrar muestra, accesiones, plataforma, química, tipo de biblioteca, read layout y procesamiento previo. En ONT, registrar además el modelo de basecalling cuando se conozca; en PacBio, identificar si el producto corresponde a HiFi u otro tipo de lectura.

### 4.2 Instalación mediante entornos separados

Con una instalación de Conda/Mamba preparada y compatible con el equipo:

```bash
conda create -n cg_qc_short --override-channels --strict-channel-priority \
    -c conda-forge -c bioconda fastqc fastp multiqc seqkit cutadapt

conda create -n cg_qc_long --override-channels --strict-channel-priority \
    -c conda-forge -c bioconda nanoplot chopper filtlong fastplong seqkit multiqc
```

No se requieren los canales `broken` o etiquetas antiguas incluidos en el material previo. El uso de `apt` corresponde a distribuciones Linux que lo admiten; no es un método general para macOS. La resolución de paquetes debe comprobarse para la plataforma concreta.

Después de instalar el entorno de lecturas cortas:

```bash
conda activate cg_qc_short
fastqc --version > metadatos/fastqc_version.txt
fastp --version > metadatos/fastp_version.txt 2>&1
multiqc --version > metadatos/multiqc_version.txt
seqkit version > metadatos/seqkit_version.txt
conda list --explicit > metadatos/cg_qc_short_explicito.txt
conda env export --from-history > metadatos/cg_qc_short_historial.yml
```

Los comandos de creación no fijan una versión concreta. El registro de la resolución efectiva permite documentar lo que realmente se utilizó. Un archivo explícito de Conda corresponde a una plataforma y no garantiza reproducibilidad binaria entre arquitecturas diferentes.

## 5. Integridad, formato y correspondencia de lecturas

Para el ejemplo Illumina:

```bash
gzip -t 00_raw/illumina/S01_R1.fastq.gz 00_raw/illumina/S01_R2.fastq.gz
seqkit stats -a -T 00_raw/illumina/S01_R1.fastq.gz 00_raw/illumina/S01_R2.fastq.gz \
    > metadatos/S01_estadisticas_raw.tsv
```

Si una comprobación falla, resolver el problema antes de continuar. `gzip -t` comprueba la integridad del contenedor comprimido; no valida por sí solo la estructura FASTQ. Las sumas de comprobación deben contrastarse con las proporcionadas por el repositorio o servicio, si están disponibles.

En datos paired-end se requiere:

1. Mismo número de registros en R1 y R2.
2. Identificadores correspondientes en el mismo orden, interpretando las convenciones de cabecera.
3. Correspondencia entre muestra, biblioteca, lane y dirección de lectura.

La igualdad del número de lecturas no demuestra por sí sola sincronización. Si hay archivos de distintos lanes, su unión debe mantener el orden y la correspondencia por muestra y dirección. Las lecturas sin pareja se gestionan como tales; añadirlas a un archivo paired-end no crea nuevos pares válidos.

<a id="fastqc"></a>
## 6. FastQC: ejecución e interpretación con las imágenes originales

### 6.1 Abrir archivos desde la interfaz gráfica

FastQC puede ejecutarse sin argumentos para abrir su interfaz, cuando existe un entorno gráfico compatible:

```bash
fastqc
```

Después seleccionar **File > Open** y abrir el archivo correspondiente.

![FastQC: menú File y opción Open en la captura original del curso](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_1.png)

*Figura 1 original. Apertura de archivos en FastQC. La captura corresponde a una versión anterior; los conceptos de lectura e interpretación se mantienen.*

![FastQC abierto desde la terminal en la captura original](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_2.png)

*Figura 2 original. Inicio de FastQC desde la terminal y apertura de su ventana gráfica.*

Para ejecutar un análisis sin interfaz gráfica y guardar resultados:

```bash
fastqc --threads 2 --outdir 01_qc/illumina/raw \
    00_raw/illumina/S01_R1.fastq.gz 00_raw/illumina/S01_R2.fastq.gz
```

FastQC genera informes HTML y archivos ZIP con resultados detallados. No es necesario abrir la interfaz para producirlos. Los formatos aceptados deben comprobarse en la versión utilizada; en esta práctica la entrada es FASTQ comprimido.

### 6.2 Indicadores y estadísticas básicas

El resumen utiliza indicadores **pass**, **warn** y **fail**. Son señales para investigar, no un diagnóstico universal de aptitud de la muestra.

Revisar nombre, formato, codificación de calidades, número de secuencias, intervalo de longitudes y GC global. En bibliotecas de baja diversidad, amplicones, RNA-seq o mezclas de organismos, varios supuestos de una biblioteca genómica aleatoria pueden no cumplirse.

Los umbrales y módulos pueden variar con la versión y configuración. Deben documentarse, evitando convertir todos los indicadores en criterios automáticos de descarte.

### 6.3 Calidad por posición: Per base sequence quality

![FastQC: calidad por base y caída de calidad hacia el final de la lectura](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_3.png)

*Figura 3 original. Distribución de calidad por posición en una lectura R2. La parte final muestra mayor dispersión y pérdida de calidad.*

En el gráfico habitual:

- La línea roja indica la **mediana**.
- La caja amarilla representa los percentiles **25 y 75**.
- Los bigotes representan los percentiles **10 y 90**.
- La línea azul indica la **media**.
- El eje horizontal representa posiciones o intervalos de posiciones, y el vertical calidades Phred.

Para decidir un recorte, identificar dónde empieza el deterioro y cuántas bases y lecturas se perderían. Comparar R1 con R2. En amplicones paired-end debe considerarse el solapamiento necesario para unir lecturas; en ensamblaje, la longitud útil y cobertura retenidas.

### 6.4 Calidad por lectura: Per sequence quality scores

![FastQC: distribución de la calidad media por secuencia](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_4.png)

*Figura 4 original. Distribución de calidad por secuencia. Una subpoblación con menor calidad merece investigación junto con los demás diagnósticos.*

Este módulo resume la distribución de calidades por lectura. No describe la misma propiedad que el gráfico por posición. Un segundo pico de menor calidad puede corresponder a un subconjunto del archivo, pero su causa requiere revisar biblioteca, corrida y metadatos; el gráfico aislado no identifica el origen del problema.

### 6.5 Composición por posición: Per base sequence content

![FastQC: proporciones de A, C, G y T a lo largo de las lecturas](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_5.png)

*Figura 5 original. Composición de bases por posición. Las diferencias deben interpretarse según la preparación de biblioteca y el contenido del organismo o comunidad.*

En una biblioteca genómica aleatoria pueden esperarse proporciones relativamente estables a lo largo de la lectura. **No se requiere que cada base represente exactamente 25%**. Un sesgo al inicio puede relacionarse con preparación, cebadores o transposición; en amplicones es esperable una composición no aleatoria.

Un patrón inusual sugiere hipótesis que deben contrastarse; no demuestra por sí solo contaminación ni justifica eliminar un número fijo de bases.

### 6.6 GC por posición: capturas históricas

![FastQC: porcentaje de GC en cada posición, captura original 6](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_6.png)

*Figura 6 original. GC a lo largo de las posiciones de la lectura.*

![FastQC: porcentaje de GC en cada posición, captura original 7](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_7.png)

*Figura 7 original. Segunda captura conservada del mismo tipo de diagnóstico: GC por posición.*

Ambas imágenes muestran **GC por posición**, como indica su eje horizontal; no son distribuciones de GC entre lecturas. Se conservan las dos para la explicación en clase. La presencia de este panel depende de la versión del informe; actualmente puede ser necesario consultar otros paneles de composición.

### 6.7 GC entre lecturas: Per sequence GC content

![FastQC: distribución del GC medio por lectura frente a una distribución teórica](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_10.png)

*Figura 10 original, ubicada en su sección correspondiente. La curva roja representa la distribución observada de GC por lectura y la azul una distribución teórica. Esta imagen no muestra duplicación de secuencias.*

Una forma multimodal o una desviación respecto al modelo puede relacionarse con mezcla de organismos, fracciones enriquecidas, secuencias técnicas o contaminación. En metagenomas no debe imponerse un único pico como requisito de calidad. FastQC tampoco conoce de antemano el GC verdadero del organismo para certificar su identidad.

### 6.8 Bases ambiguas: Per base N content

![FastQC: porcentaje de bases N a lo largo de las lecturas](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_8.png)

*Figura 8 original. La proporción de N aumenta en un intervalo de posiciones de la lectura.*

Una N representa una base no resuelta. Revisar si su aparición es aislada, concentrada en ciclos o extendida a muchas lecturas. Debe distinguirse una **base asignada con calidad baja** de una **base ambigua**; no son métricas intercambiables.

### 6.9 Distribución de longitudes

![FastQC: distribución de longitudes de las secuencias](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_9.png)

*Figura 9 original. Distribución de longitudes con lecturas de tamaños diferentes. La plataforma y el procesamiento deben conocerse antes de decidir si ese patrón es esperado.*

La variación de longitud puede ser consecuencia del recorte o una propiedad normal de la biblioteca. Es especialmente esperable en muchas preparaciones de lecturas largas. Una advertencia por longitudes variables no invalida automáticamente los datos.

### 6.10 Duplicación de secuencias

La duplicación aparente puede relacionarse con PCR, profundidad elevada o abundancia biológica real. Por ejemplo, un transcrito abundante o una secuencia de amplicón frecuente puede aparecer muchas veces sin representar un error de secuenciación.

La duplicación estimada por secuencia en FastQC no equivale a marcar duplicados utilizando coordenadas de alineamiento o UMI. No activar deduplicación como tratamiento universal; en microbiomas o transcriptómica puede alterar la señal biológica.

### 6.11 Secuencias sobrerrepresentadas

Investigar las secuencias frecuentes y su posible relación con adaptadores, cebadores, controles, organismo objetivo o contaminantes. La coincidencia con una secuencia de la lista interna de FastQC orienta una revisión, pero no constituye identificación taxonómica concluyente.

### 6.12 Contenido de adaptadores y calidad por tile

El panel de adaptadores ayuda a detectar secuencias técnicas residuales. El panel por tile, cuando puede calcularse, permite investigar heterogeneidad espacial de la corrida. La disponibilidad de información depende de las cabeceras y del formato entregado.

Estos diagnósticos complementan las imágenes originales. Si el proveedor ya procesó los datos, registrar qué adaptadores, cebadores o índices fueron retirados antes de repetir el tratamiento.

### 6.13 K-mers y enriquecimiento de motivos

Un k-mer es una subcadena de longitud k. Una secuencia de longitud L contiene **L-k+1 posiciones posibles** para k-mers solapados, cuando L es al menos k; esto no equivale al número de k-mers distintos.

![Ilustración original del concepto de k-mer](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/Kmer.png)

*Imagen Kmer original. Recurso para explicar fragmentos cortos de una secuencia.*

![FastQC: enriquecimiento relativo de k-mers a lo largo de la lectura](https://raw.githubusercontent.com/Microfred/IntroGenomics/main/Unidad_2/figura_11.png)

*Figura 11 original. Enriquecimiento de motivos en diferentes posiciones de la lectura.*

Un motivo enriquecido puede corresponder a secuencia técnica o a una propiedad biológica de la biblioteca. El tamaño de k y la disponibilidad del análisis dependen del programa y su versión. No asumir que todas las versiones actuales de FastQC producen exactamente los paneles históricos mostrados aquí.

<a id="illumina"></a>
## 7. Ruta práctica para Illumina: FastQC, fastp y evaluación posterior

### 7.1 Procesar una muestra paired-end con fastp

Este ejemplo se orienta a una biblioteca genómica de lecturas cortas. Los umbrales son **ilustrativos**, no criterios universales de aceptación.

```bash
conda activate cg_qc_short

fastp \
    --in1 00_raw/illumina/S01_R1.fastq.gz \
    --in2 00_raw/illumina/S01_R2.fastq.gz \
    --out1 02_clean/illumina/S01_R1.clean.fastq.gz \
    --out2 02_clean/illumina/S01_R2.clean.fastq.gz \
    --unpaired1 02_clean/illumina/S01_R1.unpaired.fastq.gz \
    --unpaired2 02_clean/illumina/S01_R2.unpaired.fastq.gz \
    --detect_adapter_for_pe \
    --qualified_quality_phred 20 --unqualified_percent_limit 40 \
    --n_base_limit 5 --length_required 50 \
    --html 01_qc/illumina/fastp/S01.fastp.html \
    --json 01_qc/illumina/fastp/S01.fastp.json \
    --thread 4 --dont_overwrite \
    > logs/S01_fastp.stdout.log 2> logs/S01_fastp.stderr.log
```

| Opción | Qué significa en este ejemplo |
| --- | --- |
| `--in1`, `--in2` | Archivos originales de ambos extremos. |
| `--out1`, `--out2` | Lecturas que conservan la condición de pares. |
| `--unpaired1`, `--unpaired2` | Lecturas aprobadas cuyo compañero no pasó los filtros. |
| `--detect_adapter_for_pe` | Activar detección adicional de adaptadores para paired-end. |
| `--qualified_quality_phred 20` | Considerar de calidad suficiente una base con Q al menos 20 para este criterio. |
| `--unqualified_percent_limit 40` | Excluir según la proporción máxima permitida de bases por debajo del umbral; no exige que todas las bases tengan Q20. |
| `--n_base_limit 5` | Limitar el número de bases ambiguas por lectura según el criterio del programa. |
| `--length_required 50` | Exigir al menos 50 nt después del procesamiento. |
| `--dont_overwrite` | Detenerse si ya existen los archivos de salida protegidos por esta opción. |

La detección de adaptadores debe contrastarse con el protocolo. Si se conocen las secuencias técnicas, pueden especificarse de forma explícita. Los cebadores de un amplicón requieren un tratamiento orientado a ese diseño; Cutadapt es una alternativa apropiada cuando se utilizan secuencias y orientaciones confirmadas.

Este comando **no activa explícitamente la corrección de bases por solapamiento** ni una ventana de recorte de calidad. Deben distinguirse capacidad del programa, comportamiento predeterminado y opciones solicitadas.

### 7.2 Recorte de extremos por calidad: decisión adicional

Si el diagnóstico justifica una poda del extremo final, fastp permite añadir opciones como:

```text
--cut_tail --cut_window_size 4 --cut_mean_quality 20
```

Estas opciones forman parte del comando de fastp, no se ejecutan como un programa independiente. Aplicarlas en una ejecución separada, con otros nombres de salida, para comparar la retención. Verificar que la reducción de longitud no perjudique el solapamiento, mapeo o ensamblaje necesarios.

No activar automáticamente filtros de baja complejidad, eliminación de duplicados o recorte de homopolímeros para todos los experimentos. La relevancia de esos patrones depende de la plataforma y del objetivo biológico.

### 7.3 Evaluar las salidas

```bash
gzip -t 02_clean/illumina/S01_R1.clean.fastq.gz 02_clean/illumina/S01_R2.clean.fastq.gz

fastqc --threads 2 --outdir 01_qc/illumina/clean \
    02_clean/illumina/S01_R1.clean.fastq.gz 02_clean/illumina/S01_R2.clean.fastq.gz

seqkit stats -a -T \
    00_raw/illumina/S01_R1.fastq.gz 00_raw/illumina/S01_R2.fastq.gz \
    02_clean/illumina/S01_R1.clean.fastq.gz 02_clean/illumina/S01_R2.clean.fastq.gz \
    > metadatos/S01_comparacion_raw_clean.tsv
```

Revisar los informes iniciales y finales y conservar el JSON de fastp. Informar los pares retenidos y las lecturas sin pareja por separado. No volver a añadir las lecturas unpaired a las salidas paired-end; su aprovechamiento depende de si el análisis posterior admite lecturas single-end adicionales.

<a id="multiqc"></a>
## 8. MultiQC: comparación entre muestras y etapas

MultiQC busca resultados de herramientas compatibles y construye un informe conjunto. Para integrar el QC de Illumina:

```bash
multiqc 01_qc/illumina \
    --outdir 03_multiqc/illumina --filename multiqc_illumina.html
```

Abrir el HTML y comprobar:

- Qué archivos y muestras fueron detectados.
- Si las etapas raw y clean tienen identificadores distintos.
- Qué métricas pertenecen a FastQC y cuáles a fastp.
- Si los resultados de alguna herramienta quedaron fuera del informe.

El archivo raw se llama `S01_R1.fastq.gz` y el procesado `S01_R1.clean.fastq.gz` para distinguir los resultados de sus análisis. En proyectos con nombres repetidos, utilizar nombres de muestra/etapa inequívocos o las opciones de identificación de directorios documentadas por MultiQC.

MultiQC admite informes de FastQC y fastp mediante módulos específicos. También puede interpretar estadísticas compatibles de NanoPack, incluidas las generadas por NanoPlot, mediante su módulo NanoStat. Esto **no significa que todos los HTML de NanoPlot o todos los JSON de cualquier herramienta se integren automáticamente**. La compatibilidad depende del formato del informe y de la versión de MultiQC.

Conservar los informes originales, el registro de MultiQC y los datos que acompañan al reporte. Si falta una muestra, investigar el reconocimiento de sus archivos antes de concluir que no presenta problemas.

<a id="longreads"></a>
## 9. Ruta práctica para lecturas largas

### 9.1 Diagnóstico inicial con NanoPlot y SeqKit

```bash
conda activate cg_qc_long
NanoPlot --version > metadatos/nanoplot_version.txt
chopper --version > metadatos/chopper_version.txt
conda list --explicit > metadatos/cg_qc_long_explicito.txt
conda env export --from-history > metadatos/cg_qc_long_historial.yml

gzip -t 00_raw/longreads/ONT01.fastq.gz
seqkit stats -a -T 00_raw/longreads/ONT01.fastq.gz \
    > metadatos/ONT01_raw_seqkit.tsv

NanoPlot --fastq 00_raw/longreads/ONT01.fastq.gz \
    --threads 4 --loglength --prefix ONT01_raw_ \
    --outdir 01_qc/longreads/ONT01_raw
```

NanoPlot produce gráficas, un informe HTML y un archivo de estadísticas. Examinar:

- Número de lecturas y bases totales.
- Mediana, media y distribución de longitudes.
- N50 de las lecturas, claramente distinguido del N50 de un ensamblaje.
- Distribución de calidad por lectura y relación entre longitud y calidad.
- Rendimiento temporal y otros patrones de corrida solo cuando la entrada contiene la información necesaria.

Las opciones de NanoPlot para ocultar lecturas, reducir datos de la gráfica o imponer un umbral durante el análisis afectan lo que se resume o visualiza. No constituyen por sí solas un procedimiento para exportar un FASTQ filtrado. Documentar cualquier selección aplicada al diagnóstico.

### 9.2 Filtrado sencillo con Chopper

Ejemplo **ilustrativo para ONT**: conservar lecturas de al menos 1,000 nt y calidad media definida por Chopper de al menos Q12. Antes de aplicarlo, comparar ese criterio con la distribución observada, la química y el modelo de asignación de bases.

```bash
set -o pipefail
gzip -dc 00_raw/longreads/ONT01.fastq.gz | \
    chopper --quality 12 --minlength 1000 --threads 4 \
    2> logs/ONT01_chopper.log | \
    gzip -c > 02_clean/longreads/ONT01.chopper.fastq.gz
```

El comando mostrado filtra lecturas completas según calidad y longitud; no solicita una estrategia de recorte de extremos ni detección de adaptadores. Chopper ofrece otras opciones que requieren una decisión adicional. No se establece aquí un criterio de GC, porque podría excluir grupos o regiones biológicas de interés.

Este corte no es apropiado por definición para todos los genomas, transcritos, amplicones, plásmidos o protocolos. Una muestra puede requerir otro umbral o no necesitar ese filtrado.

### 9.3 Evaluación posterior

```bash
gzip -t 02_clean/longreads/ONT01.chopper.fastq.gz

NanoPlot --fastq 02_clean/longreads/ONT01.chopper.fastq.gz \
    --threads 4 --loglength --prefix ONT01_chopper_ \
    --outdir 01_qc/longreads/ONT01_chopper

seqkit stats -a -T \
    00_raw/longreads/ONT01.fastq.gz 02_clean/longreads/ONT01.chopper.fastq.gz \
    > metadatos/ONT01_comparacion_raw_chopper.tsv

multiqc 01_qc/longreads \
    --outdir 03_multiqc/longreads --filename multiqc_longreads.html
```

Comprobar qué estadísticas detectó MultiQC; conservar y abrir también el HTML de NanoPlot. Si la selección no deja lecturas útiles, debe revisarse el criterio y la entrada antes de continuar con el análisis.

### 9.4 Alternativa: fastplong

fastplong es una herramienta distinta de fastp, orientada a lecturas largas. Puede producir informes HTML y JSON y realizar operaciones que cambian las secuencias o el número de registros. Revisar las opciones de la versión instalada:

```bash
fastplong --help > metadatos/fastplong_help.txt
```

La siguiente alternativa conserva desactivadas la retirada automática de adaptadores y la partición por adaptadores internos, y solicita filtrado por calidad y longitud. Requiere que la versión utilizada incluya las opciones indicadas en la documentación consultada.

```bash
fastplong \
    -i 00_raw/longreads/ONT01.fastq.gz \
    -o 02_clean/longreads/ONT01.fastplong.fastq.gz \
    --disable_adapter_trimming --disable_chimera_splitting \
    --mean_qual 12 --unqualified_percent_limit 100 \
    --n_percent_limit 10 --length_required 1000 \
    --html 01_qc/longreads/ONT01.fastplong.html \
    --json 01_qc/longreads/ONT01.fastplong.json \
    --thread 4 --dont_overwrite
```

El límite de bases no calificadas se fija en 100% para no imponer además un filtro por esa proporción; permanecen los criterios de calidad media, N y longitud mostrados. No asumir que su definición de calidad media es idéntica a la de otra herramienta.

Si se necesita retirar adaptadores, confirmar sus secuencias y el procesamiento previo. La documentación actual de fastplong describe división de lecturas con adaptadores internos por defecto; esto puede aumentar el número de registros aunque se reduzcan las bases totales. Toda partición debe declararse y conservar su trazabilidad. Si una opción de control no existe en la instalación, no eliminarla del comando sin revisar el comportamiento que se está habilitando.

### 9.5 Alternativa: Filtlong

Filtlong permite seleccionar un subconjunto por longitud y calidad. Ejemplo para comparar con los datos originales, **sin ejecutarlo después de Chopper o fastplong como paso obligatorio**:

```bash
set -o pipefail
filtlong --min_length 1000 --keep_percent 90 \
    00_raw/longreads/ONT01.fastq.gz \
    2> logs/ONT01_filtlong.log | \
    gzip -c > 02_clean/longreads/ONT01.filtlong.fastq.gz
```

`--keep_percent 90` selecciona según **bases**, no significa conservar exactamente 90% del número de lecturas. La opción `--target_bases` permite limitar las bases del subconjunto, pero debe relacionarse con cobertura y diseño. Su uso tampoco mejora automáticamente la representación de todas las poblaciones de una mezcla.

En la documentación consultada, `--min_mean_q` de Filtlong se expresa como identidad estimada porcentual, no como un umbral Phred intercambiable con `--quality` de Chopper. Si se proporciona referencia externa, también cambia la evidencia empleada para puntuar las lecturas.

No encadenar todas las herramientas por el solo hecho de que estén disponibles. Se comparan alternativas mediante retención de datos y utilidad para el análisis posterior.

<a id="hifi"></a>
## 10. Particularidades de PacBio HiFi y del procesamiento previo de ONT

### 10.1 PacBio HiFi

Identificar el producto recibido: lecturas HiFi, subreads u otro archivo de la plataforma. Para HiFi, revisar los criterios de consenso y el procesamiento comunicado por el proveedor. No adoptar directamente un esquema pensado para lecturas ONT de otra química o para subreads de mayor error.

Con FASTQ, SeqKit y herramientas de estadísticas/longitudes pueden utilizarse para diagnosticar el conjunto. Cuando se dispone de BAM/CRAM, deben revisarse también los metadatos y las métricas correspondientes; Cramino es una opción especializada en esos formatos. No convertir archivos únicamente para obtener una gráfica sin considerar qué información de metadatos podría perderse.

La distribución de longitudes y el N50 pueden depender de la selección de tamaño de biblioteca. Eliminar todas las lecturas más cortas o imponer un máximo arbitrario puede modificar la composición de los datos. El tratamiento de adaptadores debe relacionarse con el producto y la preparación reales.

### 10.2 Oxford Nanopore

Registrar química, modelo de basecalling, versión de Dorado u otro software utilizado y si se entregaron lecturas simplex, duplex o un producto procesado. Revisar si ya se efectuaron separación de muestras, retirada de barcodes y recorte de adaptadores.

La asignación de bases desde la señal y el procesamiento de FASTQ son etapas diferentes. Si solo se dispone del FASTQ, no se está repitiendo el basecalling desde la señal original. La interpretación de sus Q-scores también depende del modelo utilizado.

### 10.3 N50 de lecturas y pérdida de datos

El N50 de lecturas es la longitud que se alcanza al acumular al menos la mitad de las bases, ordenando las lecturas de mayor a menor longitud. Por ejemplo, para longitudes de **4,000, 3,000, 2,000 y 1,000 nt**, el total es 10,000 nt y el N50 es **3,000 nt**.

El N50 puede subir al descartar lecturas cortas sin que se haya producido una sola base de mejor calidad. Debe informarse junto con bases retenidas, número de lecturas, calidad y efecto sobre el objetivo biológico. No demuestra por sí mismo un ensamblaje completo o correcto.

<a id="entregables"></a>
## 11. Interpretación integrada y entregables

### 11.1 Tabla mínima de comparación

Completar una fila por muestra y etapa; diferenciar pares, lecturas individuales y fragmentos derivados de partición.

| Muestra/etapa | Plataforma | Lecturas o pares | Bases totales | Longitud mediana | Calidad o Q30, según corresponda | N50 de lecturas, si procede | Adaptadores/problemas | Decisión |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Muestra / original | Registrar | Registrar unidad | Registrar | Registrar | Definir métrica y programa | Registrar | Describir | Justificar |
| Muestra / procesada | Registrar | Registrar unidad | Registrar | Registrar | Misma métrica comparable | Registrar | Describir | Continuar o revisar |

Calcular por separado:

- **Retención de lecturas = 100 × lecturas finales / lecturas iniciales**, solo cuando sean unidades comparables y no haya partición que cambie su identidad.
- **Retención de bases = 100 × bases finales / bases iniciales**.
- Pares retenidos y lecturas sin pareja, cuando corresponda.

**Ejemplo artificial:** un millón de pares PE150 representa 300 millones de bases. Si quedan 900,000 pares con 140 nt por extremo, se retienen 252 millones de bases: 90% de los pares y 84% de las bases. Para un genoma aislado de 5 Mb, la cobertura teórica pasa de 60× a 50.4×. El ejemplo no es un resultado de las imágenes ni de una muestra real del curso.

### 11.2 Situaciones para discutir

| Hallazgo | Qué conviene comprobar antes de decidir |
| --- | --- |
| R2 pierde más calidad que R1. | Posición del deterioro, biblioteca, longitudes útiles y efecto sobre los pares. |
| Aparecen motivos técnicos después del recorte. | Adaptadores/cebadores correctos, orientación y parámetros. |
| Hay GC multimodal. | Diseño, mezcla esperada, posibles contaminantes y evidencia independiente. |
| La duplicación aparente es elevada. | Tipo de biblioteca, abundancia biológica, profundidad, PCR y UMI. |
| Sube la calidad pero se pierde gran parte de las bases. | Cobertura retenida y utilidad para el análisis. |
| Aumenta el número de lecturas largas procesadas. | Partición de lecturas, unidades del informe y trazabilidad. |
| MultiQC no muestra una muestra. | Reconocimiento de archivos, nombres y compatibilidad del módulo. |

### 11.3 Actividades y archivos de entrega

1. Explicar la diferencia entre FastQC, fastp y MultiQC.
2. Interpretar la figura 3 y proponer una decisión que considere el análisis posterior.
3. Comparar las figuras 6/7 con la figura 10: ejes, unidad observada y pregunta que responde cada diagnóstico.
4. Comparar el conjunto Illumina original con una versión procesada y justificar los parámetros.
5. Interpretar un conjunto de lecturas largas antes/después de **una** estrategia de procesamiento.
6. Explicar por qué un mayor N50 de lecturas o indicadores verdes no prueban por sí solos mejor calidad biológica.

Entregar informe breve en Markdown o PDF, tabla de métricas, comandos o script, versiones de programas y bases cuando correspondan, archivos de entorno e informes originales de las herramientas. Indicar qué análisis se ejecutaron y cuáles se interpretaron a partir de resultados proporcionados.

<a id="recursos"></a>
## 12. Páginas, repositorios y documentación verificados

Los recursos siguientes estaban disponibles al consultarlos el **19 de octubre de 2026**. Se revisaron repositorios y documentación primaria; la disponibilidad de una dirección no garantiza instalación, compatibilidad de paquetes o ejecución correcta con cualquier conjunto de datos.

| Herramienta/recurso | Página o documentación | Repositorio | Observación |
| --- | --- | --- | --- |
| FastQC | [Sitio oficial](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) y [ayuda](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/Help/) | [s-andrews/FastQC](https://github.com/s-andrews/FastQC) | Diagnóstico por archivo; interpretar cada módulo en contexto. |
| fastp | [Documentación en README](https://github.com/OpenGene/fastp#readme) | [OpenGene/fastp](https://github.com/OpenGene/fastp) | Orientado a lecturas cortas. |
| MultiQC | [Documentación](https://docs.seqera.io/multiqc/) | [MultiQC/MultiQC](https://github.com/MultiQC/MultiQC) | Integración de resultados compatibles. |
| SeqKit | [Manual](https://bioinf.shenwei.me/seqkit/usage/) | [shenwei356/seqkit](https://github.com/shenwei356/seqkit) | Estadísticas y manejo de FASTA/FASTQ. |
| Cutadapt | [Documentación](https://cutadapt.readthedocs.io/en/stable/) | [marcelm/cutadapt](https://github.com/marcelm/cutadapt) | Adaptadores y cebadores conocidos. |
| NanoPlot | [Documentación en README](https://github.com/wdecoster/NanoPlot#readme) | [wdecoster/NanoPlot](https://github.com/wdecoster/NanoPlot) | Gráficas, HTML y estadísticas de long reads. |
| fastplong | [Documentación en README](https://github.com/OpenGene/fastplong#readme) | [OpenGene/fastplong](https://github.com/OpenGene/fastplong) | Revisar operaciones activadas y opciones disponibles. |
| Chopper | [Documentación en README](https://github.com/wdecoster/chopper#readme) | [wdecoster/chopper](https://github.com/wdecoster/chopper) | Filtrado y estrategias de recorte para lecturas largas. |
| Filtlong | [Documentación en README](https://github.com/rrwick/Filtlong#readme) | [rrwick/Filtlong](https://github.com/rrwick/Filtlong) | Selección por longitud/calidad; porcentajes expresados en bases. |
| Dorado | [Documentación oficial](https://software-docs.nanoporetech.com/dorado/latest/) | [nanoporetech/dorado](https://github.com/nanoporetech/dorado) | Basecalling y procesamiento asociado de ONT. |
| Cramino | [Documentación en README](https://github.com/wdecoster/cramino#readme) | [wdecoster/cramino](https://github.com/wdecoster/cramino) | Evaluación de BAM/CRAM de lecturas largas. |
| NanoStat | [Documentación en README](https://github.com/wdecoster/nanostat#readme) | [wdecoster/nanostat](https://github.com/wdecoster/nanostat) | Disponible; su autor señala sustitución por Cramino y mantenimiento limitado. |
| PacBio HiFi | [Información oficial de HiFi](https://www.pacb.com/technology/hifi-sequencing/) | No aplica | Identificar el producto y procesamiento entregados. |

Documentación de integración en MultiQC: [módulo fastp](https://docs.seqera.io/multiqc/modules/fastp) y [módulo NanoStat/NanoPack](https://docs.seqera.io/multiqc/modules/nanostat).

Las imágenes enlazadas pertenecen al material original de esta unidad y conservan sus archivos: `figura_1.png` a `figura_11.png`, más `Kmer.png`. Se incorporan mediante direcciones completas del repositorio para que puedan mostrarse sin mover los archivos de imagen al actualizar este Markdown.

**Criterio transversal:** todo procesamiento debe responder a un problema identificado y sustentarse con métricas comparables antes/después y una justificación biológica.
