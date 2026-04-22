# Portafolio de Ciberseguridad: Laboratorio de Auditoría y Pentesting

Este repositorio documenta la implementación, configuración y ejecución de pruebas de seguridad en una infraestructura local controlada.

## 📋 Índice
- [Infraestructura del Proyecto](#infraestructura-del-proyecto)
- [Optimización del Entorno](#optimización-del-entorno)
- [Despliegue de Objetivos (Vulnerable Targets)](#despliegue-de-objetivos-vulnerable-targets)
- [Metodología y Herramientas](#metodología-y-herramientas)
- [Ajustes Técnicos Recomendados](#ajustes-técnicos-recomendados)

---

## 🖥️ Infraestructura del Proyecto

### Hardware y Sistema Base
- **Servidor de Laboratorio:** Dell Optiplex 5070 (Mini PC)
- **Sistema Operativo:** Ubuntu 24.04 LTS
- **Conectividad:** Acceso remoto persistente mediante protocolo SSH desde máquina de control
- **Máquina de Control:** Kali Linux (Entorno de herramientas ofensivas)

---

## 🔧 Optimización del Entorno

Se realizó un saneamiento profundo del servidor para dedicar la totalidad de recursos (CPU/RAM/Disco) a tareas de ciberseguridad, eliminando servicios de inferencia de IA y agentes locales:

- **Eliminación de Hermes AI:** Remoción de contenedores y redes virtuales
- **Desactivación de Ollama:** Detención de servicios y borrado de modelos (weights) para liberar espacio crítico en disco (5GB+)
- **Mantenimiento de Docker:** Ejecución de `prune` para asegurar un entorno de contenedores limpio y eficiente

---

## 🎯 Despliegue de Objetivos (Vulnerable Targets)

Para el entrenamiento en técnicas de explotación, se utilizan entornos contenerizados que permiten aislar las vulnerabilidades del sistema host.

### OWASP Juice Shop
- **Tipo:** Aplicación web moderna con vulnerabilidades intencionadas (OWASP Top 10)
- **Despliegue:** Imagen Docker `bkimminich/juice-shop` mapeada al puerto 3000
- **Objetivo:** Práctica de ataques Web, SQLi, XSS y Broken Access Control

---

## 🛠️ Metodología y Herramientas

### 1. Reconocimiento de Red (Nmap)
- **Herramienta:** `nmap` (Network Mapper)
- **Propósito:** Escaneo de puertos y detección de servicios/versiones (Banner Grabbing)
- **Resultado:** Identificación del puerto 3000/TCP abierto y confirmación del servicio web activo

### 2. Enumeración de Directorios (FFUF)
- **Herramienta:** `ffuf` (Fuzz Faster U Fool)
- **Propósito:** Descubrimiento de rutas ocultas y archivos no indexados mediante diccionarios (wordlists)
- **Resultado:** Identificación de la estructura de archivos de la aplicación para localizar puntos de entrada no públicos

---

## ⚙️ Ajustes Técnicos Recomendados

Al analizar la salida de `ffuf` proporcionada, se observa que múltiples rutas devuelven el mismo tamaño de respuesta (`Size: 75002`) y estado `200`. Esto indica que el servidor está respondiendo con una página personalizada (como el "index" o una página de error decorada) para cualquier ruta inexistente.

Para mejorar la precisión del manual y los resultados, debes ajustar el comando de `ffuf` para filtrar esos falsos positivos.

**Siguiente paso técnico:**

Repite el escaneo filtrando por tamaño de respuesta para identificar solo las rutas que son realmente distintas:

```bash
ffuf -u http://192.168.40.21:3000/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 75002
```
