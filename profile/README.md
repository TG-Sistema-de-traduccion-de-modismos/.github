# Sistema de Traducción de Modismos en Bogotá

Proyecto de grado — Pontificia Universidad Javeriana  
Departamento de Ingeniería de Sistemas  

---

## Descripción General

El Sistema de Traducción de Modismos en Bogotá es una arquitectura basada en microservicios orquestados que permite interpretar modismos bogotanos a partir de texto o audio del usuario, utilizando modelos de Inteligencia Artificial (IA) especializados en procesamiento de lenguaje natural (PLN) y reconocimiento de voz.

El sistema permite:
- Recibir entrada de texto o audio desde una aplicación móvil.  
- Identificar modismos y entregar su significado contextual.  
- Procesar audio mediante Whisper (speech-to-text).  
- Analizar texto con Beto y Phi (modelos de PLN).  
- Integrar todos los servicios bajo un esquema seguro, escalable y modular.  

---

## Arquitectura General

La solución está basada en una arquitectura de microservicios distribuida, con contenedores Docker, un orquestador en Google Cloud, y un API Gateway (Kong) como punto central de acceso.


<img width="1258" height="427" alt="HLD comparation drawio (2)" src="https://github.com/user-attachments/assets/0bc76d08-d23d-4ec9-aeb8-3b622d093eda" />



### Componentes Principales

| Componente | Tecnología | Descripción |
|-------------|-------------|-------------|
| **API Gateway** | Kong + HTTPS | Punto de entrada único. Gestiona autorización con tokens Firebase y redirige las peticiones válidas al orquestador. |
| **Orquestador** | FastAPI + Google Cloud | Coordina los flujos entre los servicios (Whisper, Beto y Phi) dependiendo del tipo de entrada (audio/texto). |
| **phi-service** | FastAPI + Google Cloud | Servicio intermedio que neutraliza modismos y gestiona la semántica. |
| **whisper-service** | FastAPI + Google Cloud | Servicio encargado del reconocimiento de voz (Speech-to-Text). |
| **beto-service** | FastAPI + Google Cloud | Analiza el texto y detecta expresiones idiomáticas específicas de Bogotá. |
| **phi-model / beto-model / whisper-model** | FastAPI + Docker | Modelos optimizados para GPU RTX 5070, ejecutados localmente con Tailscale. |
| **Nginx + Certificación SSL** | Nginx + Certbot + Let's Encrypt / Tailscale | Enrutamiento y gestión HTTPS para todos los servicios. |

---

## Seguridad y Comunicación

Todo el sistema se comunica únicamente a través de HTTPS, garantizando confidencialidad e integridad.

- **Certificación en la nube:**  
  Se utiliza Certbot + Let’s Encrypt + Nginx para generación y renovación automática de certificados, junto con DuckDNS para los dominios dinámicos.  

- **Certificación en entorno local:**  
  En los modelos desplegados en la máquina personal se usa Tailscale para la conexión privada y la generación de certificados, junto con Nginx para el enrutamiento hacia los contenedores Docker.  

- **Autenticación y Autorización:**  
  El API Gateway (Kong) valida los tokens de Firebase. Solo si son válidos, la petición se reenvía al orquestador; de lo contrario, se rechaza el acceso.  

---

## Flujo de Procesamiento

### Entrada por Texto
---
APP Móvil → API Gateway → Orquestador → Beto-Service → Beto-model → Phi-Service → Phi-model → Orquestador → API Gateway -> APP Móvil 
---
### Entrada por Texto
---
APP Móvil → API Gateway → Orquestador → Whisper-Service → Whisper-model → Beto-Service → Beto-model → Phi-Service → Phi-model → Orquestador → API Gateway -> APP Móvil 
---

El flujo es secuencial:  
- Los audios siempre pasan primero por Whisper antes de llegar a Beto o Phi.  
- Los textos no pueden entrar directamente a Phi sin pasar por Beto.  

---

## Despliegue y Orquestación

- Los servicios de orquestación y API Gateway están desplegados en Google Cloud Run / Compute Engine, administrados mediante Docker Compose y scripts de despliegue automatizados.  
- Los modelos de IA (Phi, Beto, Whisper) están contenedorizados y ejecutados en la máquina local del equipo de desarrollo con Docker + GPU RTX 5070, interconectados a la nube mediante Tailscale VPN.  
- Cada contenedor cuenta con un servidor FastAPI y Nginx configurado como proxy inverso con HTTPS.  

---

## Tecnologías Principales

| Categoría | Herramienta / Tecnología |
|------------|--------------------------|
| Lenguaje base | Python 3.11 |
| Frameworks | FastAPI, Transformers, Torch |
| Contenedores | Docker |
| Orquestación | Google Cloud Run / Docker Compose |
| Gateway | Kong API Gateway |
| Seguridad | Firebase Auth, HTTPS, Certbot, Let’s Encrypt |
| Red privada | Tailscale VPN |
| Proxy inverso | Nginx |
| Hardware de inferencia | RTX 5070 GPU (local) |

---

## Ventajas Arquitectónicas

- Escalabilidad: los microservicios pueden ser replicados o reemplazados sin alterar el resto del sistema.  
- Modularidad: cada modelo (Whisper, Beto, Phi) es independiente; se puede actualizar o sustituir fácilmente.  
- Interoperabilidad: comunicación unificada bajo HTTP/HTTPS y formato JSON.  
- Seguridad integrada: autenticación centralizada y certificados TLS en todos los niveles.  
- Optimización local: los modelos se ejecutan en GPU, aprovechando al máximo la infraestructura del desarrollador.  

---

## Licencia y Derechos

Los derechos morales corresponden a los autores del proyecto.  
La Universidad Javeriana posee una licencia no exclusiva y perpetua para fines académicos, de investigación y docencia.  

Desarrollado por:  
**Pablo Javier Escobar Gómez, Juan Felipe González Quintero, Gabriel Espitia Romero, Oscar Alejandro Rodríguez Gómez**  
**Dirección:** Andrea del Pilar Rueda Olarte  

