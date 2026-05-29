# Simulación Forestal en SIMANFOR mediante Docker

---

🇪🇸 **Estás viendo el contenido del repositorio en español**  

🇬🇧 *[English version here](./README_english.md)*

---

[SIMANFOR](https://www.simanfor.es) es una herramienta de apoyo a la toma de decisiones que permite la simulación de alternativas de gestión forestal.

En este repositorio encontrarás la documentación pública sobre cómo ejecutar y utilizar el simulador de SIMANFOR en tu ordenador local a través de contenedores **Docker**, sin necesidad de realizar configuraciones de dependencias o de entornos de desarrollo complejos.

---

## 📜 Contenido

En este repositorio dispones de los siguientes archivos y recursos listos para su uso:

* 📂 **[ejemplos](file:///media/aitor/7FB15A9C1FA8F1DA/trabajo/repositorios/simanfor/docker/ejemplos)**: Estructuras y archivos listos para realizar simulaciones de prueba.
  * 📂 **[ejemplo_masas_puras](file:///media/aitor/7FB15A9C1FA8F1DA/trabajo/repositorios/simanfor/docker/ejemplos/ejemplo_masas_puras)**: Escenario para la simulación de masas puras en España.
  * 📂 **[ejemplo_masas_mixtas](file:///media/aitor/7FB15A9C1FA8F1DA/trabajo/repositorios/simanfor/docker/ejemplos/ejemplo_masas_mixtas)**: Escenario para la simulación de masas mixtas en España.
* 📖 **[cheatsheet_docker_basic.html](file:///media/aitor/7FB15A9C1FA8F1DA/trabajo/repositorios/simanfor/docker/cheatsheet_docker_basic.html)**: Guía rápida interactiva y tabla de referencia con los comandos de Docker más habituales para Windows y Linux.
---

## 🚀 Requisitos e Instalación

Para poder utilizar SIMANFOR de esta forma, únicamente necesitas tener Docker instalado y en funcionamiento en tu sistema operativo:

1. Descarga e instala **Docker Desktop** (para Windows o macOS) o **Docker Engine** (para sistemas GNU/Linux) desde su [sitio web oficial](https://www.docker.com/products/docker-desktop/).
2. Asegúrate de que el servicio de Docker esté activo ejecutando en tu terminal: 
   
```bash
docker --version
```

Si no estás familiarizado con Docker, te recomendamos abrir el archivo interactivo `cheatsheet_docker_basic.html` en cualquier navegador web para consultar una guía rápida de ayuda.

---

## 📥 Obtener la Imagen del Simulador

La imagen oficial del simulador de SIMANFOR se encuentra alojada públicamente en el registro de contenedores de GitHub (GitHub Container Registry). Puedes consultar el listado completo de imágenes y versiones disponibles en el enlace público de **[🔗 Paquetes de SIMANFOR-Dask](https://github.com/orgs/simanfor-dask/packages)**, siendo la imagen completa por defecto la que indicamos en el siguiente ejemplo de descarga:

```bash
docker pull ghcr.io/simanfor-dask/simulator:latest
```

Una vez completada la descarga, puedes comprobar que la imagen se ha guardado correctamente listando las imágenes locales:

```bash
docker image ls
```

### Comprobación de funcionamiento inicial

Para verificar que el contenedor se ejecuta correctamente sin errores en tu máquina, puedes lanzarlo de manera directa sin pasarle ningún parámetro:

```bash
docker run ghcr.io/simanfor-dask/simulator:latest
```

> [!NOTE]
> Este comando de prueba ejecutará una simulación por defecto que viene preconfigurada de manera interna dentro del contenedor. Sin embargo, dado que los contenedores son entornos aislados y efímeros, cualquier archivo de resultados generado por esta prueba interna se eliminará automáticamente una vez finalice la ejecución. Para trabajar con tus propios escenarios y almacenar los resultados en tu ordenador, sigue las instrucciones a continuación sobre **volúmenes locales**.

---

## 💾 Ejecución con Datos Propios (Volúmenes Locales)

Para poder pasarle tus propios archivos de inventario (.xlsx) y escenarios (.json) al simulador, y que este a su vez te devuelva los archivos de resultados (.xlsx) directamente en tu ordenador local, es imprescindible utilizar **volúmenes de Docker** (`-v`). 

Los volúmenes actúan como carpetas compartidas entre tu ordenador (el sistema host) y el contenedor de Docker.

### 1. Estructura de carpetas recomendada

Para evitar errores y simplificar la ejecución, se recomienda crear una carpeta de trabajo específica en tu ordenador (por ejemplo, llamada `simanfor_docker/`) que contenga exactamente la siguiente estructura de carpetas y subcarpetas:

```text
simanfor_docker/
├── inputs/            <-- Aquí colocarás el archivo .json del escenario y la plantilla .xlsx del inventario
└── outputs/           <-- Aquí escribirá automáticamente el simulador los archivos .xlsx de resultados
```

### 2. Configuración interna del escenario (.json)

Para que el simulador que se ejecuta dentro del contenedor localice adecuadamente los archivos de entrada y escriba los de salida a través de las rutas montadas en Docker, es muy importante configurar correctamente las variables de ruta en tu archivo `.json` de escenario:

* **Ruta de entrada (`input`)**: Debe apuntar a `../inputs/[nombre_del_inventario].xlsx`
* **Ruta de salida (`output_path`)**: Debe apuntar a `../tests/outputs/[prefijo_deseado]__`

Esto se debe a que, internamente, el simulador de SIMANFOR se ejecuta desde el directorio de trabajo `/app/src/` del contenedor, por lo que las carpetas montadas externamente serán accesibles de forma relativa mediante el uso de un directorio superior (`../`).

Un ejemplo del fragmento de configuración en el archivo `.json`:
```json
{
  "user": "Tu Nombre",
  "overwrite_output_file": "YES",
  "output_path": "../tests/outputs/simulacion_salida__",
  ...
  "operations": [
    {
      ...
      "variables": {
        "input": "../inputs/masas_puras_Espana-plantilla.xlsx"
      }
    }
  ]
}
```

### 3. Comandos de simulación según el Sistema Operativo

Una vez tengas configurada tu carpeta local `simanfor_docker/` con tus archivos correspondientes en `inputs/`, navega desde la terminal a dicha carpeta de trabajo y ejecuta el comando de Docker adecuado para tu sistema operativo y consola:

#### 🐧 En GNU/Linux y macOS (Bash / Zsh)
Utilizamos la variable de entorno `${PWD}` para referenciar de forma automática la ruta absoluta del directorio actual:

```bash
docker run \
  -v ${PWD}/outputs:/app/tests/outputs \
  -v ${PWD}/inputs:/app/inputs:ro \
  ghcr.io/simanfor-dask/simulator:latest \
  -s /app/inputs/test_pure_models_spain_plantilla.json
```

#### 🪟 En Windows (PowerShell)
Utilizamos la variable `${PWD}` junto al carácter especial de escape de saltos de línea (backtick `` ` ``):

```powershell
docker run `
  -v ${PWD}/outputs:/app/tests/outputs `
  -v ${PWD}/inputs:/app/inputs:ro `
  ghcr.io/simanfor-dask/simulator:latest `
  -s /app/inputs/test_pure_models_spain_plantilla.json
```

#### 🪟 En Windows (CMD - Símbolo del sistema)
Utilizamos la variable `%cd%` junto al circunflejo (`^`) como carácter de continuación de línea:

```cmd
docker run ^
  -v %cd%/outputs:/app/tests/outputs ^
  -v %cd%/inputs:/app/inputs:ro ^
  ghcr.io/simanfor-dask/simulator:latest ^
  -s /app/inputs/test_pure_models_spain_plantilla.json
```

---

## 📊 Ejemplos Listos para Probar

Para que puedas verificar de inmediato el correcto funcionamiento, hemos incluido dos escenarios completos de prueba listos para ejecutar dentro de la carpeta `ejemplos/`:

### Prueba con masas puras en España
1. Abre tu terminal en el directorio `ejemplos/ejemplo_masas_puras/`.
2. Ejecuta el comando correspondiente de tu sistema operativo indicando el archivo de escenario:
   * En Linux/macOS:
     ```bash
     docker run -v ${PWD}/outputs:/app/tests/outputs -v ${PWD}/inputs:/app/inputs:ro ghcr.io/simanfor-dask/simulator:latest -s /app/inputs/test_pure_models_spain_plantilla.json
     ```
3. Verifica que la simulación finaliza correctamente y que en tu directorio local `ejemplos/ejemplo_masas_puras/outputs/` han aparecido 3 hojas de cálculo Excel correspondientes a las parcelas del inventario proyectadas en el tiempo.

### Prueba con masas mixtas en España
1. Abre tu terminal en el directorio `ejemplos/ejemplo_masas_mixtas/`.
2. Ejecuta el comando correspondiente:
   * En Linux/macOS:
     ```bash
     docker run -v ${PWD}/outputs:/app/tests/outputs -v ${PWD}/inputs:/app/inputs:ro ghcr.io/simanfor-dask/simulator:latest -s /app/inputs/test_mixed_models_spain_plantilla.json
     ```
3. Comprueba el resultado en la carpeta `outputs/` local de dicho ejemplo.

---

## ℹ️ Más información

Para más contenidos de SIMANFOR, vídeos, publicaciones y cómo citar, visita:

* [🔗 Más información sobre SIMANFOR](https://github.com/simanfor/.github/blob/main/docs/more_info_spanish.md)

---

![simanfor](https://raw.githubusercontent.com/simanfor/.github/main/skills/simanfor-public-md-documentation/resources/simanfor.png)

![SMART_GIR](https://raw.githubusercontent.com/simanfor/.github/main/skills/simanfor-public-md-documentation/resources/SMART_GIR.png)
