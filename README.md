# IntroGenomics

## Ciencias Genómicas | ENCB-IPN | Semestre 27-1

Materiales de apoyo para el curso de **Ciencias Genómicas**, dirigido a estudiantes de **Químico Bacteriólogo Parasitólogo (QBP), 10.º semestre**, de la Escuela Nacional de Ciencias Biológicas del Instituto Politécnico Nacional.

**Profesor:** Dr. Juan Alfredo Hernández García  
**Adscripción:** Laboratorio de Biología Molecular de Bacterias y Levaduras, Departamento de Microbiología, ENCB-IPN.  
**Periodo del curso:** 19 de octubre de 2026 al 11 de febrero de 2027.  
**Clases:** jueves; primera sesión el 22 de octubre de 2026.  
**Contacto académico:** avisos y entregas mediante el Google Classroom del grupo.

> Este curso conecta la pregunta biológica con el análisis de datos. Aprenderemos a justificar los procedimientos, interpretar los resultados y comunicar sus alcances y limitaciones.

## Accesos principales

| Recurso | Contenido |
| --- | --- |
| [Calendario de clases](https://github.com/Microfred/Programa-AQM2-y-AQV1/blob/main/README.md) | Fechas, temas, actividades y entregas del semestre. |
| [Unidad 1: Bash y manejo de archivos](Unidad_1_Bash.md) | Práctica progresiva con datos de ejemplo, comandos explicados y resultados esperados. |
| [Flujo genómico reproducible](Flujo_genomico_reproducible.md) | Ruta práctica de calidad de lecturas, ensamblaje, evaluación y anotación. |
| [Datos y ejemplos históricos](Prac_Uni1) | Material complementario para explorar archivos y scripts. |

El calendario comprende **15 sesiones**. No se programan clases el 24 ni el 31 de diciembre de 2026; el **7 de enero de 2027 sí hay clase**. Las dos últimas sesiones se destinan a proyectos. Los ajustes se comunicarán en Classroom y se reflejarán en el calendario enlazado.

## Propósito y resultados de aprendizaje

Integrar microbiología, biología molecular, bioquímica y bioinformática para analizar datos genómicos y comprender su relación con las demás ciencias ómicas.

Al finalizar el curso, el estudiante podrá:

1. Formular una pregunta biológica y seleccionar datos, controles y referencias pertinentes.
2. Utilizar Bash para organizar archivos, consultar secuencias y automatizar tareas sencillas.
3. Distinguir tecnologías de secuenciación, formatos de datos y criterios de calidad.
4. Procesar lecturas, obtener un ensamblaje y valorar fragmentación, cobertura y completitud.
5. Interpretar anotaciones, comparaciones genómicas, filogenias y rutas metabólicas.
6. Diferenciar metagenómica shotgun y metabarcoding, y evaluar resultados de diversidad microbiana.
7. Interpretar resultados básicos de transcriptómica, proteómica, metabolómica e integración multiómica.
8. Comunicar un análisis mediante un reporte científico y evidencias reproducibles.

## Organización temática

| Bloque temático | Contenido | Enfoque |
| --- | --- | --- |
| Fundamentos de las ómicas | Organización genómica, expresión y regulación; aplicaciones. | Preguntas biológicas y selección del enfoque experimental. |
| Herramientas computacionales | Bash, formatos, scripts, Markdown y entornos. | Ejercicios progresivos y documentación del trabajo. |
| Secuenciación | Sanger, Illumina, PacBio y Oxford Nanopore; diseño y datos públicos. | Comparación de tecnologías y selección de datos. |
| Análisis de genomas | Calidad, procesamiento, ensamblaje, mapeo, evaluación y anotación. | Eje práctico del proyecto. |
| Genómica comparada y evolución | Homología, ortología, pangenómica y filogenómica. | Interpretación de matrices, alineamientos y árboles. |
| Microbioma | Shotgun, amplicones, 16S/ITS, taxonomía y diversidad. | Casos guiados y análisis crítico del diseño. |
| Transcriptómica | RNA-seq, replicación, normalización y expresión diferencial. | Interpretación de resultados preparados. |
| Proteómica e integración | Proteínas, interacciones, estructura, metabolitos y rutas. | Integración de evidencia y límites de las predicciones. |

La secuencia de clases corresponde al calendario del semestre. Las carpetas y la unidad práctica de Bash conservan su identificación de origen; sus números no representan por sí solos el orden actual del curso.

## Preparación para las prácticas

- Conocimientos básicos de genética, microbiología, bioquímica y biología molecular.
- Equipo con GNU/Linux o macOS. Los ejercicios de programación se ejecutarán explícitamente con **Bash**.
- Editor de texto plano con soporte para Markdown, como [Visual Studio Code](https://code.visualstudio.com/).
- Para la primera práctica: Bash, `awk`, `grep`, `sed`, `tar`, `gzip` y `curl`.
- Para genómica: entorno de Conda/Mamba compatible con la arquitectura del equipo y los programas requeridos.
- Para los casos que lo necesiten: [R](https://www.r-project.org/) y [RStudio](https://docs.posit.co/ide/user/).

Antes de descargar datos, comprobaremos almacenamiento, memoria y tiempo de ejecución. Los datos de práctica serán pequeños; los genomas y las referencias del proyecto se acordarán con el profesor.

## Cómo trabajar con este repositorio

1. Consultar el tema y la actividad en el calendario.
2. Abrir la guía correspondiente y revisar sus objetivos, entradas y productos esperados.
3. Ejecutar los comandos en el orden indicado, dentro de una carpeta de práctica propia.
4. Registrar versiones, parámetros, resultados y decisiones en una bitácora.
5. Entregar las evidencias por Classroom y discutir su significado biológico.

Para descargar una copia con Git:

```bash
git clone https://github.com/Microfred/IntroGenomics.git
cd IntroGenomics
```

También puede utilizarse la opción **Code > Download ZIP** de GitHub. Si se trabaja sobre una copia propia, los resultados deben mantenerse separados de los archivos originales del curso.

## Proyecto final

El proyecto consiste en el **análisis integral de un genoma público** de una bacteria, levadura u hongo, con una pregunta biológica definida y un alcance viable.

| Componente | Evidencia mínima |
| --- | --- |
| Datos y diseño | Organismo, pregunta, accesiones, metadatos, referencias y justificación. |
| Lecturas | Calidad inicial/final y decisiones de procesamiento. |
| Ensamblaje | Método, parámetros, tamaño, fragmentación y evaluación. |
| Anotación | Estrategia adecuada al organismo y resultados estructurales/funcionales. |
| Filogenómica | Selección de genomas y marcadores, método, soporte e interpretación. |
| Ruta metabólica | Genes/proteínas identificados, pasos sustentados y vacíos de evidencia. |
| Comunicación | Reporte en PDF con estructura de artículo, figuras, tablas y referencias. |
| Reproducibilidad | Scripts, versiones, bases de datos, parámetros, registros y README de ejecución. |

Cada proyecto tendrá una exposición de **10-15 minutos**, más el espacio de preguntas asignado. El borrador completo se entrega el **28 de enero de 2027**; las exposiciones son el **4 y 11 de febrero**, y la versión final de todos los proyectos se entrega el **11 de febrero**.

Se distinguirán los análisis ejecutados por el estudiante de los resultados proporcionados para interpretación. Una ruta predicha o una función anotada deben presentarse como evidencia computacional, con sus limitaciones.

## Evaluación

Se considerarán prácticas, tareas, evaluaciones de fundamentos, participación y proyecto final. Las ponderaciones y condiciones de entrega se comunicarán en el encuadre y en Classroom.

Se valorará que cada estudiante pueda explicar **qué hizo, por qué lo hizo, qué obtuvo y qué significa biológicamente**. Los resultados deben estar respaldados por archivos y procedimientos identificables. Se citarán los datos, programas y fuentes utilizados, y se declararán apoyos externos relevantes.

## Material complementario histórico

Los siguientes documentos se conservan como consulta. Para la ejecución del flujo principal, seguir la [guía actualizada](Flujo_genomico_reproducible.md), que unifica nombres, rutas y criterios de evaluación.

| Material | Tema |
| --- | --- |
| [Unidad_2](Unidad_2/Readme_U2.md) | Interpretación de informes de FastQC. |
| [Unidad_3](Unidad_3/Readme.md) | Ejemplos ampliados de procesamiento, ensamblaje y mapeo. |
| [Scaffolding](Unidad_3/Unidad_3.1/README.md) | Uso de una referencia para ordenar/orientar contigs y discutir sesgos. |
| [Unidad_4](Unidad_4/Unidad4.md) | Estadísticas de ensamblaje y cobertura. |
| [Unidad_5](Unidad_5/Unidad5.md) | Anotación bacteriana y resultados de anotación. |
| [Unidad_6](Unidad_6/Miniconda_mamba.md) | Antecedentes de gestión de entornos. |
| [Unidad_7](Unidad_7/README.md) | Ejemplos de fastp y MultiQC. |
| [SPAdes y QUAST](Unidad_7/spades_quast.md) | Ejemplos de ensamblaje y evaluación. |
| [Unidad_8](Unidad_8/README.md) | Antecedentes de BUSCO y anotación en bacterias y hongos. |

## Autoría y reconocimiento de fuentes

**Responsable académico:** [Dr. Juan Alfredo Hernández García](https://github.com/Microfred), ENCB-IPN.

El material introductorio original reconoce adaptaciones de recursos de **Alicia Mastretta Yanes**. Se conserva este reconocimiento y se remite al repositorio [BioinfinvRepro](https://github.com/AliciaMstt/BioinfinvRepro) para consultar los materiales de origen.

Consultar el archivo [LICENSE](LICENSE) del repositorio y conservar las atribuciones que acompañan a los materiales reutilizados.

**Revisión editorial y técnica:** 19 de octubre de 2026.
