# Nihongo Static Assets CDN

Repositorio de contenido estatico para la app NihonGO.

Su objetivo es permitir actualizaciones de contenido en caliente (hotfix de datos) sin necesidad de publicar una nueva version de la app, siempre que no cambie la logica de la aplicacion ni el esquema de datos.

## Que contiene este repo

- Archivos JSON de gramatica por leccion.
- JSON de vocabulario.
- JSON de kanji.
- JSON de silabarios.
- Un archivo maestro `manifest.json` con la lista de assets y sus rutas remotas.

## Estructura actual

```text
main/
  n5/
    gramatica/
      c1-l0-v10.json
      c1-l1-v10.json
      ...
      c1-l12-v10.json
    kanji/
      kanji.json
    vocabulario/
      vocabulario.json
    silabarios/
      silabarios.json
manifest.json
```

## Rol del manifest

`manifest.json` es la fuente de verdad para el cliente.

El cliente debe:

1. Descargar el manifest remoto.
2. Compararlo contra el manifest local aplicado.
3. Detectar que entries han cambiado.
4. Descargar solo los archivos cambiados.
5. Validar JSON y reemplazar de forma atomica.

Esto minimiza trafico, reduce tiempo de sync y evita descargar contenido que no cambio.

## Formato del manifest

Ejemplo simplificado:

```json
{
  "manifest_version": 2,
  "files": [
    {
      "nombre": "leccion 3.json",
      "ruta": "main/n5/gramatica/lecciones/leccion 3.json",
      "version": "1.0"
    },
    {
      "nombre": "vocabulario.json",
      "ruta": "main/n5/vocabulario/vocabulario.json",
      "version": "1.0"
    },
    {
      "nombre": "kanji.json",
      "ruta": "main/n5/kanji/kanji.json",
      "version": "1.0"
    },
    {
      "nombre": "silabarios.json",
      "ruta": "main/n5/silabarios/silabarios.json",
      "version": "1.0"
    }
  ]
}
```

## Convencion de versionado de contenido

- Cambios pequenos o medianos de contenido: subir nuevo JSON y actualizar su entry en `manifest.json`.
- Cambios masivos (por ejemplo nuevo curso con cambios estructurales): evaluar release nueva de app.

Recomendacion:

- Mantener `nombre` estable cuando sea posible.
- Versionar en `ruta` cuando haga falta invalidacion clara de cache.
- Usar `version` para detectar diferencias entre manifest local y remoto y decidir que asset descargar.

## Flujo de trabajo recomendado

1. Editar y auditar JSON fuente.
2. Copiar al path final del repo (`main/...`).
3. Actualizar `manifest.json` con rutas correctas.
4. Validar que todos los JSON parsean correctamente.
5. Commit + push.

## Criterios para lanzar nueva version de app

Lanzar nueva version de app cuando:

- Cambia codigo de negocio o de red.
- Cambia el schema local de datos.
- Hay nuevas pantallas o navegacion.
- Se introduce contenido que requiere nueva logica cliente.

No hace falta nueva app version cuando:

- Solo corriges textos, ejemplos, lecciones, vocabulario o kanji ya soportados por la app actual.

## Notas de robustez para cliente

- Aplicar timeout y retries con backoff al descargar.
- Usar reemplazo atomico de archivos locales.
- Mantener fallback al ultimo contenido valido local.
- Registrar log de que assets se actualizaron y cuando.
