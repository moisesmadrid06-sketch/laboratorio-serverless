# Laboratorio de Práctica – Práctica con Serverless

**Asignatura:** Introducción al Cómputo en la Nube
**Unidad:** 3 – Competencias Fundamentales en la Nube
**Tutor:** Vladimir Mate
**Estudiante:** _MOISES SAMUEL BERRIOS RIVAS_ (_moises_madrid06@yahoo.es_)

## Objetivo
Implementar un entorno Serverless local con **OpenFaaS** (Docker Desktop + Kubernetes) para desarrollar, desplegar y ejecutar una función en Python que registra datos desde un formulario web, genera un token UUID único y produce un archivo JSON descargable con firma SHA-256.

## Estructura del repositorio
```
laboratorio-serverless/
├── registro-serverless/
│   ├── handler.py          # Función Serverless (GET = formulario, POST = JSON)
│   └── requirements.txt
├── template/               # Plantillas oficiales python3-http (faas-cli template store pull)
├── build/                  # Archivos usados para construir la imagen Docker
├── stack.yaml              # Configuración de despliegue de OpenFaaS
├── registro_<token>.json   # EVIDENCIA PRINCIPAL generada por la función
├── .gitignore
└── README.md
```

## Cómo funciona `handler.py`
| Componente | Función |
|---|---|
| GET | Presenta el formulario HTML |
| POST | Recibe nombre, apellido y correo |
| uuid | Token aleatorio único por registro |
| platform | Características del entorno (contenedor Linux) |
| socket | Nombre (hostname) del contenedor |
| datetime | Fecha y hora UTC |
| hashlib | Firma SHA-256 del contenido |
| json | Genera el archivo descargable |

## Comandos usados
```powershell
# Parte 2 – OpenFaaS
kubectl create namespace openfaas
kubectl create namespace openfaas-fn
helm repo add openfaas https://openfaas.github.io/faas-netes/
helm upgrade --install openfaas openfaas/openfaas --namespace openfaas --set functionNamespace=openfaas-fn --set generateBasicAuth=true --set serviceType=ClusterIP
kubectl port-forward -n openfaas svc/gateway 8080:8080

# Parte 3 – Función
faas-cli template store pull python3-http
faas-cli new --lang python3-http registro-serverless
helm upgrade openfaas openfaas/openfaas --namespace openfaas --reuse-values --set functions.imagePullPolicy=Never
docker build -t registro-serverless:v1 ./build/registro-serverless
docker run --rm -p 8081:8080 registro-serverless:v1
```

## Evidencia
El archivo `registro_<token>.json` de la raíz del repositorio contiene: nombre, apellido, correo, token UUID, fecha/hora UTC, información del entorno de ejecución (Linux / WSL2 dentro del contenedor), firma SHA-256 y estado de ejecución.

> Nota: el sistema operativo que aparece en el JSON es el del **contenedor Linux**, no Windows 11.
