# Conciliación de pallets ESPI

App de Streamlit que cruza los pallets registrados en el archivo de
**LIQUIDACIÓN** (una hoja por productor/lote) contra los manifiestos del
archivo **RCF LIQUIDACIONES**, y devuelve el mismo archivo de LIQUIDACIÓN con:

- Cada pallet marcado en verde (encontrado y coincide), amarillo (encontrado
  pero con diferencia de calibre/cajas) o rojo (no encontrado en ningún
  manifiesto) — con un comentario en la celda indicando en qué manifiesto se
  encontró.
- Una tabla nueva de **"CONCILIACIÓN AUTOMÁTICA"** al final de cada hoja, con
  el total de pallets y cajas por calibre, cuántos se confirmaron, cuántos
  faltan (con su número de pallet) y cuántos "sobran" (pallets del RCF que
  pertenecen a ese lote/productor pero no aparecen listados en la hoja).

No toca ninguna fórmula ni tabla existente del archivo original — solo agrega
las marcas de color/comentarios y la tabla nueva al final de cada hoja.

## Cómo correrla

```bash
pip install -r requirements.txt
streamlit run app.py
```

Se abre en el navegador. Sube ahí los dos archivos `.xlsx` (LIQUIDACIÓN y
RCF), presiona "Ejecutar conciliación" y descarga el Excel resultante.

## Cómo funciona el cruce (por si necesitas ajustarlo)

1. **LOTE como identificador principal**: cada hoja de LIQUIDACIÓN declara un
   número de lote (celda "LOTE:"); si esa celda trae texto en vez de número
   (ej. "LOTE ISSAC LOPEZ"), se usa el número del nombre de la pestaña como
   respaldo (ej. la pestaña "8.1" → lote 8).
2. **Nombre del productor como respaldo**: como un mismo número de lote puede
   estar compartido por varios productores (ocurre con el lote 4 y el lote 8
   en los archivos de ejemplo), el cruce siempre compara también el nombre del
   productor (tolerando variantes/errores de escritura como "RAFAEL B." vs
   "RAFAEL BALDERRAMA") para no confundir pallets de un productor con los de
   otro que comparte el mismo lote.
3. **Pallet + lote como llave de búsqueda**: un mismo # de pallet físico
   puede repartirse entre dos lotes distintos en el RCF (pallets mixtos), así
   que nunca se busca solo por número de pallet.
4. **Cajas/calibre faltantes en la hoja no cuentan como diferencia**: algunas
   hojas de LIQUIDACIÓN solo registran el # de pallet y el calibre, sin cajas
   por pallet (asumen la caja estándar del lote); en ese caso el pallet se
   marca como encontrado si el número existe en el RCF, sin exigir que
   coincida un dato que la hoja nunca capturó.
5. **Deduplicación del RCF**: el archivo RCF trae una pestaña maestra que
   repite todo lo que ya está en las pestañas individuales de cada
   manifiesto — se deduplica automáticamente para no inflar los conteos.

## Archivos

- `core.py` — toda la lógica de lectura, cruce y escritura (sin interfaz).
- `app.py` — interfaz Streamlit.
- `requirements.txt` — dependencias.

## Qué revisar en la próxima temporada / con más pestañas

- Si aparecen nuevas pestañas de manifiesto en el RCF con encabezados muy
  distintos a "FECHA / # PALLET / LOTE / CAJAS / CALIBRE / PRODUCTOR /
  MANIFIESTO", la app las reporta como "omitidas" en vez de fallar — conviene
  revisar esa lista después de cada corrida.
- El umbral de similitud de nombres de productor está en `core.py`
  (`PRODUCTOR_MATCH_THRESHOLD = 0.45`) — si ves cruces raros entre
  productores con nombres parecidos, se puede subir ese número.
