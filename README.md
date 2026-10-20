# Web Scraping de Supermercados Hiperdino (Práctica 1 - UOC)

Proyecto académico de la asignatura **Tipología y ciclo de vida de los datos**, del
Grado de Ciencia de Datos de la Universitat Oberta de Catalunya (UOC).

> **Integrantes del grupo:** Daniel Pérez Rodríguez, Nicolangelo Famiglietti Acuña  
> **Asignatura:** Tipología y ciclo de vida de los datos  
> **Práctica:** Práctica 1 — Web Scraping

## Objetivo

El objetivo es obtener de forma responsable información pública del catálogo de
productos de la tienda en línea de [Supermercados Hiperdino](https://www.hiperdino.es/),
con especial atención a su oferta en Canarias. Los datos recopilados permiten
explorar y analizar precios, marcas, ofertas y categorías de productos.

El scraping se realiza con fines educativos y de análisis de datos. La
disponibilidad de productos, precios y campos puede cambiar con el tiempo, con
la zona de entrega o con la estructura de la página web.

## Estructura del repositorio

```text
.
├── source/
│   └── scraper.py
├── dataset/
│   ├── hiperdino_cat.csv
│   └── categorias_descubiertas.txt
├── .gitignore
├── README.md
└── requirements.txt
```

- **`source/`**: código fuente del proyecto. `scraper.py` contiene el
  descubrimiento y filtrado de categorías, el acceso al sitio, el parseo del
  catálogo y la exportación del CSV.
- **`dataset/`**: resultados generados. `hiperdino_cat.csv` contiene los
  productos extraídos y `categorias_descubiertas.txt` conserva la lista local de
  categorías para reutilizarla en ejecuciones posteriores. El script también
  puede crear `debug_page.html` para inspeccionar el HTML renderizado de la
  primera categoría.
- **`requirements.txt`**: dependencias de Python necesarias para ejecutar el
  proyecto.
- **`.gitignore`**: excluye el entorno virtual, archivos de bytecode y los CSV
  almacenados en `dataset/`.
- **`README.md`**: documentación, instalación, uso y criterios del proyecto.

Los archivos de datos y de depuración se generan al ejecutar el programa; puede
que no estén incluidos en el repositorio.

## Requisitos e instalación

Se recomienda Python 3.10 o posterior. Para el modo `--selenium`, también se
necesita Google Chrome o Chromium compatible. Selenium 4 suele gestionar
automáticamente el controlador del navegador; el primer inicio puede requerir
conexión a Internet.

### Clonar el repositorio

Sustituye `https://github.com/nfamiglietti/hiperdino-web-scraping-uoc-pract1` por la URL del repositorio del grupo:

```bash
git clone https://github.com/nfamiglietti/hiperdino-web-scraping-uoc-pract1
cd source
```

### Windows (PowerShell)

```powershell
py -3 -m venv venv
.\venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Si PowerShell impide la activación del entorno virtual, permite scripts para la
sesión actual y vuelve a activarlo:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\venv\Scripts\Activate.ps1
```

### macOS o Linux

```bash
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Para salir del entorno virtual en cualquier plataforma:

```bash
deactivate
```

## Modo de uso y ejecución

Ejecuta los comandos desde la raíz del repositorio y con el entorno virtual
activado.

### Listar categorías

Descubre las categorías, las imprime y las guarda en
`dataset/categorias_descubiertas.txt`. Este modo no descarga productos:

```bash
python source/scraper.py --list-categories
```

Para renderizar el menú JavaScript con Chrome headless:

```bash
python source/scraper.py --list-categories --selenium
```

### Ejecución de prueba

Procesa como máximo las tres primeras categorías y usa Selenium para renderizar
el contenido dinámico:

```bash
python source/scraper.py --selenium --limit 3
```

### Extracción completa

```bash
python source/scraper.py --selenium
```

En modo Selenium, el script admite además un máximo de páginas por categoría:

```bash
python source/scraper.py --selenium --limit 3 --max-pages-per-category 2
```

`--max-pages-per-category` es un entero positivo y su valor predeterminado es
`3`. La paginación depende de los enlaces que presente la página y de lo que
permita `robots.txt`. Si no se indica `--selenium`, se usa `requests`; dado que
el catálogo puede renderizarse con JavaScript, ese modo puede no encontrar los
productos.

El CSV final se escribe en `dataset/hiperdino_cat.csv`, con codificación
`utf-8-sig` para facilitar su apertura en hojas de cálculo.

## Cumplimiento ético y `robots.txt`

El proyecto aplica las siguientes medidas:

- **User-Agent académico:** `UOC-TipologiaDatos-AcademicScraper/1.0`.
- **Retardo de cortesía:** espera al menos 2 segundos entre peticiones HTTP
  gestionadas por la sesión. La navegación Selenium aplica esperas adicionales
  para renderizado y carga perezosa.
- **URLs de categorías limpias:** el descubrimiento guarda enlaces sin
  parámetros ni fragmentos. Si la web ofrece paginación mediante parámetros,
  solo se intenta seguir el enlace cuando `robots.txt` lo permite para el
  User-Agent configurado.
- **Consulta de `robots.txt`:** antes de solicitar páginas o seguir enlaces, el
  script evalúa la URL con `RobotFileParser.can_fetch`. No se deben ignorar ni
  eludir las reglas aplicables.

En la versión consultada para este proyecto,
[`robots.txt`](https://www.hiperdino.es/robots.txt) incluye `Disallow: /*?*`
y varias rutas técnicas dentro de un grupo que nombra a `AhrefsBot`, `PetalBot`,
`GPTBot`, `SiteAuditBot` y `dotbot`. No se observa un grupo `User-agent: *` que
extienda esas reglas a todos los agentes. Por ello, `RobotFileParser` interpreta
las reglas de ese grupo para los agentes enumerados; para el User-Agent
académico del proyecto, no las considera una prohibición general. La política
del sitio puede cambiar: se debe volver a consultar `robots.txt` antes de cada
ejecución o actualización del scraper.

La existencia de una regla técnica que se aplique a otros agentes no autoriza a
eludir controles de acceso, límites de frecuencia ni condiciones de uso. El
proyecto evita rutas técnicas explícitamente prohibidas y mantiene la pausa
entre peticiones. La paginación puede utilizar URLs con `?p=...`; su acceso
queda sujeto a la evaluación de `can_fetch` y no se debe forzar si la política
vigente la prohíbe para el agente.

## Descripción del dataset

El archivo `dataset/hiperdino_cat.csv` almacena una fila por producto encontrado
en el catálogo. Las columnas son:

| Variable | Tipo de dato esperado | Descripción |
|---|---|---|
| `Nombre` | Texto | Nombre comercial del producto. |
| `Categoría` | Texto | Categoría principal utilizada durante la extracción. |
| `Subcategoría` | Texto | Subcategoría del producto según el campo disponible en el sitio. |
| `Precio` | Numérico o texto | Precio final publicado; el formato depende del dato proporcionado por el sitio. |
| `Precio/Unidad` | Texto | Precio unitario y unidad de referencia, cuando está disponible. |
| `Marca` | Texto | Marca indicada por el sitio o, si no aparece, una aproximación basada en el nombre del producto. |
| `Oferta` | Texto categórico (`Sí`/`No`) | Indica si el producto muestra precio especial o descripción promocional. |
| `EAN` | Texto | Código EAN del producto; se conserva como texto para mantener ceros iniciales. |

La disponibilidad de cada campo depende de la información publicada por
Hiperdino. Los campos ausentes pueden quedar vacíos. El esquema describe la
salida prevista y no garantiza que todas las filas tengan todos los valores.

## Licencia y citación

### Licencia del conjunto de datos

Como licencia propuesta para la **compilación académica generada por el grupo**,
se utiliza [Creative Commons
Reconocimiento-NoComercial-CompartirIgual 4.0 Internacional (CC BY-NC-SA
4.0)](https://creativecommons.org/licenses/by-nc-sa/4.0/). Esta declaración se
refiere al trabajo de recopilación y organización del grupo y no transfiere
derechos sobre las marcas, fotografías, descripciones ni otros contenidos de
Hiperdino. Antes de publicar o redistribuir los datos, el grupo debe revisar las
condiciones de uso del sitio y confirmar que la licencia propuesta es adecuada.

**DOI de Zenodo:** [añadir el DOI una vez publicado](https://doi.org/).

> Pérez Rodríguez D., Famiglietti Acuña N. (2026). *Web Scraping de
> Supermercados Hiperdino: Práctica 1 — Tipología y ciclo de vida de los datos*
> (versión [número]). Zenodo. https://doi.org/[DOI]

Datos extraidos el 20 de octubre de 2026
