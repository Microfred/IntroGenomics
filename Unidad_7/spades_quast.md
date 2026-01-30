# Ensamblaje de Genomas con SPAdes

Este flujo de trabajo describe el proceso de ensamblaje *de novo* y la evaluación de calidad de los contigs resultantes.

## 🛠️ Configuración del Entorno

Para garantizar la reproducibilidad y evitar conflictos de dependencias, creamos un entorno específico de Conda para el ensamblaje:

```
# Crear el entorno con SPAdes y QUAST
conda create -n ensamblaje -c bioconda -c conda-forge spades quast -y

# Activar el entorno
conda activate ensamblaje
```
---

🚀 Uso de SPAdes (Ensamblaje)
spades.py es un ensamblador versátil que funciona excelente tanto para genomas pequeños (bacterias), así como genomas de levaduras y hongos, incluyendo datos metagenómicos.

Ejemplo de comando (Paired-End):

```
spades.py \
  -1 0/sample1_R1.fq.gz \
  -2 cleaned_data/sample1_R2.fq.gz \
  -o assembly_output/ \
  --threads 8 \
  --memory 32 \
  --careful
Nota: La opción --careful se recomienda para bacterias para reducir el número de errores en los contigs.
```

---
Entender qué hace cada "bandera" o flag 🚩 es fundamental para ajustar el ensamble según el tipo de genoma y la capacidad de tu servidor. 

| Opción | Descripción | Recomendación / Nota | Ejemplo de uso |
|------|------------|---------------------|----------------|
| `-1 / -2` | Archivos de lecturas forward y reverse (Paired-end) | Usa siempre archivos limpios (post-fastp). | `-1 sample_R1.fastq.gz -2 sample_R2.fastq.gz` |
| `-o` | Directorio de salida | SPAdes creará la carpeta si no existe. | `-o spades_output` |
| `--threads` (`-t`) | Número de núcleos de CPU a utilizar | Verifica recursos disponibles con `nproc`. | `--threads 16` |
| `--memory` (`-m`) | Límite de RAM en GB | Para bacterias, 32–64 GB suelen bastar. | `--memory 64` |
| `--careful` | Reduce mismatches y errores de indels | Indispensable para genomas pequeños. | `--careful` |
| `--cov-cutoff` | Elimina contigs con baja cobertura | `auto` ayuda a limpiar contaminación de bajo nivel. | `--cov-cutoff auto` |
| `-k` | Tamaños de K-meros (ej. `21,33,55,77`) | SPAdes puede elegirlos automáticamente. | `-k 21,33,55,77` |
| `--sc` | Modo Single-cell | Solo para datos MDA (baja biomasa). | `--sc` |

---
Antes de realizar el anális de calidad del ensamblado, es importante filtrar los contigs o scaffolds de acuerdo a la longitud de estos, depentiendo el número de scaffolds, puedes elegir 500 ó 1000:

`seqtk seq -A scaffolds.fasta | awk '{if(NR%2==0) {if(length($0)>=1000) print header"\n"$0} else header=$0}' > Scaffolds_1000.fasta´ 
---

📊 Evaluación con **QUAST**
Una vez generado el archivo contigs.fasta, es vital evaluar su calidad (N50, número de contigs, longitud total).


`quast.py \
  spades_output/contigs.fasta \
  -r reference.fasta \
  -g annotation.gff \
  --threads 8 \
  -o quast_results
`

Entender qué hace cada "bandera" o flag 🚩 es fundamental para ajustar el ensamble según el tipo de genoma y la capacidad de tu servidor. 

| Flag | Significado | Notas Técnicas | Ejemplo de uso |
|------|------------|----------------|----------------|
| `-o` | Directorio de salida del reporte | `report.html` es el archivo más informativo. | `-o quast_results` |
| `-r` | Genoma de referencia (opcional) | Permite calcular errores de alineación y cobertura. | `-r reference.fasta` |
| `-g` | Archivo de anotación (`.gff` o `.bed`) | Evalúa genes completos y fragmentados. | `-g annotation.gff` |
| `--min-contig` | Longitud mínima del contig | Por defecto 500 bp; filtra ruido corto. | `--min-contig 1000` |
| `--threads` | Número de hilos | 4–8 hilos suelen ser suficientes. | `--threads 8` |


---
📚 Citación
SPAdes: Bankevich A., et al. (2012). SPAdes: A New Genome Assembly Algorithm and Its Applications to Single-Cell Sequencing. Journal of Computational Biology. doi:10.1089/cmb.2012.0021

QUAST: Gurevich A., et al. (2013). QUAST: quality assessment tool for genome assemblies. Bioinformatics. doi:10.1093/bioinformatics/btt086


