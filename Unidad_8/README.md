# Predicción y Anotación Funcional

Una vez obtenido el ensamble de calidad (evaluado con QUAST), el siguiente paso es identificar las regiones codificantes y asignarles una función biológica.

## Complejidad y anotación de genomas.

### Evaluación de Completitud con [BUSCO6](https://busco.ezlab.org/busco_userguide.html)

Antes de anotar, usamos BUSCO para medir qué tan completo es nuestro ensamble basándonos en genes ortólogos de copia única.

Instalación
`conda create -n busco6 -c bioconda -c conda-forge -c bioconda busco=6.0.0 sepp=4.5.5`

Si la instalación tarda mucho, recuerda que puedes ejecutar conda:

`mamba install -c conda-forge -c bioconda busco=6.0.0 sepp=4.5.5`

Antes de ejecutar el análisis, es muy importante que verifiques la DB:
`busco --list-datasets`

Ejecución (usando el linaje de hongos como ejemplo)

`busco -i assembly_output/contigs.fasta -l fungi_odb10 -o busco_output -m genome`

---

## 🧬 1. Anotación de Procariontes (Bacterias) con Prokka

[Prokka](https://github.com/tseemann/prokka) es una herramienta "todo en uno" que coordina aplicaciones como Prodigal (predicción de genes), RNAmmer (rRNA) y Blast/HMMER (anotación funcional).

### Instalación
```
conda create -n prokka -c bioconda -c conda-forge prokka -y
conda activate prokka

#ejecución básica:

prokka --outdir annotation_results --prefix muestra_01 --kingdom Bacteria assembly_output/contigs.fasta

```
Los flags para las opciones de PROKKA son los siguientes

`--outdir`: Carpeta de salida.

`--prefix`: Nombre base para los archivos generados (.gff, .gbk, .faa, etc.).

`--kingdom`: Especifica si es Bacteria o Archaea.

`--cpus`: Número de hilos para el análisis.

---
