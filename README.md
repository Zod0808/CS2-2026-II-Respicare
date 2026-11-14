# CS2 2026-II — RespiCare

Repositorio académico del curso **Construcción de Software II** (Universidad Privada de Tacna, semestre 2026-II), usado para organizar, gestionar y dar seguimiento a los entregables, laboratorios y evidencias de las tres unidades del curso, aplicando el proyecto **RespiCare** (Sistema Web y Móvil para la detección de enfermedades respiratorias en Tacna) como caso de aplicación.

## Datos del curso

| Campo | Detalle |
|---|---|
| Curso | Construcción de Software II |
| Docente | Ing. Ricardo Valcárcel Alvarado |
| Semestre | 2026-II |
| Modalidad de trabajo | Grupal |

## Integrantes

| Nombre | Código | Rol |
|---|---|---|
| Cesar Fabian Chavez Linares | 2019063854 | Líder de equipo / Desarrollo |
| Piero Alexander Paja de la Cruz | 2020067576 | Documentación |

## Organización del repositorio

El repositorio se organiza en tres carpetas principales, una por cada unidad académica del curso:

```
CS2-2026-II-Respicare/
├── UNIDAD_01/                     # Revisión y planificación del software (ejemplo inicial)
│   ├── Revision_Codigo/
│   ├── Especificaciones_Funcionales/
│   ├── Casos_Prueba/
│   ├── Plan_Iteracion/
│   ├── Laboratorios_Evidencias/
│   └── Codigo_Fuente/             # Código fuente completo del proyecto RespiCare
├── UNIDAD_02/                     # Calidad, Calendarización, Plan de Despliegue, Documento de Configuración de Software, Plan de Pruebas
│   ├── Calidad/
│   ├── Calendarizacion/
│   ├── Plan_Despliegue/
│   ├── Documento_Configuracion_Software/
│   └── Plan_Pruebas/
├── UNIDAD_03/                     # Se definirá progresivamente según el avance del curso
└── Laboratorio_Instrucciones/     # Enunciado del laboratorio y formato de informe de uso de GitHub
```

- **UNIDAD_01** contiene la estructura de ejemplo solicitada: revisión de código, especificaciones funcionales, casos de prueba/catálogo de pruebas, plan de iteración, laboratorios con sus evidencias, y el código fuente del proyecto.
- **UNIDAD_02** cubre los temas indicados por el docente para la segunda unidad: calidad, calendarización, plan de despliegue, documento de configuración de software y plan de pruebas.
- **UNIDAD_03** se completará progresivamente conforme el docente indique los temas, actividades y entregables correspondientes.
- Cada subcarpeta de `UNIDAD_01` y `UNIDAD_02` incluye un `README.md` que enlaza a los documentos y/o rutas de código correspondientes dentro de `Codigo_Fuente/`, evitando duplicar archivos.

## Proyecto base: RespiCare

RespiCare es un sistema web y móvil para la detección temprana de enfermedades respiratorias en Tacna, compuesto por backend (Node.js/TypeScript), frontend web (React), aplicación móvil (Capacitor + Next.js / React Native) y un servicio de IA (Python) para análisis de síntomas y señales de wearables. El código fuente completo se encuentra en `UNIDAD_01/Codigo_Fuente/`.
