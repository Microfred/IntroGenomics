# fastp: Un preprocesador de datos FASTQ ultrarrápido y todo en uno

[fastp](https://github.com/OpenGene/fastp) es una herramienta diseñada para realizar el control de calidad, el filtrado y el recorte de adaptadores en archivos de secuenciación FastQ. Está desarrollado en C++ y soporta multihilo para ofrecer un rendimiento extremadamente alto.

## 🚀 Características Principales
* **Filtrado de calidad:** Elimina lecturas de baja calidad o con demasiadas bases N.
* **Recorte de adaptadores:** Detección y recorte automático de adaptadores (tanto para *single-end* como *paired-end*).
* **Corrección de bases:** Capacidad de corregir bases erróneas en datos *paired-end* basándose en el solapamiento.
* **Reportes visuales:** Genera informes detallados en formato HTML y JSON listos para inspección.
* **Alta velocidad:** Significativamente más rápido que herramientas tradicionales como Trimmomatic o FastQC.

## 🛠️ Instalación y Uso Básico

Puedes instalarlo fácilmente a través de **Conda**:
`
conda install -c bioconda fastp`


Ejemplo de comando (Paired-End):

`fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz --html report.html --json report.json`

📊 Salida de Datos
Al finalizar, fastp genera dos archivos clave:
`report.html:` Un informe interactivo que puedes abrir en cualquier navegador para visualizar la calidad antes y después del filtrado.
`report.json:` Ideal para procesamiento automático o para integrar con herramientas como MultiQC.

**Nota**: fastp es capaz de detectar automáticamente la mayoría de los adaptadores de Illumina, por lo que no suele ser necesario proporcionar una lista de secuencias manualmente.

---

## 📈 Integración con MultiQC

Si estás procesando múltiples muestras, puedes consolidar todos los reportes de **fastp** utilizando [MultiQC](https://multiqc.info/). Esto permite comparar la calidad de todas tus bibliotecas en una sola vista.

## 🛠️ Instalación de MultiQC

Para consolidar los reportes generados por **fastp**, puedes instalar **MultiQC** utilizando diferentes gestores de paquetes. Se recomienda el uso de **Conda/Mamba** para gestionar las dependencias de bioinformática:

### Opción 1: Vía Conda (Recomendado)
`conda install -c bioconda -c conda-forge multiqc`

###Opción 2: Vía Pip (Python)
`pip install multiqc`


### Pasos para integrar:

1. **Ejecuta fastp** para todas tus muestras, asegurándote de que los archivos `.json` tengan nombres únicos o estén en carpetas separadas:
   `fastp -i muestra1_R1.fq.gz -I muestra1_R2.fq.gz -o out1_R1.fq.gz -O out1_R2.fq.gz -j muestra1.json`
---

# 🧬 Flujo de Trabajo Sugerido (Metodología)

El siguiente flujo describe el procesamiento de calidad desde los datos crudos hasta el reporte final consolidado.

### 1. Control de Calidad y Filtrado Inicial
Se recomienda ejecutar **fastp** para limpiar las lecturas, eliminar adaptadores y corregir bases por solapamiento (en datos paired-end).

Ejemplo para una sola muestra
`fastp \
  -i raw_data/sample1_R1.fastq.gz \
  -I raw_data/sample1_R2.fastq.gz \
  -o cleaned_data/sample1_trimmed_R1.fastq.gz \
  -O cleaned_data/sample1_trimmed_R2.fastq.gz \
  -j reports/sample1.json \
  -h reports/sample1.html \
  --detect_adapter_for_pe \
  --thread 4
`
### 2. Procesamiento por Lotes (Opcional)
Si tienes múltiples archivos en una carpeta, puedes automatizar el proceso con un ciclo simple en Bash:

Ejemplo para un conjunto de muestras

```
for r1 in 00_raw/*_R1.fastq.gz; do
    r2=${r1/_R1.fastq.gz/_R2.fastq.gz}
    base=$(basename $r1 _R1.fastq.gz)
   
    fastp -i $r1 -I $r2 \
          -o 01_qc/clean${base}_R1.fq.gz -O 01_qc/clean${base}_R2.fq.gz \
          -j reports/${base}.json -h reports/${base}.html
done
```

### 3. Integración de Resultados
Una vez generados los archivos .json de todas las muestras, se utiliza MultiQC para crear un único reporte interactivo que resuma la calidad de todo el proyecto.
`multiqc reports/ -o final_report/`


---

## 👨‍🏫 Contacto y Afiliación

Este repositorio es mantenido como parte de las actividades académicas y de investigación en el área de **Ciencias Genómicas**.

**Dr. Juan Alfredo Hernàndez García** *Titular de la materia de Ciencias Genómicas*
**Escuela Nacional de Ciencias Biológicas (ENCB)**
**Instituto Politécnico Nacional (IPN)** 
Miembro del Sistema Nacional de Investigadores (**SNI**)

---
<p align="center">
  <img src="https://raw.githubusercontent.com/Ileriayo/markdown-badges/master/bld/logos/python.svg" alt="Python" height="25">
  <img src="https://img.shields.io/badge/Bioinformatics-Evolution-blue" alt="Bioinformatics">
  <img src="https://img.shields.io/badge/Institution-IPN-maroon" alt="IPN">
</p>
