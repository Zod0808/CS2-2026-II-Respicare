# UNIVERSIDAD PRIVADA DE TACNA
## FACULTAD DE INGENIERÍA
### ESCUELA DE INGENIERÍA DE SISTEMAS

---

# Reporte de Laboratorio Nº 03
## "Pruebas de Software"

### Curso: Construcción de Software II
### Docente: Mag. Ricardo Eduardo Valcárcel Alvarado
### Estudiante: Cesar Fabian Chávez Linares
### Fecha: Octubre 2026
### TACNA – PERÚ

---

## 1. Información del Evento Práctico

### 1.1. Título del Evento Práctico
```
Elaboración de Casos de Prueba sobre los Métodos más Relevantes de la
Solución RespiCare (Sistema Web y Móvil para la Detección de Enfermedades
Respiratorias en Tacna).
```

### 1.2. Objetivos (según Guía de Laboratorio Nº 03)

- ✅ Elaborar casos de prueba de software sobre los métodos más relevantes de la solución.
- ✅ Establecer cuándo utilizar los diferentes tipos de prueba en el proyecto.

### 1.3. Equipos, Materiales y Recursos

- Software ofimático (Word/Excel) para la documentación del proyecto.
- Node.js 18+, Python 3.12, Jest, Supertest, Pytest (stack ya utilizado por RespiCare).
- Aplicación RespiCare diseñada e implementada en Construcción de Software I.
- Formatos de despliegue de la documentación del proyecto (plantilla de caso de prueba UPT).

### 1.4. Marco Teórico (resumen de `AseguramientoCalidad.pptx`)

| Tipo de prueba | ¿Cuándo se usa en RespiCare? |
|:---|:---|
| **Unitaria** | Al validar la lógica interna de un método aislado (p. ej. `RandomForestDiseaseClassifier.predict()`, `reviewPrediction()`), con dependencias externas simuladas (mocks). |
| **Integración** | Al validar que un método se comporta correctamente junto con sus dependencias reales (base de datos de prueba, endpoint FastAPI/Express). |
| **Funcional / Aceptación** | Al validar un flujo de negocio completo end-to-end (p. ej. paciente reporta síntomas → recibe diagnóstico → médico revisa y firma). |
| **Rendimiento** | Al validar que un método crítico (ensemble de IA, endpoints de emergencia) cumple el SLA definido en los RNF del proyecto. |
| **Regresión** | Al re-ejecutar la suite completa (Jest/Pytest) tras cada cambio, para asegurar que una corrección no rompe comportamiento ya probado. |
| **Smoke** | Al validar rápidamente, tras un despliegue, que `/health` y los endpoints críticos responden antes de correr la suite completa. |
| **Exploratoria** | Al usar manualmente el Panel del Médico o el análisis de síntomas sin un script previo, buscando comportamientos inesperados. |
| **Estática** | Al ejecutar `tsc --noEmit` / linting antes de ejecutar una sola prueba, para detectar errores de tipo sin correr el código. |

---

## 2. Selección de los Métodos más Relevantes

Se seleccionaron 5 métodos que concentran el riesgo clínico y técnico más alto de la solución: si fallan de forma silenciosa, afectan directamente el diagnóstico, la atención de una emergencia o la validez legal de una revisión médica. Para cada uno se definió **un caso de prueba de éxito** y **un caso de prueba con escenario de error**, en línea con lo exigido por la Guía de Laboratorio Nº 03.

| Nº | Método | Archivo | Rol en RespiCare |
|:---:|:---|:---|:---|
| 1 | `RandomForestDiseaseClassifier.predict()` | `ai-services/ml_models/random_forest_model.py:162` | Clasificación de enfermedad respiratoria a partir de síntomas |
| 2 | `SHAPDiseaseExplainer.explain_prediction()` | `ai-services/shap_explainer.py:86` | Explicabilidad de la predicción de IA (RNF-011) |
| 3 | `CoughAnalysisService.analyze()` | `ai-services/services/cough_analysis_service.py:437` | Análisis espectral de audio de tos |
| 4 | `EmergencyService.detectEmergencyFromSymptoms()` | `backend/src/services/emergencyService.ts:478` | Detección automática de emergencia clínica |
| 5 | `AIAnalysisReviewService.reviewPrediction()` | `backend/src/services/aiAnalysisReviewService.ts:94` | Revisión y firma electrónica del médico sobre una predicción de IA |

---

## 3. Casos de Prueba Elaborados

### 3.1. `RandomForestDiseaseClassifier.predict(symptoms, all_symptoms)`

**Caso de prueba de éxito — CP-RF01-E**
- **Entrada:** `symptoms = ['tos', 'sibilancias', 'dificultad para respirar']`, modelo entrenado (`is_trained = True`).
- **Acción:** se llama a `predict()`.
- **Resultado esperado:** no se activa ninguna regla de emergencia (`is_emergency = False`); retorna `disease`, `confidence`, `top_3_predictions` y `feature_importance` ordenados por relevancia.

**Caso de prueba con escenario de error — CP-RF01-F**
- **Entrada:** `symptoms = ['cianosis']` (coincide con `emergency_rules['critical']`), **o bien** se llama a `predict()` antes de entrenar el modelo (`is_trained = False`).
- **Acción:** se llama a `predict()`.
- **Resultado esperado:** con síntoma crítico, el método **no llega a invocar el modelo**: retorna de inmediato `{'disease': 'Emergencia médica', 'confidence': 1.0, 'is_emergency': True, 'warning': 'BUSCAR ATENCIÓN MÉDICA INMEDIATA'}`. Sin modelo entrenado, lanza `ValueError('Model not trained yet')`.

### 3.2. `SHAPDiseaseExplainer.explain_prediction(symptoms, patient_age)`

**Caso de prueba de éxito — CP-SHAP01-E**
- **Entrada:** modelo cargado (`load_model()` ejecutado previamente), `symptoms = "tos, sibilancias, dificultad respiratoria, opresion pecho"`.
- **Resultado esperado:** retorna `disease`, `confidence`, `top_3_predictions` y `explanation` con `positive_factors`, `negative_factors` y `decision_factors` (top 5) ordenados por `feature_importance`.

**Caso de prueba con escenario de error — CP-SHAP01-F**
- **Entrada:** `self.model = None` (modelo no cargado).
- **Resultado esperado:** el método no lanza excepción; retorna un error controlado `{'error': 'Model not loaded'}`, evitando que el fallo se propague al endpoint `/v1/ml-explanation`.

### 3.3. `CoughAnalysisService.analyze(audio_data, ...)`

**Caso de prueba de éxito — CP-COUGH01-E**
- **Entrada:** audio `.wav` válido (≥3 s, decodificado desde base64).
- **Resultado esperado:** extrae features espectrales (zcr, mfcc, centroide, rolloff), clasifica el tipo de tos y retorna severidad, características y nivel de urgencia.

**Caso de prueba con escenario de error — CP-COUGH01-F**
- **Entrada:** `librosa` no disponible en el entorno (`_check_dependencies()` falla) o audio base64 corrupto.
- **Resultado esperado:** `_extract_audio_features()` lanza `ImportError('librosa not available...')` de forma controlada; a nivel de endpoint (`/api/v1/audio/cough`), un base64 inválido retorna 400/422 sin caída del servicio.

### 3.4. `EmergencyService.detectEmergencyFromSymptoms(userId, patientId, symptoms, vitalSigns, location)`

**Caso de prueba de éxito — CP-EMG01-E**
- **Entrada:** `symptoms = ['dificultad_respiratoria_severa']` (coincide con `criticalSymptoms`) **con** `location` GPS válida.
- **Resultado esperado:** llama a `createEmergency()` y retorna un `EmergencyResponse` con `severity: 'critical'`.

**Caso de prueba con escenario de error — CP-EMG01-F**
- **Entrada:** mismos síntomas críticos, **pero sin `location`** (GPS no disponible).
- **Resultado esperado:** el método registra una advertencia (`logger.warn`) y retorna `null` — **no** se crea una emergencia sin ubicación, evitando un despacho de ambulancia sin destino válido. (Síntomas no críticos también retornan `null`, sin generar alerta alguna.)

### 3.5. `AIAnalysisReviewService.reviewPrediction(analysisId, decision, doctorId, comments, signature, adjustment)`

**Caso de prueba de éxito — CP-REV01-E**
- **Entrada:** `doctorId` válido con rol `doctor`, `analysisId` existente con `review.status = 'pending'`, `decision = 'approved'`, `signature` presente.
- **Resultado esperado:** invoca `analysis.approve()`, crea una alerta al paciente (si `analysis.patientId` existe) y retorna el `AIAnalysisDocument` actualizado.

**Caso de prueba con escenario de error — CP-REV01-F**
- **Entrada (3 variantes):** (a) `doctorId` inexistente o con rol distinto de `doctor`/`admin`; (b) `analysisId` inexistente; (c) `analysisId` cuyo `review.status` ya no es `'pending'` (doble revisión).
- **Resultado esperado:** lanza `AppError('El médico revisor no existe o no es válido', 400)`, `AppError('Análisis de IA no encontrado', 404)` o `AppError('Este análisis de IA ya fue revisado', 400)` respectivamente, sin modificar el documento.

---

## 4. Evidencia y Trazabilidad

Los 5 métodos anteriores no son casos aislados: ya cuentan con pruebas automatizadas reales ejecutándose en la suite del proyecto, y están documentados con el mismo criterio de "escenario de éxito / escenario de error" en los artefactos de `UNIDAD_01`:

| Método | Caso de prueba (catálogo) | Especificación detallada | Suite automatizada |
|:---|:---|:---|:---|
| `RandomForestDiseaseClassifier.predict()` | CP-002-U | `Especificacion_CP-002_Prediccion_Diagnostico_Sintomas.docx` | `test_symptom_analysis_service.py` |
| `SHAPDiseaseExplainer.explain_prediction()` | CP-006-U / CP-006-I | `Especificacion_CP-006_Explicacion_SHAP.docx` | `test_shap_explainer.py` |
| `CoughAnalysisService.analyze()` | CP-003-U / CP-003-I | `Especificacion_CP-003_Analisis_Audio_Tos.docx` | `test_audio_analyzer_endpoints.py` |
| `EmergencyService.detectEmergencyFromSymptoms()` | CP-004-U | `Especificacion_CP-004_Motor_Reglas_Emergencia.docx` | `emergencyService.test.ts` |
| `AIAnalysisReviewService.reviewPrediction()` | CP-007-U | `Especificacion_CP-007_Panel_Doctor_Revision.docx` | `aiAnalysisReviewService.test.ts` |

- Catálogo completo (68+ casos de prueba, 5 tipos por requerimiento): `UNIDAD_01/Casos_Prueba/G01_casos_prueba_V2.xlsx` y `Codigo_Fuente/Documentation/pruebas/Catalogo_de_Pruebas_RespiCare.xlsx`.
- Especificaciones individuales por caso: `Codigo_Fuente/Documentation/pruebas/especificaciones/`.
- Plan de pruebas y criterios de aceptación: `Codigo_Fuente/Documentation/pruebas/Plan_de_Pruebas_de_Software_RespiCare.docx`.

---

## 5. Conclusiones

1. **Elegir el tipo de prueba correcto según el riesgo del método, no de forma uniforme.** Los métodos con impacto clínico directo (emergencia, revisión médica) se probaron también a nivel de integración y aceptación, mientras que utilidades internas se cubrieron solo a nivel unitario.
2. **Los escenarios de error son tan importantes como los de éxito.** En los 5 métodos analizados, el camino de error no es un `throw` genérico: cada uno implementa una regla de negocio explícita (emergencia detectada antes de consultar el modelo, revisión bloqueada si ya fue atendida, emergencia no despachada sin GPS), y probarla evita que esa regla se rompa silenciosamente en un futuro cambio.
3. **Las pruebas reducen costo y riesgo a largo plazo.** Consistente con la teoría de `AseguramientoCalidad.pptx`: detectar en `aiAnalysisReviewService.test.ts` que una doble revisión no estaba bloqueada es mucho más barato que descubrirlo en producción con una predicción médica revisada dos veces de forma contradictoria.
4. **La documentación de pruebas ya existente en el proyecto (catálogo + especificaciones + suites) satisface el objetivo de este laboratorio**; este reporte consolida y justifica, sobre 5 métodos concretos, el criterio con el que se aplicaron los distintos tipos de prueba.

---

## 6. Referencias

- Guía de Laboratorio Nº 03 – Pruebas de Software (SI-083), Mag. Ricardo Eduardo Valcárcel Alvarado — Construcción de Software II, UPT.
- Material teórico: `AseguramientoCalidad.pptx` (tipos de prueba, automatización, pruebas estáticas/dinámicas, alfa/beta).
- `Codigo_Fuente/Documentation/pruebas/Plan_de_Pruebas_de_Software_RespiCare.docx`.
