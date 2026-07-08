# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) al trabajar con el código de este repositorio.

## Qué es esto

Un tema de Shopify **Ella** personalizado (de Halothemes, v6.7.2 — ver `theme_info` en `config/settings_schema.json`) para la tienda **levnicare**. No hay paso de compilación (build), gestor de paquetes ni suite de pruebas — es un tema de Shopify puro que se edita directamente y se despliega con la CLI de Shopify. La tienda es principalmente **en español**; el texto de interfaz escrito a medida se redacta directamente en español en lugar de mediante claves de traducción (locale keys).

## Comandos

Todo el desarrollo pasa por la CLI de Shopify (`shopify theme ...`), ejecutada desde la raíz del repositorio:

- `shopify theme dev` — servidor de desarrollo local con recarga en caliente contra una tienda de desarrollo
- `shopify theme push` / `shopify theme pull` — sincronizar con la tienda conectada
- `shopify theme check` — linter de Liquid/tema (Theme Check); la única "prueba" disponible. No hay archivo de configuración en el repositorio, así que se ejecuta con los valores por defecto.

No hay herramientas de lint/formato para los assets de JS/CSS — se editan y se suben tal cual.

## Arquitectura

Estructura de directorios estándar de un tema de Shopify (`layout/`, `sections/`, `snippets/`, `templates/`, `assets/`, `config/`, `locales/`). Conviven dos capas que deben tratarse de forma distinta:

### Capa del proveedor (framework Ella / "halo") — modificar con cuidado

- **`assets/theme.js`** es el núcleo del tema: un único objeto global `halo` (basado en jQuery, IIFE) con `halo.init()` / `halo.ready()` que orquesta casi toda la interactividad — carrito AJAX, quick view/shop, wishlist, comparación, moneda, sliders, mega menú, etc. La mayoría de los snippets `halo-*.liquid` y los assets `halo.*.js` / `halo-*.js` pertenecen a este framework.
- **`layout/theme.liquid`** construye un gran conjunto de clases de `<body>` a partir de `settings.*` (modo de layout, estilos de flechas/puntos, layout de tarjeta de producto 01–08, tipo de quick-shop, RTL). El CSS depende de estas clases, así que el comportamiento está gobernado por la configuración del tema, no solo por el marcado (markup).
- **`snippets/variable.liquid`** genera el CSS de `@font-face` / fuentes a partir de la configuración; **`snippets/global-script.liquid`** genera los globales `window.routes`, `window.variantStrings`, `window.cartStrings`, etc., además de los SVG de `window.arrows` que lee el JS. Al agregar una cadena de JS que deba traducirse, añádela aquí mediante una clave de traducción `'...' | t` en lugar de escribirla directamente (hardcode).
- **`snippets/global-style.liquid`** + los múltiples archivos `assets/component-*.css`: cada componente trae su propia hoja de estilos, cargada por sección con `{{ 'component-x.css' | asset_url | stylesheet_tag }}`.
- Las plantillas que terminan en `.liquid` dentro de `templates/` con prefijo de tipo de objeto y `ajax_` (p. ej. `product.ajax_quick_view.liquid`, `collection.ajax_product_card.liquid`, `cart.ajax_side_cart.liquid`) son **fragmentos de respuesta AJAX** renderizados por el JS de halo, no plantillas de página normales. Hay ~23 de estas.

### Capa personalizada (específica de la tienda) — se puede editar con libertad

Las secciones a medida del dueño de la tienda se añaden sobre Ella y es donde ocurre la mayor parte del trabajo activo (ver el historial reciente de git: `product-comparison-table`, `image-comparison`, `product-features-highlight`, `spotlight-block`, `whatsapp-button`, `multilayer-image`, `banner-reveal-hover`, `product-hover-cards`, `custom-*`).

Reconócelas por sus convenciones:
- **Secciones autocontenidas**: bloque `{%- style -%}` en línea gobernado por `--css-var: {{ section.settings.x }}`, con alcance limitado a una clase `...-id-{{ section.id }}` para que múltiples instancias no colisionen, más un archivo `component-*.css` dedicado.
- **Prefijos de clase distintivos y no propios de Ella** (p. ej. `comp-v3-` en `product-comparison-table.liquid`) — marcan el marcado escrito por la tienda.
- **Plantillas de producto personalizadas** con nombres en español: `product.plantilla-basica.json`, `product.plantilla-oferta-vip.json`, `product.cabezal-runa.json`, `product.producto-nexo.json`. Las plantillas JSON conectan secciones + bloques + configuración; la sección `.liquid` correspondiente contiene el marcado.

## Convenciones

- **Editar una sección = editar el `.liquid` de la sección + su `component-*.css` en `assets/`** (y su `.js` si es interactiva). El CSS no se empaqueta (bundle), así que una nueva hoja de estilos debe referenciarse con `stylesheet_tag` desde la sección que la usa.
- **Limita el alcance del CSS personalizado a la instancia de la sección** mediante `section.id` para evitar que los estilos se filtren a otras partes de la página.
- **Las claves de traducción** viven en `locales/*.json` (cadenas de interfaz) y `locales/*.schema.json` (etiquetas del editor de temas). Ella usa claves `t:...` en todos sus schemas; las nuevas secciones personalizadas de este repositorio a menudo se saltan esto y escriben el español directamente — sigue el patrón que ya use el archivo que estés editando.
- **`assets/custom.css` y `assets/custom.js`** son la vía de escape prevista para ajustes puntuales de estilo/comportamiento (custom.js solo se carga cuando `settings.use_custom_js` está activo).
- Como el framework es jQuery + un objeto global, el nuevo comportamiento interactivo generalmente se engancha a `halo` o escucha sus eventos, en lugar de introducir un framework aparte.
