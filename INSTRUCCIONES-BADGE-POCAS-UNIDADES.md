# Cómo subir el badge "Pocas unidades" manualmente

Ve a: **Shopify Admin → Tienda online → Temas → (tu tema en vivo) → ⋯ Acciones → Editar código**

Son 5 archivos. Haz cada uno y pulsa **Guardar** después de cada cambio.

---

## ARCHIVO 1 — `snippets/product-badge.liquid`  (REEMPLAZAR TODO)

Borra todo el contenido del archivo y pega exactamente esto:

```liquid
{%- liquid
    assign position = settings.badge_postion

    if settings.show_sale_badge
        assign sale_badge = false
        assign sale_badge_type = settings.sale_badge_type
        if badge_detail
            assign current_variant = product.selected_or_first_available_variant
            if current_variant.compare_at_price > current_variant.price
                assign sale_badge = true
            endif
        else
            if product.compare_at_price_min > product.price_min
                assign sale_badge = true
            endif
        endif
    endif

    if settings.show_sold_out_badge
        assign sold_out_badge = false
        if product.available == false
            assign sold_out_badge = true
        endif
    endif

    if settings.show_custom_badge
        assign custom_badge = false
        assign custom_badge_text = product.metafields.custom.custom_badge

        for tag in product.tags
            assign tag_handle = tag | handle
            if tag_handle == 'label'
                assign custom_badge = true
            endif
        endfor

        if custom_badge_text
            assign custom_badge = true
        endif
    endif

    if settings.show_bundle_badge
        assign bundle_badge = false
        if product.metafields.c_f.grouped_sub_product and settings.show_bundle_badge
            assign bundle_badge = true
        endif
    endif

    if settings.show_low_stock_badge
        assign low_stock_badge = false
        assign low_stock_limit = settings.low_stock_badge_limit | default: 5
        if badge_detail
            assign current_variant = product.selected_or_first_available_variant
            if current_variant.inventory_management == 'shopify' and current_variant.available and current_variant.inventory_quantity > 0 and current_variant.inventory_quantity <= low_stock_limit
                assign low_stock_badge = true
            endif
        else
            if product.available
                assign product_inventory = 0
                assign inventory_tracked = false
                for variant in product.variants
                    if variant.inventory_management == 'shopify'
                        assign inventory_tracked = true
                        if variant.inventory_quantity > 0
                            assign product_inventory = product_inventory | plus: variant.inventory_quantity
                        endif
                    endif
                endfor
                if inventory_tracked and product_inventory > 0 and product_inventory <= low_stock_limit
                    assign low_stock_badge = true
                endif
            endif
        endif
    endif

    if settings.show_new_badge
        assign new_badge = false
        assign new_badge_type = settings.new_badge_type
        if new_badge_type == 'auto'
            assign new_badge = true
            assign date_published = product.published_at | date:'%s'
            assign date_now = 'now' | date:'%s'
            assign date_minus = date_now | minus: date_published
            assign date_difference = date_minus | divided_by: 86400
            assign new_badge_limit = settings.new_badge_limit
            assign new_badge_time = settings.new_badge_time
        else
            for tag in product.tags
            assign tag_handle = tag | handle
                if tag_handle == 'new'
                    assign new_badge = true
                endif
            endfor
        endif
    endif
-%}
{%- if sale_badge or sold_out_badge or custom_badge or bundle_badge or new_badge or low_stock_badge -%}
    <div class="{{ class }}{% if sale_badge_type == 'discount' or sale_badge_type == 'text_discount' %} has-badge-js{% endif %} badge-{{ position }} halo-productBadges halo-productBadges--{{ position }} date-{{ date_minus }} date1-{{ date_difference }}{% unless sale_badge %} sale_badge_disable{% endunless %}"
        {% if sale_badge_type == 'text_discount' or sale_badge_type == 'text' %}data-text-sale-badge="{{ 'products.product.on_sale' | t }}"{% endif %}
        {% if sale_badge_type == 'discount' %}data-text-sale-badge="- "{% endif %}
        data-new-badge-number="{{ new_badge_limit }}"
    >
        {%- if new_badge -%}
            {%- if new_badge_type == 'auto' -%}
                {%- if date_difference < new_badge_time -%}
                    {%- if serial <= new_badge_limit -%}
                        <span class="badge new-badge" aria-hidden="true">
                            {{ 'products.product.new' | t }}
                        </span>
                    {%- endif -%}
                {%- endif -%}
            {%- else -%}
                <span class="badge new-badge" aria-hidden="true">
                    {{ 'products.product.new' | t }}
                </span>
            {%- endif -%}
        {%- endif -%}
        {%- if sale_badge -%}
            <span class="badge sale-badge" aria-hidden="true">
                {%- if sale_badge_type == 'discount' -%}
                    {%- if badge_detail -%}
                        -{{ current_variant.compare_at_price | minus: current_variant.price | times: 100.0 | divided_by: current_variant.compare_at_price | round }}%
                    {%- else -%}
                        {% liquid
                            assign list_compare = product.variants | where: 'compare_at_price'
                            assign compare = 0
                            for variant in list_compare
                                assign saving = variant.compare_at_price | minus: variant.price | times: 100.0 | divided_by: variant.compare_at_price | round
                                if saving > compare
                                    assign compare = saving
                                endif
                            endfor
                            if compare  < 1
                                assign compare = product.compare_at_price_min | minus: product.price_min | times: 100.0 | divided_by: product.compare_at_price_min | round
                            endif
                        %}
                        -{{ compare | append: '%'}}
                    {%- endif -%}
                {%- elsif sale_badge_type == 'text_discount' -%}
                    {{ 'products.product.on_sale' | t }}
                    {%- if badge_detail -%}
                        {{ current_variant.compare_at_price | minus: current_variant.price | times: 100.0 | divided_by: current_variant.compare_at_price | round }}%
                    {%- else -%}
                        {% liquid
                            assign list_compare = product.variants | where: 'compare_at_price'
                            assign compare = 0
                            for variant in list_compare
                                assign saving = variant.compare_at_price | minus: variant.price | times: 100.0 | divided_by: variant.compare_at_price | round
                                if saving > compare
                                    assign compare = saving
                                endif
                            endfor
                            if compare  < 1
                                assign compare = product.compare_at_price_min | minus: product.price_min | times: 100.0 | divided_by: product.compare_at_price_min | round
                            endif
                        %}
                        {{ compare | append: '%'}}
                    {%- endif -%}
                {%- else -%}
                    {{ 'products.product.on_sale' | t }}
                {%- endif -%}
            </span>
        {%- endif -%}
        {%- if sold_out_badge -%}
            <span class="badge sold-out-badge" aria-hidden="true">
                {{ 'products.product.sold_out' | t }}
            </span>
        {%- endif -%}
        {%- if custom_badge -%}
            <span class="badge custom-badge" aria-hidden="true">
                {%- if custom_badge_text -%}
                    {{ custom_badge_text }}
                {%- else -%}
                    {{ 'products.product.custom' | t }}
                {%- endif -%}
            </span>
        {%- endif -%}
        {%- if bundle_badge -%}
            <span class="badge bundle-badge" aria-hidden="true">
                {{ 'products.product.bundle' | t }}
            </span>
        {%- endif -%}
        {%- if low_stock_badge -%}
            <span class="badge low-stock-badge" aria-hidden="true">
                {{ 'products.product.low_stock' | t }}
            </span>
        {%- endif -%}
    </div>
{%- endif -%}
```

---

## ARCHIVO 2 — `assets/component-badge.css`  (AGREGAR AL FINAL)

Ve al final del archivo y pega este bloque:

```css
.halo-productBadges .badge.low-stock-badge{
    color: var(--low-stock-badge-color, #ffffff);
    background-color: var(--low-stock-badge-bg, #e07a1f);
}
```

---

## ARCHIVO 3 — `locales/es.json`

Con Ctrl+F busca:  `"bundle":"Manojo"`

Reemplaza:
```
"bundle":"Manojo","unavailable":
```
Por:
```
"bundle":"Manojo","low_stock":"Pocas unidades","unavailable":
```

---

## ARCHIVO 4 — `locales/en.default.json`

Con Ctrl+F busca la línea:  `"bundle": "Bundle",`

Debajo agrega una línea nueva para que quede así:
```
      "bundle": "Bundle",
      "low_stock": "Low stock",
      "unavailable": "Unavailable",
```

---

## ARCHIVO 5 — `config/settings_schema.json`

Con Ctrl+F busca:  `"id": "show_bundle_badge"`

Verás este bloque:
```
      {
        "type": "checkbox",
        "id": "show_bundle_badge",
        "label": "t:settings_schema.product_card.settings.product_badge.label__19",
        "default": true
      },
```

Justo DESPUÉS de la llave `},` de ese bloque, pega esto:
```json
      {
        "type": "checkbox",
        "id": "show_low_stock_badge",
        "label": "Mostrar badge \"Pocas unidades\"",
        "info": "Aparece automáticamente cuando el inventario del producto es igual o menor al límite definido. Requiere que Shopify rastree el inventario del producto.",
        "default": false
      },
      {
        "type": "range",
        "id": "low_stock_badge_limit",
        "min": 1,
        "max": 50,
        "step": 1,
        "unit": "u",
        "label": "Límite de unidades para \"Pocas unidades\"",
        "default": 5
      },
```

---

## DESPUÉS DE SUBIR LOS 5 ARCHIVOS

1. En el tema en vivo → **Personalizar** → **Configuración del tema** (engranaje ⚙️) → sección de badges de tarjeta de producto.
2. Activa **"Mostrar badge Pocas unidades"** y pon el límite → **Guardar**.
3. Abre un producto con pocas unidades en ventana de incógnito y revisa.

## IMPORTANTE
- El límite tiene un máximo de 50 en el deslizador. Si pusiste 42, está bien.
- En la PÁGINA del producto, el badge usa el stock de UNA variante (la seleccionada). Si el producto tiene varias variantes, cada una debe tener su propio stock ≤ límite.
- Si el producto está agotado, sale "Agotado", no "Pocas unidades".
- El inventario DEBE estar rastreado en Shopify ("Rastrear cantidad" activado).
