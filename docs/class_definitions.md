# 📋 Definiciones de Clases — Construction Site Safety

## Resumen

El dataset contiene 10 clases orientadas a la seguridad en sitios de construcción. Se dividen en tres categorías funcionales:

1. **EPP presente** (equipo de protección personal detectado correctamente)
2. **EPP ausente** (violación de norma de seguridad)
3. **Entidades del sitio** (personas, vehículos, maquinaria, señalización)

---

## Clase 0: Hardhat (Casco de Seguridad)

**Definición:** Casco de seguridad rígido presente en la cabeza de una persona.

**Criterios de etiquetado:**
- El bounding box encierra el casco visible, incluyendo visera y ala
- Se etiqueta cuando el casco está sobre la cabeza de una persona (no en el suelo)
- Cascos de diferentes colores (blanco, amarillo, azul, naranja) se agrupan en la misma clase
- Mínimo 30% del casco debe ser visible para etiquetar

**Confusiones comunes:** Gorras o sombreros → NO son Hardhat. Cascos de motocicleta → NO son Hardhat.

---

## Clase 1: Mask (Cubrebocas/Mascarilla)

**Definición:** Cubrebocas o mascarilla cubriendo nariz y boca de una persona.

**Criterios de etiquetado:**
- El bounding box encierra la mascarilla visible en el rostro
- Incluye mascarillas quirúrgicas, N95, y cubrebocas de tela
- Debe cubrir al menos nariz y boca para considerarse "Mask"
- Mascarillas colgando del cuello → se etiqueta como NO-Mask

**Confusiones comunes:** Bufandas cubriendo el rostro → depende del contexto. Caretas protectoras → categoría distinta, pero en este dataset se agrupan.

---

## Clase 2: NO-Hardhat (Sin Casco)

**Definición:** Cabeza de una persona visible SIN casco de seguridad.

**Criterios de etiquetado:**
- El bounding box encierra la cabeza descubierta
- Se etiqueta cuando la persona está en zona de obra donde el casco es obligatorio
- Personas con gorras, sombreros o sin nada en la cabeza califican
- Señala una **violación de seguridad**

**Importancia AECO:** Clase crítica para alertas automatizadas. Un falso negativo (no detectar a alguien sin casco) es potencialmente peligroso.

---

## Clase 3: NO-Mask (Sin Cubrebocas)

**Definición:** Rostro de una persona visible SIN cubrebocas/mascarilla.

**Criterios de etiquetado:**
- El bounding box encierra el área del rostro donde debería estar la mascarilla
- Aplica cuando la normativa del sitio requiere uso de mascarilla
- Mascarilla en la barbilla o colgando del cuello → NO-Mask

**Contexto:** Relevante especialmente en entornos con polvo, materiales tóxicos o protocolos sanitarios.

---

## Clase 4: NO-Safety Vest (Sin Chaleco de Seguridad)

**Definición:** Torso de una persona visible SIN chaleco reflectante/de seguridad.

**Criterios de etiquetado:**
- El bounding box encierra el torso de la persona sin chaleco visible
- Aplica cuando la zona requiere uso obligatorio de chaleco de alta visibilidad
- Ropa de color brillante sin bandas reflectantes → NO-Safety Vest
- Señala una **violación de seguridad**

---

## Clase 5: Person (Persona)

**Definición:** Persona completa o parcialmente visible en el sitio de construcción.

**Criterios de etiquetado:**
- El bounding box encierra la persona completa o la porción visible
- Se etiqueta independientemente del EPP que porte
- Una misma persona puede tener múltiples etiquetas simultáneas (Person + Hardhat + Safety Vest)
- Mínimo 30% del cuerpo visible para etiquetar

**Nota:** Es la clase más frecuente en el dataset. Sirve como referencia base para las clases de EPP.

---

## Clase 6: Safety Cone (Cono de Seguridad)

**Definición:** Cono de señalización/seguridad en el sitio de obra.

**Criterios de etiquetado:**
- El bounding box encierra el cono completo o la porción visible
- Incluye conos naranjas estándar, conos con bandas reflectantes
- Conos apilados: cada cono visible se etiqueta individualmente si es distinguible
- Conos caídos o volteados también se etiquetan

**Importancia AECO:** Delimitan zonas de riesgo. Su presencia o ausencia ayuda a evaluar el cumplimiento de protocolos de seguridad.

---

## Clase 7: Safety Vest (Chaleco de Seguridad)

**Definición:** Chaleco reflectante/de alta visibilidad portado por una persona.

**Criterios de etiquetado:**
- El bounding box encierra el chaleco visible en el torso
- Debe tener bandas reflectantes o ser de color de alta visibilidad (amarillo, naranja, verde neón)
- Chalecos parcialmente cubiertos por chamarras → se etiqueta si las bandas son visibles

---

## Clase 8: machinery (Maquinaria Pesada)

**Definición:** Equipo pesado de construcción como excavadoras, retroexcavadoras, grúas, bulldozers, rodillos compactadores, etc.

**Criterios de etiquetado:**
- El bounding box encierra la maquinaria completa o la porción visible
- Incluye: excavadoras, grúas (torre y móviles), bulldozers, retroexcavadoras, cargadores frontales, rodillos, montacargas
- No incluye herramientas manuales o equipos pequeños
- Maquinaria estacionada o en operación se etiqueta igual

**Importancia AECO:** La proximidad de maquinaria pesada a personas es un factor de riesgo principal en obras. La detección permite alertas de zona de peligro.

---

## Clase 9: vehicle (Vehículo)

**Definición:** Vehículos motorizados en el sitio de construcción (no maquinaria pesada).

**Criterios de etiquetado:**
- El bounding box encierra el vehículo completo o la porción visible
- Incluye: camiones de volteo, camionetas, camiones de carga, vehículos de transporte de personal
- Diferencia con machinery: los vehículos son para transporte; la maquinaria es para operaciones de construcción
- Vehículos estacionados o en movimiento se etiquetan igual

**Confusiones comunes:** Camiones grúa → machinery (si tienen brazo articulado visible). Camiones de volteo → vehicle (función principal es transporte).

---

## Tabla Resumen

| ID | Clase | Categoría | Criticidad |
|----|-------|-----------|------------|
| 0 | Hardhat | EPP presente | Media |
| 1 | Mask | EPP presente | Media |
| 2 | NO-Hardhat | EPP ausente | **Alta** |
| 3 | NO-Mask | EPP ausente | **Alta** |
| 4 | NO-Safety Vest | EPP ausente | **Alta** |
| 5 | Person | Entidad | Media |
| 6 | Safety Cone | Señalización | Baja |
| 7 | Safety Vest | EPP presente | Media |
| 8 | machinery | Entidad | **Alta** |
| 9 | vehicle | Entidad | Media |
