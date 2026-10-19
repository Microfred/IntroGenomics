# Flujo genómico reproducible

**Curso:** Ciencias Genómicas, ENCB-IPN, semestre 27-1.  
**Profesor:** Dr. Juan Alfredo Hernández García.  
**Regresar a:** [página principal](README.md).

## Alcance y decisiones iniciales

La ruta ejecutable de esta guía corresponde a **lecturas Illumina paired-end de un aislamiento bacteriano**. Los nombres `muestra01_R1.fastq.gz` y `muestra01_R2.fastq.gz` son nombres de ejemplo que deben adaptarse al conjunto de datos acordado. La última sección describe las decisiones adicionales para hongos y levaduras.

El procedimiento supone lecturas demultiplexadas y correctamente pareadas. No se aplica automáticamente a amplicones, RNA-seq, metagenomas, datos single-cell o lecturas largas. Para esos casos deben seleccionarse métodos y parámetros específicos.

Los comandos bioinformáticos se revisaron contra documentación oficial; su instalación y ejecución con el conjunto de datos de cada proyecto deben comprobarse antes de la práctica.

| Etapa | Pregunta principal | Evidencia |
| --- | --- | --- |
| Datos | ¿Qué organismo y experimento representan? | Accesiones y metadatos. |
| Lecturas | ¿Qué problemas requieren procesamiento? | FastQC, fastp y MultiQC. |
| Ensamblaje | ¿Qué secuencias pudieron reconstruirse? | FASTA, registro y parámetros. |
| Evaluación | ¿Qué limitaciones presenta el ensamblaje? | QUAST, mapeo, cobertura y BUSCO. |
| Anotación | ¿Qué genes y funciones se predicen? | Anotaciones y evidencia funcional. |
| Interpretación | ¿Qué sostiene la pregunta biológica? | Tablas, figuras, comparación y discusión. |

## 1. Organización y procedencia

Desde una carpeta nueva para el proyecto, iniciar Bash y crear:

```bash
bash
set -o pipefail
mkdir -p 00_raw 01_qc 02_ensamblaje 03_evaluacion 04_anotacion 05_comparativa 06_informe logs metadatos
```

Los comandos posteriores se ejecutan desde esta raíz. Mantener los archivos originales separados de los resultados. Registrar en `metadatos`:

- Identificador de muestra, organismo declarado, accesiones y procedencia.
- Tecnología, tipo de biblioteca, read layout y correspondencia R1/R2.
- Fecha de descarga y, cuando estén disponibles, sumas de comprobación del proveedor/repositorio.
- Versión de cada programa, base de datos, parámetros y recursos empleados.

Si hay varios lanes, se mantendrá la trazabilidad de su correspondencia. La unión de archivos se justificará por muestra, biblioteca y dirección de lectura; no se decidirá únicamente por nombres parecidos.

## 2. Entornos de trabajo

Instalar un gestor Conda compatible con el equipo siguiendo la documentación de [Miniforge](https://github.com/conda-forge/miniforge), o utilizar la instalación institucional preparada. Los entornos de Conda separan paquetes y dependencias; no equivalen a contenedores.

```bash
conda create -n cg_qc --override-channels --strict-channel-priority \
    -c conda-forge -c bioconda fastqc fastp multiqc

conda create -n cg_ensamblaje --override-channels --strict-channel-priority \
    -c conda-forge -c bioconda spades quast bwa samtools

conda create -n cg_busco --override-channels --strict-channel-priority \
    -c conda-forge -c bioconda busco
```

Estos comandos solicitan paquetes al resolver el entorno, por lo que no fijan por sí solos una versión reproducible entre fechas y plataformas. Después de instalar y comprobar cada entorno, conservar su resolución exacta y las especificaciones de alto nivel. Ejemplo para el primero:

```bash
conda activate cg_qc
conda list --explicit > metadatos/cg_qc_explicito.txt
conda env export --from-history > metadatos/cg_qc_historial.yml
```

Repetir el registro para los demás entornos. El archivo explícito corresponde a una plataforma concreta y no garantiza una reconstrucción idéntica en otra arquitectura. No reinstalar todo ni modificar permanentemente los canales globales para repetir una práctica.

## 3. Calidad inicial y procesamiento

Colocar o referenciar las dos lecturas correspondientes en `00_raw`. Revisar las rutas antes de continuar:

```bash
test -s 00_raw/muestra01_R1.fastq.gz
test -s 00_raw/muestra01_R2.fastq.gz
gzip -t 00_raw/muestra01_R1.fastq.gz 00_raw/muestra01_R2.fastq.gz
mkdir -p 01_qc/fastqc_inicial 01_qc/fastqc_final 01_qc/reportes

conda activate cg_qc
fastqc --threads 2 --outdir 01_qc/fastqc_inicial \
    00_raw/muestra01_R1.fastq.gz 00_raw/muestra01_R2.fastq.gz

fastp \
    --in1 00_raw/muestra01_R1.fastq.gz \
    --in2 00_raw/muestra01_R2.fastq.gz \
    --out1 01_qc/muestra01_R1.clean.fastq.gz \
    --out2 01_qc/muestra01_R2.clean.fastq.gz \
    --detect_adapter_for_pe --thread 4 \
    --html 01_qc/reportes/muestra01_fastp.html \
    --json 01_qc/reportes/muestra01_fastp.json \
    > logs/muestra01_fastp.stdout.log 2> logs/muestra01_fastp.stderr.log

fastqc --threads 2 --outdir 01_qc/fastqc_final \
    01_qc/muestra01_R1.clean.fastq.gz 01_qc/muestra01_R2.clean.fastq.gz

multiqc 01_qc --outdir 01_qc/multiqc
```

El comando de fastp utiliza sus valores predeterminados salvo las opciones mostradas. Registrar la versión y revisar esos valores antes de adoptarlos. No activa explícitamente la corrección de bases por solapamiento. Cuando se conocen los adaptadores o existen cebadores, se evaluará el procedimiento adecuado para retirarlos.

**Interpretación:** comparar número de lecturas, bases retenidas, longitudes, adaptadores y perfiles de calidad antes/después. Una advertencia de FastQC debe interpretarse según la biblioteca; no hay que recortar datos solo para obtener indicadores verdes. La integridad gzip no valida por sí sola el formato FASTQ ni el pareo.

**Entrega:** tabla antes/después y justificación del procesamiento. Los informes de varias muestras deben tener nombres o carpetas distintos.

## 4. Ensamblaje del aislamiento bacteriano

```bash
conda activate cg_ensamblaje
spades.py \
    -1 01_qc/muestra01_R1.clean.fastq.gz \
    -2 01_qc/muestra01_R2.clean.fastq.gz \
    --isolate --threads 4 --memory 8 \
    -o 02_ensamblaje/muestra01_spades \
    > logs/muestra01_spades.stdout.log 2> logs/muestra01_spades.stderr.log
```

`--isolate` corresponde a datos Illumina de aislamiento multicelular con cobertura alta. No debe combinarse con `--careful`; tampoco sustituye la selección de modos específicos para otros diseños. Los 4 hilos y 8 GB son límites ilustrativos, no una promesa de que cualquier genoma pueda ensamblarse con esos recursos.

Conservar `contigs.fasta`, `scaffolds.fasta`, el registro y el grafo de ensamblaje. Un scaffold puede contener huecos; no debe confundirse con una secuencia completamente resuelta. Utilizar un directorio nuevo al comparar parámetros y documentar cada ejecución.

## 5. Filtrado explícito y evaluación con QUAST

Para el ejemplo se conserva una copia con contigs de al menos **500 nt**. Este corte es una decisión didáctica que deberá justificarse; también puede excluir secuencias biológicamente relevantes.

El siguiente filtro maneja secuencias multilínea:

```bash
awk -v minimo=500 '
function emitir() {
    if (cabecera != "" && length(secuencia) >= minimo)
        print cabecera "\n" secuencia
}
/^>/ { emitir(); cabecera=$0; secuencia=""; next }
{ gsub(/[[:space:]]/, ""); secuencia=secuencia $0 }
END { emitir() }
' 02_ensamblaje/muestra01_spades/contigs.fasta \
    > 03_evaluacion/muestra01_contigs_ge500.fasta

test -s 03_evaluacion/muestra01_contigs_ge500.fasta
quast.py 03_evaluacion/muestra01_contigs_ge500.fasta \
    --min-contig 500 --threads 4 --output-dir 03_evaluacion/quast_muestra01
```

Los análisis posteriores de esta ruta utilizan **el mismo FASTA filtrado**. Se conserva el ensamblaje original para documentar el efecto del corte.

Revisar longitud total, GC, número de contigs, contig mayor, N50 y presencia de bases ambiguas. Un N50 alto no demuestra completitud, ausencia de contaminación ni exactitud estructural.

La ejecución sin referencia produce métricas de contigüidad. Para evaluar alineamientos o posibles errores relativos a una referencia, agregar una referencia biológicamente pertinente y verificar su accesión. Si se aportan anotaciones a QUAST, deben representar posiciones de elementos en esa referencia y tener identificadores compatibles; un FASTA `.ffn` no reemplaza esas coordenadas.

## 6. Mapeo y cobertura con un denominador definido

```bash
bwa index 03_evaluacion/muestra01_contigs_ge500.fasta

bwa mem -t 4 03_evaluacion/muestra01_contigs_ge500.fasta \
    01_qc/muestra01_R1.clean.fastq.gz \
    01_qc/muestra01_R2.clean.fastq.gz \
    2> logs/muestra01_bwa.log | \
    samtools sort -@ 2 -o 03_evaluacion/muestra01_lecturas.bam

samtools index 03_evaluacion/muestra01_lecturas.bam
samtools flagstat 03_evaluacion/muestra01_lecturas.bam \
    > 03_evaluacion/muestra01_flagstat.txt

samtools depth -aa -s -q 20 -Q 20 03_evaluacion/muestra01_lecturas.bam \
    > 03_evaluacion/muestra01_profundidad.tsv

awk '
{ posiciones++; suma+=$3; cubiertas1+=($3>=1); cubiertas5+=($3>=5) }
END {
    if (posiciones == 0) exit 1
    printf "metrica\tvalor\n"
    printf "posiciones_evaluadas\t%d\n", posiciones
    printf "profundidad_media\t%.4f\n", suma/posiciones
    printf "amplitud_ge1_pct\t%.4f\n", 100*cubiertas1/posiciones
    printf "amplitud_ge5_pct\t%.4f\n", 100*cubiertas5/posiciones
}' 03_evaluacion/muestra01_profundidad.tsv \
    > 03_evaluacion/muestra01_resumen_cobertura.tsv
```

Se incluyen posiciones con profundidad cero, también en secuencias de referencia no utilizadas (`-aa`). El denominador es la longitud de las secuencias del FASTA utilizado para mapear, no un tamaño genómico asumido. `-s` evita contar dos veces la zona de solapamiento de un par; `-q 20` y `-Q 20` son filtros de calidad de base y de mapeo, respectivamente. Deben informarse y justificarse, junto con los filtros de alineamientos de SAMtools.

La profundidad media y la amplitud responden a preguntas distintas. Tampoco deben confundirse la proporción de lecturas mapeadas y la proporción de posiciones cubiertas. Usar `>=5` cuando se informa amplitud a **al menos 5×**; `>5` corresponde a otro umbral.

## 7. Completitud con BUSCO

```bash
conda activate cg_busco
busco --list-datasets > metadatos/busco_linajes_disponibles.txt
busco -m genome \
    -i 03_evaluacion/muestra01_contigs_ge500.fasta \
    --auto-lineage-prok -c 4 \
    -o muestra01_busco --out_path 03_evaluacion
```

Este ejemplo usa selección automática para procariotas y requiere acceso a las bases necesarias o una preparación equivalente del entorno. Registrar el linaje finalmente seleccionado, la versión de la base, la versión de BUSCO y el método. Para comparaciones entre genomas debe fijarse una estrategia comparable.

| Categoría | Significado |
| --- | --- |
| C | Marcadores completos; **C = S + D**. |
| S | Marcadores completos en copia única. |
| D | Marcadores completos duplicados. |
| F | Marcadores fragmentados. |
| M | Marcadores no encontrados. |

Reportar también el número de marcadores evaluados. Los duplicados y ausentes pueden tener varias causas biológicas o técnicas. No hay un porcentaje único de BUSCO que garantice que todo genoma está listo para publicación; la interpretación requiere las demás evidencias de calidad.

## 8. Anotación bacteriana

Para nuevos análisis bacterianos se propone [Bakta](https://github.com/oschwengers/bakta). El repositorio oficial de [Prokka](https://github.com/tseemann/prokka) informa que su autor ya no lo mantiene y recomienda la transición a Bakta. Prokka se conserva en los documentos anteriores para interpretar resultados históricos.

```bash
conda create -n cg_bakta --override-channels --strict-channel-priority \
    -c conda-forge -c bioconda bakta
conda activate cg_bakta
bakta --version
```

Se necesita una base de datos compatible. Revisar su tamaño y disponibilidad antes de descargarla; en clase puede utilizarse una copia institucional preparada. El siguiente ejemplo descarga la variante ligera:

```bash
bakta_db download --output metadatos/bakta_db --type light
```

Verificar el directorio de la base descargada. Si se encuentra en la ruta siguiente:

```bash
cg_bakta_db='metadatos/bakta_db/db-light'
test -f "$cg_bakta_db/db.json"
bakta --db "$cg_bakta_db" --threads 4 \
    --output 04_anotacion/muestra01_bakta --prefix muestra01 \
    03_evaluacion/muestra01_contigs_ge500.fasta
```

Registrar versión y tipo de base. La elección de una base ligera o completa debe mantenerse documentada y ser comparable entre muestras. Bakta se utiliza aquí para bacterias; no se presenta como anotador universal de arqueas u hongos.

Revisar los archivos de anotación, proteínas y tablas. Las etiquetas funcionales son predicciones: contrastarlas con evidencia de secuencia, dominios, contexto genómico y referencias antes de atribuir una función específica.

## 9. Adaptación a hongos y levaduras

La ruta fúngica requiere revisar tamaño, ploidía, heterocigosidad, repeticiones y estrategia de secuenciación. El ensamblaje y la predicción de genes deben adaptarse al organismo y a los recursos disponibles.

- Seleccionar un linaje de BUSCO apropiado al grupo y disponible en la versión utilizada.
- Evaluar repeticiones y enmascaramiento antes de la predicción de genes, cuando corresponda.
- Utilizar una estrategia eucariota, por ejemplo [Funannotate](https://funannotate.readthedocs.io/en/latest/), con bases instaladas, modelos, entrenamiento y evidencias de proteínas o transcritos adecuados al caso.
- Distinguir predicción estructural, asignación funcional y revisión de los modelos.
- Documentar las versiones y todas las evidencias. Un comando aislado de predicción no sustituye esta preparación.

## 10. Informe y entregables

Cada proyecto debe incluir:

1. Accesiones y metadatos; versiones y parámetros.
2. Tabla antes/después del procesamiento y calidad de las lecturas.
3. Métricas del ensamblaje y efecto del corte de longitud.
4. Mapeo, denominador, filtros y resumen de cobertura.
5. BUSCO con C/S/D/F/M, número de marcadores y linaje.
6. Anotación y análisis de una ruta metabólica con evidencia y limitaciones.
7. Análisis comparativo/filogenómico con referencias seleccionadas de forma explícita.
8. Figuras, tablas, interpretación y archivos necesarios para reproducir el trabajo.

La ausencia de una predicción no demuestra por sí sola ausencia biológica. Los resultados deben leerse en conjunto y contrastarse con el diseño y las referencias.

## Documentación oficial

| Herramienta | Fuente |
| --- | --- |
| Conda/Mamba y paquetes | [Miniforge](https://github.com/conda-forge/miniforge), [Mamba](https://mamba.readthedocs.io/en/latest/), [Bioconda](https://bioconda.github.io/) |
| FastQC | [Babraham Bioinformatics](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) |
| fastp | [GitHub](https://github.com/OpenGene/fastp) |
| MultiQC | [Sitio oficial](https://seqera.io/multiqc/) |
| SPAdes | [Opciones de ejecución](https://ablab.github.io/spades/running.html) |
| QUAST | [Manual](https://quast.sourceforge.net/docs/manual.html) |
| BWA | [GitHub](https://github.com/lh3/bwa) |
| SAMtools | [depth](https://www.htslib.org/doc/samtools-depth.html), [fastq](https://www.htslib.org/doc/samtools-fasta.html) |
| BUSCO | [Guía de usuario](https://busco.ezlab.org/busco_userguide.html) |
| Bakta / Prokka | [Bakta](https://github.com/oschwengers/bakta), [Prokka](https://github.com/tseemann/prokka) |
| Funannotate | [Documentación](https://funannotate.readthedocs.io/en/latest/) |

**Actualización documental:** 19 de octubre de 2026.
