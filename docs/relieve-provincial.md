# Relieve provincial complementario

## Resultado y alcance

Castellón, Valencia y Alicante tienen dos PMTiles v3 adicionales por provincia en `releases/maps/`: sombreado raster PNG (z9–13) y curvas MVT/PBF gzip (z9–14). Los mapas OSM Shortbread originales no se alteran. Las curvas son cada 50 m y tienen `elevation_m`, `major_100` y `major_200`, tal como esperan Topográfico y Combinado. Senderos conserva solo la cartografía vectorial. Los paquetes se asignan a las mismas teselas completas que la base provincial; al instalar las tres provincias no se duplican teselas en los límites. Una sola provincia puede presentar la franja fronteriza ya documentada en [mapas-provinciales-v21.md](mapas-provinciales-v21.md).

| Provincia | Sombreado | Curvas | Total adicional |
| --- | ---: | ---: | ---: |
| Castellón | 10,4 MiB · 762 teselas | 6,5 MiB · 2732 teselas | 16,9 MiB |
| Valencia | 13,6 MiB · 1158 teselas | 7,2 MiB · 3950 teselas | 20,8 MiB |
| Alicante | 6,9 MiB · 636 teselas | 3,8 MiB · 2207 teselas | 10,7 MiB |

Los tamaños exactos, SHA-256, número de teselas y dependencia del PMTiles vectorial figuran en `releases/maps/relief-manifest.json`. Los tres componentes de una provincia (OSM, sombreado y curvas) forman un **grupo lógico** en el catálogo local. El manifiesto no contiene URL ni habilita descargas. En el futuro, una única acción podrá descargar/verificar/instalar el grupo completo de forma transaccional, previa decisión sobre alojamiento y costes. Por ahora se importan los tres archivos manualmente y de forma independiente.

## Datos, calidad y derechos

Origen: [Copernicus DEM GLO-90, publicación AWS Open Data](https://registry.opendata.aws/copernicus-dem/), versión 2021, COG de 1° por 1° (1200 × 1200 muestras). Se reutilizan los seis COG ya empleados en Tavernes y se descargaron cuatro nuevos para N37 y N40. El manifiesto conserva las huellas de los diez COG. Es un **modelo digital de superficie** de unos 90 m: incluye vegetación y construcciones, no ofrece la precisión de un MDT de terreno ni permite deducir la seguridad o transitabilidad de una senda. Las curvas de 50 m son una representación cartográfica aproximada del modelo, no mediciones de campo. No se usan para la altitud GPS.

La [licencia específica GLO-90](https://dataspace.copernicus.eu/sites/default/files/media/files/2025-06/copernicus_contributing_mission_data_access_v2_cop_dem_licenses.pdf) permite reproducción, modificación y distribución gratuita, también comercial, con atribución, aviso de ausencia de responsabilidad y sin aparentar respaldo oficial. La atribución completa y el aviso están en metadatos de cada PMTiles, en Información de la app y en `releases/maps/LEEME.md`. La licencia ODbL de la base OSM continúa separada. [CNIG MDT05](https://centrodedescargas.cnig.es/CentroDescargas/home) permitiría un relieve de terreno de mayor resolución, pero requiere otro volumen de datos y transformación; no se mezcló con GLO-90 ni se cambió el motor en este incremento.

## Producción reproducible

Con Python 3 y las librerías locales de `tools/python`:

```powershell
$env:PYTHONPATH='tools/python'
py -3 tools/build_provincial_relief.py castellon
py -3 tools/build_provincial_relief.py valencia
py -3 tools/build_provincial_relief.py alicante
py -3 tools/build_relief_manifest.py
py -3 tools/build_province_catalog.py
py -3 tests/provincial_relief_verify.py
py -3 tools/benchmark_relief.py
```

El generador solo descarga COG ausentes desde el bucket público de Copernicus. No sobrescribe paquetes existentes: para reproducir byte a byte, utilizar una carpeta de trabajo aislada o retirar explícitamente los PMTiles de relieve generados. Calcula la iluminación una vez sobre el mosaico continuo y después corta teselas, evitando costuras por gradiente por tesela. Las curvas se extraen por ventana DEM con borde de muestra y se codifican en la capa `contours`. El esquema y los colores de los estilos originales no cambian; la app clona sus capas de relieve para cada provincia instalada y sustituye Tavernes por Valencia cuando su componente está presente. Si falta el componente, no crea alturas ni sombras de relleno.

## Verificación y límites

`tests/provincial_relief_verify.py` abrió y decodificó seis PMTiles, comprobó SHA-256, tipos PNG/MVT, atributos de curvas, coincidencia de teselas de sombreado con la base, ausencia de duplicados entre provincias y muestra no uniforme de sombreado en Segorbe. El recorrido completo de esa verificación tardó aproximadamente 1,1 s en este PC. Una muestra aleatoria y reproducible de 300 teselas (50 de cada componente) se leyó y decodificó en 1,54 s en este PC. **Estas cifras no miden renderizado ni consumo en Android.** La aplicación compila y sus pruebas automáticas se ejecutan por separado. Quedan pendientes la inspección visual de costuras y etiquetas en un Android real, la fluidez con los tres paquetes instalados, el consumo de memoria/batería y la legibilidad exterior. El emulador local no arrancó porque este equipo carece del controlador de aceleración Android.

## Pendiente para distribución

Elegir alojamiento autorizado y coste de transferencia, publicar metadatos firmados o verificados, implementar descarga transaccional del grupo por provincia (con recuperación de interrupciones, espacio y red), atribución visible y ensayos físicos de rendimiento. Ninguna de estas funciones de red está activa ahora.

## Evolución posterior: descarga preparada

La APK posterior incorpora un catálogo local verificado `catalog-maps-v0.1.0.json` y un instalador que descarga los tres componentes a un directorio temporal, reanuda archivos parciales por HTTP Range, compara tamaños y SHA-256, y solo entonces activa el conjunto por un cambio de directorio. Una actualización fallida conserva la versión anterior. Los archivos manuales siguen funcionando. GitHub Releases es el primer proveedor; el catálogo mantiene las URLs fuera del código del gestor para migrar después a R2. El borrador de Release no sirve archivos públicamente: la prueba de descarga real y el rendimiento Android quedan pendientes de su publicación y prueba física. Publicar y modificar el repositorio remoto son acciones reservadas al propietario.
