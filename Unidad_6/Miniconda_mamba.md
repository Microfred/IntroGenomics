# 🧬 Gestión de Entornos y Flujos de Trabajo en Bioinformática

**Materia:** Ciencias Genómicas (QBP) | **Institución:** ENCB - IPN  
**Instructor:** Dr. Juan Alfredo Hernández García (@Microfred)

Este repositorio contiene la guía práctica para la instalación y el manejo de **Miniconda** y **Mamba**, herramientas esenciales para garantizar la reproducibilidad en análisis genómicos de procariotas (bacterias) y eucariotas (hongos/levaduras).

---

## 📋 Tabla de Contenidos
1. [El Problema: El Infierno de las Dependencias](#1-el-problema-el-infierno-de-las-dependencias)
2. [Solución: Conda y Mamba](#2-solución-conda-y-mamba)
3. [Guía de Instalación (Linux y Mac)](#3-guía-de-instalación-paso-a-paso)
4. [Configuración de Canales](#4-configuración-de-canales)
5. [Cheatsheet de Comandos](#5-cheatsheet-de-comandos)
6. [Workflows: Bacterias vs Hongos](#6-workflows-bacterias-vs-hongos)
7. [Reproducibilidad](#7-reproducibilidad)

---
# Instalación y Configuración de Conda/Mamba para Bioinformática

[![Conda](https://img.shields.io/badge/conda-environment-blue?logo=anaconda)](https://docs.conda.io/)
[![Bioconda](https://img.shields.io/badge/bioconda-packages-green?logo=leaflet)](https://bioconda.github.io/)
[![Conda-Forge](https://img.shields.io/badge/conda--forge-dependencies-orange?logo=condaforge)](https://conda-forge.org/)
[![macOS](https://img.shields.io/badge/macOS-Intel%20%7C%20Apple%20Silicon-black?logo=apple)](https://www.apple.com/macos/)
[![Linux](https://img.shields.io/badge/Linux-compatible-yellow?logo=linux)](https://www.kernel.org/)
[![HPC](https://img.shields.io/badge/HPC-ready-purple)](#)
[![Reproducible](https://img.shields.io/badge/Reproducible-Yes-brightgreen)](#)
[![Environment](https://img.shields.io/badge/environment.yml-included-success?logo=yaml)](./environment.yml)

---

## 1. El Problema: El "Infierno de las Dependencias"

En el laboratorio, nunca mezclarías reactivos de dos protocolos distintos en el mismo tubo porque contaminarías la muestra.
En bioinformática ocurre más o menos lo mismo con los softwares.

El problema principal es la **incompatibilidad de versiones**:

* Un **ensamblador** de bacterias puede requerir `Python 3.7`.
* Un **anotador** de hongos puede requerir `Python 3.10`.

* **Resultado:** Si instalas ambos en tu sistema base, uno dejará de funcionar.

> **La Solución:** Usar **Entornos Virtuales**. ¿qué es un __entorno__ Son "contenedores" aislados donde instalamos software específico para una tarea específica, sin que interfiera con los demás.

---

## 2. Solución: Conda y Mamba

### 🐍 Miniconda
Es un sistema de gestión de paquetes ligero. A diferencia de *Anaconda* (que instala miles de librerías innecesarias), Miniconda solo incluye lo esencial (Python y Conda).

### 🐍 Mamba
Es la evolución de Conda, reescrita en C++.
* **Problema de Conda:** A veces es lento resolviendo qué librerías necesita instalar.
* **Ventaja de Mamba:** Es extremadamente rápido y realiza descargas en paralelo.
* **Resumen:** *Mamba hace lo mismo que Conda, pero mucho más rápido.*

---

## 3. Guía de Instalación Paso a Paso

### Opción A: 🐧 Linux (Ubuntu, Pop!_OS, Debian)

1. Abre tu terminal (`Ctrl + Alt + T`).
2. Descarga e instala Miniconda:

```bash
# Crear directorio
mkdir -p ~/miniconda3

# Descargar script
wget [https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh](https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh) -O ~/miniconda3/miniconda.sh

# Ejecutar instalación
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3

# Inicializar shell
~/miniconda3/bin/conda init bash
# (Nota: Si usas zsh, cambia 'bash' por 'zsh')
