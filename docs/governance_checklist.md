# ⚖️ Checklist de Gobernanza de IA — Detección en Sitios de Construcción

## 1. Privacidad y Consentimiento

### Estado: ⚠️ Requiere atención antes de despliegue

| Aspecto | Evaluación |
|---------|-----------|
| ¿Se capturan rostros identificables? | Sí — las imágenes contienen rostros de trabajadores |
| ¿Existe consentimiento informado? | No verificable — el dataset es público (Roboflow Universe) pero no documenta el proceso de consentimiento de los sujetos fotografiados |
| ¿Cumple con regulaciones locales? | Pendiente — verificar LFPDPPP (México) y regulaciones laborales aplicables |
| ¿Se aplica anonimización? | No — los rostros no están difuminados en el dataset |

### Recomendaciones para despliegue en producción:
- Implementar difuminado automático de rostros en las imágenes almacenadas
- Obtener consentimiento informado por escrito de los trabajadores monitoreados
- Publicar aviso de privacidad visible en el sitio de construcción
- Designar un responsable de protección de datos
- Limitar el acceso a las imágenes originales a personal autorizado

---

## 2. Minimización de Datos

### Principio: Recopilar solo los datos estrictamente necesarios

| Dato | ¿Necesario? | Justificación |
|------|-------------|---------------|
| Imagen completa de la escena | Sí | Contexto espacial para detectar proximidad persona-maquinaria |
| Rostros individuales | No | Solo se necesita la posición de la cabeza para evaluar casco |
| Identidad del trabajador | No | El sistema detecta EPP, no identifica personas |
| Ubicación GPS de la cámara | Sí | Para mapear zonas de riesgo |
| Historial temporal de imágenes | Limitado | Retener solo el periodo mínimo para auditoría (máx. 30 días) |

### Medidas implementables:
- Procesar las imágenes en tiempo real y almacenar solo los metadatos de detección (clase, confianza, coordenadas del bbox)
- Eliminar las imágenes originales tras el procesamiento, salvo incidentes de seguridad
- Aplicar borrado automático de imágenes almacenadas tras el periodo de retención

---

## 3. Declaración de Limitaciones — Cuándo NO Usar Este Sistema

### ❌ Este sistema NO debe usarse para:

1. **Toma de decisiones disciplinarias automatizadas** — No usar la detección de NO-Hardhat/NO-Mask como único criterio para sancionar a un trabajador. Se requiere verificación humana.

2. **Sustitución de supervisores de seguridad** — El modelo es una herramienta de apoyo, no un reemplazo del personal de seguridad certificado.

3. **Entornos no representados en el entrenamiento** — Obras subterráneas, nocturnas, o con condiciones climáticas extremas (lluvia intensa, neblina) no están representadas en el dataset de entrenamiento.

4. **Certificación de cumplimiento regulatorio** — Las detecciones del modelo no constituyen evidencia legal de cumplimiento o incumplimiento de normas de seguridad.

5. **Monitoreo encubierto de productividad** — Usar la detección de personas para medir tiempos de trabajo o productividad es un uso no previsto y éticamente cuestionable.

6. **Entornos con clases no entrenadas** — El modelo no detecta arneses de seguridad, lentes protectores, guantes, protección auditiva u otro EPP no incluido en las 10 clases.

### Condiciones mínimas para uso aceptable:
- Supervisión humana de las alertas generadas
- Imágenes con resolución suficiente (mínimo 640×480)
- Iluminación diurna o artificial adecuada
- Cámara posicionada a <30m de la zona monitoreada

---

## 4. Análisis de Riesgos: Falsos Negativos vs Falsos Positivos

### Matriz de Impacto

| Tipo de Error | Escenario | Severidad | Consecuencia |
|---------------|-----------|-----------|-------------|
| **Falso Negativo** — No detectar persona sin casco | Trabajador entra a zona de caída de objetos sin ser alertado | **Crítica** | Posible lesión craneal grave o muerte |
| **Falso Negativo** — No detectar maquinaria | Excavadora opera en proximidad de personas sin alerta | **Crítica** | Posible atropellamiento o aplastamiento |
| **Falso Negativo** — No detectar ausencia de chaleco | Trabajador invisible para operadores de maquinaria | **Alta** | Riesgo de atropellamiento por baja visibilidad |
| **Falso Positivo** — Detectar cono inexistente | Sistema reporta zona señalizada cuando no lo está | **Media** | Falsa sensación de seguridad en zona no delimitada |
| **Falso Positivo** — Detectar chaleco donde no hay | Sistema valida EPP incorrecto | **Alta** | Trabajador expuesto con validación falsa |
| **Falso Positivo** — Detectar maquinaria inexistente | Alerta de equipo pesado sin que esté presente | **Baja** | Molestia operativa, posible desconfianza en el sistema |

### Conclusión de riesgo:
En este dominio, los **falsos negativos son más peligrosos** que los falsos positivos. Un FN en seguridad puede resultar en lesión o muerte. Un FP genera molestia o desconfianza pero no riesgo físico directo. Por lo tanto:

- **Priorizar el Recall** sobre la Precision para las clases críticas (NO-Hardhat, NO-Mask, machinery)
- Ajustar el umbral de confianza hacia abajo (e.g., 0.20) para las clases de seguridad
- Implementar alertas redundantes: si el modelo no detecta, el supervisor humano sigue siendo responsable

---

## 5. Sesgo y Equidad

### Sesgos identificados en el dataset:

| Sesgo | Descripción | Impacto potencial |
|-------|-------------|-------------------|
| **Geográfico** | Las imágenes provienen de sitios de construcción predominantemente de ciertas regiones; las condiciones de obra en México (materiales, vestimenta, tipos de maquinaria) pueden diferir | Menor rendimiento en contextos locales no representados |
| **Temporal** | Todas las imágenes son diurnas; no hay escenas nocturnas ni con iluminación artificial | El modelo falla en turnos nocturnos |
| **Demográfico** | Posible sesgo en la detección de cubrebocas según tono de piel y forma facial | Personas de ciertos grupos demográficos podrían tener mayor tasa de FN en Mask/NO-Mask |
| **De escala** | Predominan imágenes con maquinaria a distancia media; la variación de tamaño aparente es limitada | Detección reducida en objetos muy cercanos o muy lejanos |

### Mitigaciones:
- Evaluar métricas desagregadas por condiciones de iluminación y distancia
- Si se despliega en México, complementar con imágenes de obras locales para fine-tuning
- Monitorear la tasa de error diferencial por turno (diurno vs nocturno)

---

## 6. Transparencia

### Documentación requerida para usuarios del sistema:

- [x] Descripción del modelo y sus capacidades → Este repositorio
- [x] Limitaciones explícitas → Sección 3 de este documento
- [x] Clases detectables y no detectables → `class_definitions.md`
- [x] Métricas de rendimiento → `README.md`, sección de resultados
- [x] Análisis de errores → `error_analysis.md`
- [x] Licencias y atribuciones → `README.md`, sección de licencias

### Principios de transparencia:
1. **Los trabajadores deben saber** que están siendo monitoreados por un sistema de IA
2. **Los supervisores deben entender** que el sistema comete errores y no reemplaza la inspección visual
3. **Los tomadores de decisiones deben conocer** las métricas de rendimiento y sus limitaciones antes de autorizar el despliegue
4. **Las alertas del sistema deben incluir** el nivel de confianza de la detección

---

## 7. Tabla de Licencias

| Componente | Licencia | Obligaciones | Enlace |
|------------|----------|-------------|--------|
| **Código del proyecto** | MIT | Libre uso con atribución | [LICENSE](../LICENSE) |
| **Dataset Construction Site Safety** | CC BY 4.0 | Atribución al creador original | [Roboflow Universe](https://universe.roboflow.com/roboflow-universe-projects/construction-site-safety) |
| **YOLOv8 (Ultralytics)** | AGPL-3.0 | Código derivado debe ser open source; uso comercial requiere licencia enterprise | [Ultralytics](https://github.com/ultralytics/ultralytics/blob/main/LICENSE) |
| **MobileSAM** | Apache 2.0 | Libre uso con atribución y aviso de cambios | [GitHub](https://github.com/ChaoningZhang/MobileSAM) |
| **PyTorch** | BSD-3-Clause | Libre uso con atribución | [PyTorch](https://github.com/pytorch/pytorch/blob/main/LICENSE) |
| **Roboflow SDK** | Apache 2.0 | Libre uso con atribución | [Roboflow](https://github.com/roboflow/roboflow-python) |

### ⚠️ Nota sobre AGPL-3.0 (YOLOv8):
La licencia AGPL-3.0 de Ultralytics requiere que cualquier software que use YOLOv8 y se distribuya o despliegue como servicio (SaaS) también sea open source. Para uso comercial cerrado, se requiere adquirir una licencia Enterprise de Ultralytics. Este proyecto, al ser académico, cumple con los términos de la AGPL-3.0.

---

## Fecha de revisión

**Última actualización:** Marzo 2026  
**Próxima revisión recomendada:** Antes de cualquier despliegue en producción
