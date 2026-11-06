# ☁️ APIs y Cloud Computing | APIs & Cloud Computing

## ¿Qué es? | What is it?
**ES:** El Cloud Computing es el suministro de servicios de computación (servidores, almacenamiento, bases de datos) a través de Internet. Una API (Application Programming Interface) es el "puente" que permite que dos aplicaciones se comuniquen entre sí.

**EN:** Cloud Computing is the delivery of computing services (servers, storage, databases) over the Internet. An API (Application Programming Interface) is the "bridge" that allows two applications to communicate with each other.

---

## 🌐 APIs: El Puente de los Datos | APIs: The Data Bridge

Para un profesional de datos, las APIs son la fuente principal de información externa.

### 1. REST APIs (El estándar)
Utilizan el protocolo HTTP y se basan en "Recursos".
- **Métodos Principales**:
  - `GET`: Solicitar datos (Ej: Obtener el precio actual del Bitcoin).
  - `POST`: Enviar datos para crear algo nuevo (Ej: Enviar un registro de sensor a una DB).
  - `PUT/PATCH`: Actualizar datos existentes.
  - `DELETE`: Borrar un recurso.

### 2. Formato de Intercambio: JSON
Casi todas las APIs modernas responden en formato **JSON (JavaScript Object Notation)**, que es la estructura que luego convertimos en un Diccionario de Python.

---

## ☁️ Cloud Computing: El Ecosistema Profesional | Cloud Ecosystem

Ya no instalamos todo en nuestra laptop; usamos el Cloud para escalar.

### 1. Los Tres Grandes Proveedores
- **AWS (Amazon Web Services)**: El más usado, la mayor cantidad de herramientas.
- **Azure (Microsoft)**: Ideal para empresas que ya usan el ecosistema Microsoft (Power BI, Active Directory).
- **GCP (Google Cloud Platform)**: Líder en herramientas de IA y Big Data (BigQuery).

### 2. Modelos de Servicio
- **IaaS (Infrastructure as a Service)**: Alquilas la máquina virtual y tú instalas todo (Ej: AWS EC2).
- **PaaS (Platform as a Service)**: Te dan la plataforma y tú solo subes el código (Ej: Heroku, Google App Engine).
- **SaaS (Software as a Service)**: Usas el software ya terminado (Ej: Gmail, Salesforce).

---

## 💻 Implementación Técnica de una API en Python | Technical Implementation

#### Ejemplo: Consumiendo una API con la librería `requests`
```python
import requests

# 1. Definir el endpoint de la API
url = "https://api.exchangerate-api.com/v4/latest/USD"

# 2. Hacer la petición GET
response = requests.get(url)

if response.status_code == 200:
    data = response.json() # Convertir la respuesta JSON a un diccionario de Python
    tasa_euro = data['rates']['EUR']
    print(f"La tasa de cambio USD a EUR es: {tasa_euro}")
else:
    print("Error al conectar con la API")
```

---

## Aplicación Real en Datos | Real-world Data Application

**Caso: Pipeline de Datos Automatizado**
1. Un script de Python en la **Terminal Linux** se ejecuta cada hora.
2. El script hace una petición **API** para obtener el precio de materias primas.
3. Los datos se guardan en **PostgreSQL** corriendo en una instancia de **Azure (Cloud)**.
4. El proceso entero está empaquetado en un contenedor **Docker**.
5. El resultado final se visualiza en un dashboard de **Power BI**.

---

## Conceptos Relacionados | Related Concepts
- [[08 - Ingeniería y Herramientas|Docker]] $\longleftrightarrow$ Desplegamos contenedores en la nube para crear APIs.
- [[02 - Programación]] $\rightarrow$ El uso de librerías como `requests` o `FastAPI` para crear y consumir APIs.

---
#cloud #api #aws #azure #gcp #backend
