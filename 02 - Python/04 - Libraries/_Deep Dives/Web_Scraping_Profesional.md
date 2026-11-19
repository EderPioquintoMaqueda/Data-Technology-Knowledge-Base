# 🕸️ Web Scraping Profesional: BeautifulSoup y Scrapy

Cuando la información no está en una base de datos ni en un CSV, el Ingeniero de Datos debe saber extraerla de la web. El Web Scraping es el proceso de convertir el HTML no estructurado de una página en un dataset estructurado.

## 1. BeautifulSoup: El Escalpelos para Páginas Pequeñas

`BeautifulSoup` es ideal para extraer datos de páginas estáticas. Funciona analizando el árbol DOM del HTML.

### Flujo de Extracción:
1. **Request**: Obtener el HTML con la librería `requests`.
2. **Parsing**: Convertir el HTML en un objeto navegable con BeautifulSoup.
3. **Extracción**: Buscar etiquetas específicas (`find`, `find_all`, `select`).

```python
import requests
from bs4 import BeautifulSoup

url = "https://www.ejemplo-banco.com/tasas"
response = requests.get(url)
soup = BeautifulSoup(response.text, 'html.parser')

# Buscar todas las tasas de interés en una tabla
tasas = []
for fila in soup.find_all('tr'):
    cols = fila.find_all('td')
    if cols:
        tasas.append({'producto': cols[0].text, 'tasa': cols[1].text})
```

---

## 2. Scrapy: El Tanque para Extracción Masiva

Para extraer miles de páginas, BeautifulSoup es demasiado lento. **Scrapy** es un framework completo que permite:
- **Asincronía**: Descargar múltiples páginas simultáneamente.
- **Pipelines**: Limpiar y guardar los datos en la base de datos automáticamente mientras se descargan.
- **Manejo de Proxies**: Rotar IPs para evitar ser bloqueado por el servidor.

### Comparativa Técnica:
| Característica | BeautifulSoup | Scrapy |
| :--- | :--- | :--- |
| **Tipo** | Librería (Sencilla) | Framework (Completo) |
| **Velocidad** | Lenta (Síncrona) | Muy Rápida (Asíncrona) |
| **Curva Aprendizaje** | Muy Baja | Media |
| **Uso Ideal** | Una sola página / Script rápido | Crawlers masivos / Sitios complejos |

---

## 3. Ética y Legalidad (El Código del Scraper)

Un profesional no "ataca" un servidor. Sigue estas reglas:
1. **`robots.txt`**: Revisa siempre `url.com/robots.txt` para ver qué rutas están prohibidas.
2. **User-Agent**: Identifícate en los headers para que el administrador sepa quién eres.
3. **Rate Limiting**: Añade pausas (`time.sleep()`) entre peticiones para no causar un ataque de Denegación de Servicio (DoS).

#web-scraping #beautifulsoup #scrapy #data-extraction #python-pro
