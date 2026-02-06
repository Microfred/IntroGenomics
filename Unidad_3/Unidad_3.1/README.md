# 🧬 Scaffolding Genómico con (RagTag)[https://github.com/malonge/RagTag]

Este repositorio documenta el flujo de trabajo y la metodología para realizar el **scaffolding** (andamiaje) de ensamblajes genómicos utilizando la herramienta **RagTag**.  
Este protocolo forma parte de los pipelines de análisis de **MicroFred**.

---

## 🧬 ¿Qué es el Scaffolding?

El scaffolding es una etapa crítica en el ensamblaje de genomas *de novo*.  
Consiste en ordenar y orientar los **contigs** (fragmentos de secuencia continua) para formar estructuras más largas llamadas **scaffolds**.

- **Contig:** Secuencia continua de ADN sin interrupciones (A, C, G, T).
- **Scaffold:** Conjunto de contigs unidos en un orden específico, separados por *gaps* de longitud estimada, representados por `N`.

🎯 **Objetivo:** pasar de una colección fragmentada de secuencias a una estructura que represente fielmente los cromosomas del organismo.

---

## 📋 ¿Cuándo y por qué usarlo?

### 🕒 ¿Cuándo se debe usar?
El scaffolding se realiza **después del ensamblaje inicial** (SPAdes, Flye, Canu) y **antes** de la anotación genómica o análisis filogenómicos profundos.

Es especialmente recomendable cuando:
1. El ensamblaje está altamente fragmentado.
2. Existe un genoma de referencia cercano y de buena calidad (nivel cromosoma).

---

### ✅ Ventajas
1. **Mayor continuidad (↑ N50).**
2. **Contexto genómico:** genes vecinos, operones y BGCs completos.
3. **Mejor detección de variantes estructurales.**
4. **Filtrado indirecto de contigs espurios o contaminantes.**

---

### ⚠️ Riesgos
1. **Sesgo de referencia:** orden forzado si la referencia es lejana.
2. **Uniones falsas:** scaffolds incorrectos.
3. **Limitaciones en sintenia:** sin scaffolding, comparaciones entre especies son deficientes.

---

## 🛠️ Herramienta: RagTag

**RagTag** permite corregir ensamblajes y realizar scaffolding basado en homología contra una referencia.
¿Cómo funciona?


### 🔧 Instalación (Conda / Mamba)

`mamba install -c bioconda ragtag`

---

###  💻 Flujo de Trabajo

1️⃣ Corrección de contigs (opcional pero recomendado)
El flujo de trabajo estándar consta de dos pasos principales: corrección y andamiaje.

## 1. Corrección de Contigs (Opcional pero recomendado)

Antes de unir, RagTag puede romper contigs que tengan errores de ensamblaje (quimeras) comparándolos con la referencia.

```
  #Sintaxis: ragtag.py correct <referencia.fasta> <mis_contigs.fasta>
ragtag.py correct referencia.fna mis_contigs.fasta -o ragtag_output_corrected -t 8

# Ejemplo:
ragtag.py correct Mi_genoma_de_referencia.fna mis_Scaffolds.fasta \
  -o ragtag_output_corrected \
  -t 8
```

## 2. Scaffolding (Andamiaje)
  Este comando ordena y orienta tus contigs (o los contigs ya corregidos).
```
  # Sintaxis: ragtag.py scaffold <referencia.fasta> <contigs_a_procesar.fasta>
  ragtag.py scaffold referencia.fna ragtag_output_corrected/ragtag.correct.fasta -o ragtag_output_scaffold -t 8

# Ejemplo
ragtag.py scaffold Mi_genoma_de_referencia.fna \
  ragtag_output_corrected/ragtag.correct.fasta \
  -o ragtag_output_scaffold \
  -t 8
```
Parámetros clave

`-o` → Directorio de salida
``-t`` → Hilos (CPUs)
``-f`` → Tamaño mínimo de contig (default: 100 bp)
``-u`` → Excluir contigs no alineados (opcional)

## 🤖 Automatización (Script Batch)
Para procesar múltiples genomas (por ejemplo, varias cepas de Geosmithia o Xylaria) simultáneamente contra una misma referencia, utiliza el siguiente script.

Script: (`batch_ragtag.sh`)[!batch_ragtag.sh]

Dale permisos de ejecución:
`chmod +x batch_ragtag.sh`

## 📂 Archivos de Salida
En la carpeta de resultados (ragtag_output_scaffold o ragtag_final_results), los archivos clave son:

| Archivo | Descripción |
|-------|------------|
| `ragtag.scaffold.fasta` | El resultado final. Genoma ensamblado en scaffolds, listo para anotación. |
| `ragtag.scaffold.stats` | Estadísticas del andamiaje (N50, número de gaps, longitud total). |
| `ragtag.scaffold.agp` | Archivo AGP v2.0 que describe las coordenadas exactas de unión entre contigs. |
| `ragtag.scaffold.confidence.txt` | Puntajes de confianza para cada unión realizada. |
