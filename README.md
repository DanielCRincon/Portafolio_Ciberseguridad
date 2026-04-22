# Portafolio de Ciberseguridad: Laboratorio de Auditoría y Pentesting

Este repositorio documenta la implementación, configuración y ejecución de pruebas de seguridad en una infraestructura local controlada para el entrenamiento en técnicas de hacking ético.

## 📋 Índice
- [Infraestructura del Proyecto](#infraestructura-del-proyecto)
- [Optimización y Hardening Inicial](#optimización-y-hardening-inicial)
- [Despliegue de Objetivos (Vulnerable Targets)](#despliegue-de-objetivos-vulnerable-targets)
- [Metodología de Auditoría](#metodología-de-auditoría)
- [Resumen de Comandos (Cheat Sheet)](#resumen-de-comandos-cheat-sheet)

---

## 🖥️ Infraestructura del Proyecto

### Hardware y Sistema Base
- **Servidor de Laboratorio (Víctima):** Dell Optiplex 5070 (Mini PC)
  - **Procesador:** Intel Core i5
  - **RAM:** 16GB
  - **SO:** Ubuntu 24.04 LTS
  - **IP Local:** 192.168.40.21
- **Máquina de Control (Atacante):** Laptop con Kali Linux
- **Conectividad:** Acceso mediante SSH para administración y red local para pruebas de penetración

---

## 🔧 Optimización y Hardening Inicial

Se eliminaron servicios de alta demanda de recursos (IA local) para estabilizar el servidor y liberar espacio en disco:

- **Saneamiento:** Remoción de contenedores Hermes AI y desinstalación de Ollama
- **Espacio recuperado:** ~5.15 GB de imágenes Docker y archivos de modelos
- **Mantenimiento de Docker:** Ejecución de `prune` para asegurar un entorno de contenedores limpio y eficiente

---

## 🎯 Despliegue de Objetivos (Vulnerable Targets)

Se utiliza la contenerización para desplegar aplicaciones vulnerables de forma aislada.

### OWASP Juice Shop
- **Tipo:** Aplicación web moderna (Node.js/Express/Angular) diseñada con fallos de seguridad intencionados basados en el OWASP Top 10
- **Método de despliegue:** Docker
- **Puerto de servicio:** 3000
- **Comando de inicialización:**
  ```bash
  sudo docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
  ```
- **Objetivo:** Práctica de ataques Web, SQLi, XSS y Broken Access Control

---

## 🛠️ Metodología de Auditoría

### Fase 1: Reconocimiento de Red y Servicios
Utilización de Nmap para identificar puertos abiertos y versiones de software.

**Comando ejecutado:**
```bash
nmap -sV -p 3000 192.168.40.21
```

**Análisis de flags:**
- `-sV`: Detecta la versión de los servicios en ejecución
- `-p 3000`: Escaneo específico sobre el puerto donde reside el objetivo

**Hallazgo:** Identificación de un servidor web activo y capacidad de realizar "Banner Grabbing" para obtener metadatos del encabezado HTTP.

### Fase 2: Enumeración y Fuzzing de Directorios
Uso de FFUF para descubrir rutas no indexadas o archivos ocultos mediante ataques de diccionario.

**Comando ejecutado:**
```bash
ffuf -u http://192.168.40.21:3000/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

**Análisis de flags:**
- `-u`: URL del objetivo
- `FUZZ`: Palabra clave que el software reemplaza por cada entrada del diccionario
- `-w`: Ruta local al diccionario de palabras (common.txt)

**Hallazgo:** Localización de rutas con códigos de estado 200 (OK) y 301 (Redirect) que no son visibles en el menú principal de la aplicación.

### Ajustes Técnicos Recomendados
Al analizar la salida de `ffuf` proporcionada, se observa que múltiples rutas devuelven el mismo tamaño de respuesta (`Size: 75002`) y estado `200`. Esto indica que el servidor está respondiendo con una página personalizada (como el "index" o una página de error decorada) para cualquier ruta inexistente.

Para mejorar la precisión del manual y los resultados, debes ajustar el comando de `ffuf` para filtrar esos falsos positivos.

**Siguiente paso técnico:**
```bash
ffuf -u http://192.168.40.21:3000/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 75002
```

### 4. Explotación: Bypass de Autenticación (SQL Injection)
- **Vulnerabilidad:** Inyección SQL (SQLi)
- **Vector de Ataque:** Formulario de Login (`/#/login`)
- **Payload utilizado:** `' OR 1=1--`

**Descripción del hallazgo:**
La aplicación concatena directamente la entrada del usuario en la consulta SQL sin validación previa. Al introducir una tautología (`1=1`) seguida de un comentario (`--`), se anula la verificación de la contraseña y se fuerza al motor de búsqueda a devolver el primer usuario de la base de datos (usualmente el administrador).

**Prueba manual:**
- **Email:** `' OR 1=1--`
- **Password:** (cualquier valor)

**Impacto:** Acceso no autorizado con privilegios de administrador, compromiso total de la confidencialidad de la base de datos.

### 5. Mitigación y Buenas Prácticas
Para corregir la vulnerabilidad de SQLi detectada, se deben implementar las siguientes medidas en la capa de persistencia:

- **Implementación de Prepared Statements:** Reemplazar la concatenación de strings por consultas parametrizadas para forzar la separación entre código y datos.

- **Validación de Entradas:** Aplicar filtros de tipo de dato y longitud en el lado del servidor.

- **Seguridad en Base de Datos:** Configurar el usuario de la aplicación con permisos restringidos (Data Reader/Writer únicamente) para limitar el radio de explosión en caso de compromiso.

---

## 📝 Resumen de Comandos (Cheat Sheet)

### Gestión del Entorno (Limpieza)
```bash
# Detener y borrar contenedores antiguos
sudo docker stop hermes && sudo docker rm hermes

# Eliminar imágenes pesadas
sudo docker rmi -f <IMAGE_ID>

# Limpieza profunda de Docker
sudo docker system prune -af --volumes

# Desactivar servicio Ollama
sudo systemctl stop ollama && sudo systemctl disable ollama

# Eliminar binarios y modelos
sudo rm $(which ollama) && sudo rm -rf /usr/share/ollama
```

### Preparación del Laboratorio
```bash
# Actualización de repositorios
sudo apt update && sudo apt upgrade -y

# Instalación de herramientas esenciales
sudo apt install -y nmap ffuf wordlists curl git

# Despliegue de Juice Shop
sudo docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop
```

### Ejecución de Pruebas (Desde Kali Linux)
```bash
# Escaneo de servicios
nmap -sV -p 3000 192.168.40.21

# Fuzzing de directorios
ffuf -u http://192.168.40.21:3000/FUZZ -w /usr/share/wordlists/dirb/common.txt

# Fuzzing filtrando falsos positivos (por tamaño de respuesta)
ffuf -u http://192.168.40.21:3000/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 75002
```
