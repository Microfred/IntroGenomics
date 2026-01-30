# Predicción y Anotación Funcional

Una vez obtenido el ensamble de calidad (evaluado con QUAST), el siguiente paso es identificar las regiones codificantes y asignarles una función biológica.


#### Evaluación de Completitud con [BUSCO6](https://busco.ezlab.org/busco_userguide.html)

Antes de anotar, usamos BUSCO para medir qué tan completo es nuestro ensamble basándonos en genes ortólogos de copia única.

Instalación:

`conda create -n busco6 -c bioconda -c conda-forge -c bioconda busco=6.0.0 sepp=4.5.5`

Si la instalación tarda mucho, recuerda que puedes ejecutar conda:

`mamba install -c conda-forge -c bioconda busco=6.0.0 sepp=4.5.5`

Antes de ejecutar el análisis, es muy importante que verifiques la DB:
`busco --list-datasets`

Ejecución (usando el linaje de hongos como ejemplo)

`busco -i assembly_output/contigs.fasta -l fungi_odb10 -o busco_output -m genome`

---
### Interpretación de resultados de BUSCO:

## 📊 Interpretación de Resultados de BUSCO v6

**BUSCO (Benchmarking Universal Single-Copy Orthologs)** evalúa la completitud del genoma buscando genes que deberían estar presentes en una sola copia en el linaje seleccionado (ej. `fungi_odb10` o `bacteria_odb10`).

### 📝 El "Código de Colores" de BUSCO

Al finalizar el análisis, obtendrás un gráfico de barras (usualmente generado con el script `generate_plot.py`) que utiliza la siguiente nomenclatura:

* **Complete (C) [Azul]:** Genes encontrados íntegros y con la longitud esperada.
    * *Single-copy (S):* Una sola copia encontrada (Ideal).
    * *Duplicated (D):* Dos o más copias (Podría indicar un ensamble mal colapsado o una poliploidía).
* **Fragmented (F) [Amarillo]:** Solo se encontró una parte del gen. Indica que el ensamble está roto en zonas codificantes.
* **Missing (M) [Rojo]:** No se encontró el gen. Puede deberse a una baja cobertura en la secuenciación o a que el organismo es muy divergente.

---

### 📉 ¿Qué valores son aceptables para el Taller?

Para un genoma de alta calidad destinado a publicación, busca los siguientes umbrales:

| Calidad | % Complete (S) | Acción Sugerida |
| :--- | :--- | :--- |
| **Excelente** | > 95% | Proceder a anotación funcional con confianza. |
| **Aceptable** | 80% - 94% | Revisar si hay mucha fragmentación en QUAST. |
| **Pobre** | < 80% | Considerar re-ensamblar o verificar contaminación. |

---

## 🛠️ Generación de Gráficos para el Reporte

Para que tus alumnos incluyan resultados visuales en sus presentaciones, diles que ejecuten este comando dentro del entorno de BUSCO:


# Dentro de la carpeta de resultados de BUSCO
`generate_plot.py -wd busco_output/`

---

# 🧬 1. Anotación de Procariontes (Bacterias) con Prokka
## Complejidad y anotación de genomas.

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
# Anotación con [Funannotate](https://funannotate.readthedocs.io/en/latest/index.html)

Funannotate es un pipeline diseñado específicamente para hongos que simplifica el entrenamiento de predictores y la limpieza de modelos de genes.

Paso 1: Limpieza de contigs (eliminar duplicados pequeños):

`funannotate clean -i assembly.fasta -o assembly_cleaned.fasta`

Paso 2: Sort (ordenar por longitud):

`funannotate sort -i assembly_cleaned.fasta -o assembly_sorted.fasta`

Paso 3: Predicción de genes: 

`funannotate predict -i assembly_sorted.fasta -o funnel_out -s "Especie name" --cpus 8`

*:Nota: Requiere especificar el linaje para los modelos de Augustus*
---

## En resumen:

| Característica | Prokka | Funannotate |
|---------------|--------|-------------|
| Organismo target | Bacterias / Arqueas | Hongos / Levaduras |
| Complejidad | Baja (rápido) | Alta (requiere entrenamiento) |
| Soporte de intrones | No | Sí (esencial para hongos) |
| Archivos de salida | `.gff`, `.gbk`, `.faa`, `.ffn` | `.gff3`, `.gbk`, `.tbl`, `.sqn` |


