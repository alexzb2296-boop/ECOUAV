# IMPERIAL — Plan de unificación funcional

Fecha de auditoría: 2026-10-11

## Arquitectura acordada

- **Base principal:** `desktop-v2.html` (diseño de tres columnas).
- **Origen de funcionalidades maduras:** `index.html` (versión adaptable/antigua).
- **Entrada de escritorio:** `escritorio.html` (visor que carga `desktop-v2.html`).
- **Meta:** una única experiencia funcional y adaptable, sin bifurcar la lógica para móvil y escritorio.
- **Migración gradual:** conservar `index.html` operativo hasta validar la paridad funcional. No sustituir ambos documentos por redirección todavía.

## Inventario inicial observado en código

| Área | Versión adaptable `index.html` | Escritorio `desktop-v2.html` | Próximo paso |
|---|---|---|---|
| Mapas satélite, relieve y capas | Sí | Sí, parcialmente | Contrastar opciones y fuentes |
| Búsqueda de coordenadas | Sí | Sí, básica | Completar formatos |
| Conversión UTM y MGRS | Sí | Sin implementación identificada | Migrar |
| Medición de distancia, azimut, radio y área | Sí | Herramientas más limitadas | Migrar |
| Cuadrículas del mapa y de zona | Sí | Sin controles equivalentes identificados | Migrar |
| Perfil A/B y perfil de ruta | Sí | Sin implementación identificada | Migrar |
| Análisis de desnivel de ruta | Sí | Sin implementación identificada | Migrar |
| Importación DEM GeoTIFF y recuperación | Sí | Sin implementación identificada | Migrar con validación y límites |
| Visor 3D | Sí | Sí, básico/demostrativo | Comparar y unificar |
| Meteorología Open-Meteo | Sí | Sí | Consolidar |
| Herramientas de antenas y LOS | Sí, más completas | Sí, parcialmente | Unificar con aviso de límites del modelo |
| POI, rutas y deshacer | Sí | Sí | Comparar casos de uso |
| GeoJSON | Sí | Sí | Verificar equivalencia de datos |
| Exportación KML/GPX | Sí | Sin controles equivalentes identificados | Migrar |
| Proyectos locales / misiones | Sí | Sí, parcialmente | Definir formato compartido y persistencia |

> Los estados reflejan inspección del código y los controles disponibles, **no pruebas completas de funcionamiento en navegador**. El relieve IGN del archivo adaptable presenta una incidencia previamente identificada y se mantiene pendiente.

## Fases de implementación

1. **Fundamentos:** extraer módulos compartidos sin romper el diseño de escritorio; definir estado cartográfico común, persistencia y gestión de errores.
2. **Cartografía:** coordenadas, conversiones, cuadrículas, mediciones y gestión de capas.
3. **Terreno:** perfiles A/B, perfiles y desniveles de ruta, DEM importado y visor 3D.
4. **Meteorología y antenas:** consolidar herramientas y sus limitaciones declaradas; diferenciar visualización ilustrativa de estimaciones verificables.
5. **Misiones:** POI, rutas, proyectos, importación/exportación GeoJSON, KML y GPX.
6. **Adaptabilidad y pruebas:** móvil/tablet desde el mismo núcleo, controles de navegación, rendimiento y pruebas de regresión.

## Criterios de aceptación

- No perder ningún control útil que esté operativo en la versión adaptable.
- Preservar diseño principal de tres columnas en escritorio.
- Mantener funcionalidad del mapa y proyectos en cada fase.
- Validar sintaxis JS, interacción cartográfica y formatos importados/exportados.
- No afirmar que la visualización de antenas es predicción RF real ni que las indicaciones sirven como autorización de vuelo.
- Retirar la versión antigua solo después de verificar paridad funcional con casos de prueba.

## Progreso — bloque cartografía 1 (2026-10-11)

- Integrado en `desktop-v2.html`: consulta WGS84 decimal, presentación grados/minutos/segundos, conversión WGS84→UTM (incluidas zonas especiales de Noruega/Svalbard, ámbito 80°S–84°N), copiar coordenadas, ir a coordenadas.
- Integrada medición de dos puntos A/B con distancia geodésica Leaflet y azimut inicial respecto al norte verdadero; trazado y botón de limpieza.
- Actualizado el iframe de `escritorio.html` para cargar el nuevo código.
- **Verificado:** sintaxis de JavaScript tras el cambio. **Pendiente:** validación visual y funcional en navegador, comprobación numérica de la transformación UTM frente a muestras de referencia; MGRS, radio, área y cuadrículas siguen pendientes.

## Progreso — bloque cartografía 2 (2026-10-11)

- Escritorio: conversión WGS84 → MGRS mediante `mgrs@2.1.0` y búsqueda/centrado desde una cadena MGRS.
- Medición de radio con círculo cartográfico Leaflet y cálculo de superficie circular aproximado; límite configurable 1–100000 m.
- Delimitación de polígonos por vértices (máximo 150), cierre explícito y superficie esférica aproximada.
- Cuadrícula de referencia visual con pasos de 100, 250, 500, 1000 y 5000 m; supresión de mallas densas y actualización al mover el mapa. **Nota:** cuadrícula visual aproximada en latitud/longitud, no una malla UTM reglamentaria ni cuadrícula de zona con celdas geodésicas exactas.
- Se mantuvieron las capas de ruta/POI/antena y se aislaron las figuras en capas propias; actualizado `escritorio.html`.
- Verificada sintaxis JavaScript; sin pruebas integrales de ejecución y sin validación numérica contra datos de control. Quedan pendientes cuadrícula UTM de zona real, test de librería externa y controles móviles.

## Progreso — perfiles de terreno A/B y ruta (2026-10-11)

- Integrado en `desktop-v2.html` panel **Perfiles y desniveles** con selección de A/B sobre el mapa, marcadores arrastrables y limpieza independiente.
- Incluida consulta de 64 cotas interpoladas de Open-Meteo Elevation para A/B y 72 para rutas existentes (dibujadas o importadas), con consultas agrupadas mediante el cliente de elevaciones ya existente.
- Los gráficos SVG presentan cotas mínimas y máximas, distancia, ascenso acumulado, descenso acumulado, pendiente neta y pendiente máxima muestreada.
- Protección frente a respuestas asíncronas obsoletas cuando se modifican los puntos o las rutas. Los cálculos no tienen exactitud certificada y se muestran como aproximados.
- Modo de marcado A/B excluyente con otras herramientas activas y actualización de `escritorio.html` para cargar la versión revisada.
- **Comprobación realizada:** sintaxis JavaScript. **Pendiente:** prueba funcional de interfaz real y validación numérica de datos; migración de DEM GeoTIFF local, contornos y demás funciones avanzadas de terreno.
