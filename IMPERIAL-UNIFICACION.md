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

## Progreso — DEM GeoTIFF y curvas de nivel (2026-10-11)

- Añadido panel de **DEM GeoTIFF y curvas** a `desktop-v2.html` sin retirar funciones del mapa.
- Importación de GeoTIFF georreferenciado **EPSG:4326** (hasta 60 MB). Se toma la primera banda, se remuestrea para mostrarla (máximo aproximado 640 px en su mayor dimensión), se representa como imagen superpuesta con opacidad configurable y se encuadra el mapa.
- Soporte para mostrar/ocultar el DEM, eliminarlo y conservar/restaurar el archivo mediante **IndexedDB** en el propio navegador.
- Curvas aproximadas por *marching squares* sobre cuadrícula limitada de 18 × 18 puntos en la vista actual. Se utilizan muestras del **DEM local** cuando está cargado y, en caso contrario, el servicio **Open-Meteo Elevation**. Intervalos configurables: 10, 20, 50 y 100 m.
- Se limita el número de niveles y se invalida el cálculo anterior al ocultar/limpiar o cambiar la configuración; se mantiene separado el resto de capas y controles.
- Lanzador `escritorio.html` actualizado para cargar la nueva versión.
- **Verificación realizada:** compilación de la sintaxis JavaScript con `new Function` y comprobación de presencia de controles. **Pendiente:** validación funcional real en navegador, tratamiento de más proyecciones y bandas, precisión geográfica/estadística de las curvas, nodata y compatibilidad de archivos comprimidos. No se afirma certificación cartográfica. La incidencia IGN antigua permanece pendiente.

## Progreso — meteorología horaria y proyectos locales (2026-10-11)

- Meteorología de escritorio: se conserva el resumen actual de **Open-Meteo** y se añade una previsión para las próximas **seis horas** (temperatura, velocidad y dirección del viento, rachas y probabilidad de precipitación). Se distingue previsión de observaciones reales.
- Proyectos de escritorio: guardar, seleccionar, recuperar y eliminar sesiones **localmente** mediante `localStorage` del navegador, sin nube. Se guardan nombre, fecha, puntos de ruta, POI, antena, mapa base, centro, zoom y estado de demostración; límite de 15 proyectos. Se validan las coordenadas y el tamaño de los datos antes de restaurarlos.
- No se incluyen DEM locales ni pronósticos meteorológicos en los proyectos; la biblioteca DEM usa almacenamiento separado en IndexedDB. Los proyectos de la versión antigua no se convierten automáticamente todavía.
- Actualizado el lanzador `escritorio.html` y verificada la sintaxis JavaScript.
- **Pendiente:** pruebas manuales reales en el navegador (guardar/recargar, cuota de almacenamiento, respuesta horaria de Open-Meteo), compatibilidad import/export de proyectos con la versión antigua y adaptación móvil/tablet desde la misma aplicación.

## Progreso — interfaz única adaptable (2026-10-11)

- El núcleo funcional **`desktop-v2.html`**, previamente diseñado para tres columnas, ahora utiliza `width=device-width` y puntos de ruptura para escritorio (>1100 px), tablet (721–1100 px) y móvil (≤720 px).
- En tablet y móvil, las columnas de herramientas y datos se transforman en paneles laterales accesibles desde una barra inferior de tres acciones: **Herramientas, Mapa y Datos**; se añaden cierre por fondo y tecla Escape, y actualización del tamaño de Leaflet ante cambios de ventana o panel.
- Se compactan buscador, cabecera, visor 3D, leyendas y controles de mapa para teléfonos sin duplicar el código cartográfico.
- Nueva URL de entrada para la aplicación unificada: **`imperial.html`**, que abre el mismo motor `desktop-v2.html`. **`index.html` legado permanece operativo temporalmente** por seguridad de la migración; todavía no se ha redirigido el dominio raíz.
- Actualizado el lanzador `escritorio.html` para refrescar el contenido del iframe en el ordenador.
- Verificada sintaxis de JavaScript; **pendiente prueba de interacción en navegadores reales** (incluidas orientaciones móviles, Safari iOS, capas, flujos de importación y guardado) antes de retirar el legado o declarar equivalencia completa.

## Revisión de navegación adaptable (2026-10-11)

- El despliegue **#117** (primera interfaz adaptable) terminó con resultado **success** en GitHub Actions.
- Mejoras posteriores: al seleccionar una herramienta que requiere marcar puntos en el mapa desde un panel móvil, se cierra ese panel para poder interactuar con el mapa; el fondo de cierre usa posicionamiento fijo y los paneles admiten desplazamiento táctil.
- El lanzador de zoom `escritorio.html` ahora muestra «Aplicación» con enlace a `imperial.html`, en lugar de remitir a la versión móvil heredada.
- Revisión estática del último `desktop-v2.html`: **JavaScript sintácticamente válido**, **133 IDs HTML únicos**, **110 referencias `$('id')` comprobadas, ninguna ausente**, y presencia de controles de navegación móvil.
- **Límite de esta revisión:** no se ha podido abrir la aplicación desplegada con un navegador de prueba desde este entorno, por lo que **no está verificado** el comportamiento visual y táctil real. El despliegue posterior (#119) estaba en curso en la última consulta.
- Por este motivo **no se redirige `index.html` ni se elimina la interfaz legada**. Antes de fijar IMPERIAL como página inicial definitiva hay que pasar las pruebas manuales en móvil y tablet.

## Página principal unificada y sesiones portátiles (2026-10-11)

- Después de la confirmación del usuario de que el comportamiento observado iba bien, se ha hecho `index.html` la entrada principal a la aplicación adaptable `desktop-v2.html` (redirección inmediata, preservando parámetros y ancla; `?classic=1` permite abrir el legado).
- **Copia íntegra y recuperable** de la página antigua: `legacy-index.html` (203710 caracteres, idéntica a la versión anterior en el momento del respaldo). La navegación «Clásico» del escritorio apunta allí.
- Se ha integrado **importación y exportación de sesiones de proyectos IMPERIAL en JSON**, con comprobación de versión, límites de archivo y validación de coordenadas. No equivale al formato GeoJSON que permanece disponible por separado.
- Controles existentes y CSS adaptable preservados. Revisión estática: sintaxis JS correcta, ningún ID HTML duplicado, ninguna referencia `$('id')` sin nodo correspondiente.
- **Pendiente:** publicación GitHub Pages de los últimos commits y prueba completa en navegador y dispositivos. Se mantiene el respaldo accesible para regresar al sistema anterior si hiciera falta.
