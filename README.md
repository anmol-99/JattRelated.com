<!doctype html>
<html class="js full-height" lang="{{ request.locale.iso_code }}">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <meta name="theme-color" content="">
    <link rel="canonical" href="{{ canonical_url }}">

    {%- if settings.favicon != blank -%}
      <link rel="icon" type="image/png" href="{{ settings.favicon | image_url: width: 32, height: 32 }}">
    {%- endif -%}

    {%- unless settings.type_header_font.system? -%}
      <link rel="preconnect" href="https://fonts.shopifycdn.com" crossorigin>
    {%- endunless -%}

    <title>{{ shop.name }}</title>
    <meta name="description" content="{{ page_description | escape }}">

    {% render 'meta-tags' %}

    {{ content_for_header }}

    {%- liquid
      assign body_font_bold = settings.type_body_font | font_modify: 'weight', 'bold'
      assign body_font_italic = settings.type_body_font | font_modify: 'style', 'italic'
      assign body_font_bold_italic = body_font_bold | font_modify: 'style', 'italic'
    %}

    {% style %}
      {{ settings.type_body_font | font_face: font_display: 'swap' }}
      {{ body_font_bold | font_face: font_display: 'swap' }}
      {{ body_font_italic | font_face: font_display: 'swap' }}
      {{ body_font_bold_italic | font_face: font_display: 'swap' }}
      {{ settings.type_header_font | font_face: font_display: 'swap' }}

      {% for scheme in settings.color_schemes -%}
        {% assign scheme_classes = scheme_classes | append: ', .color-' | append: scheme.id %}
        {% if forloop.index == 1 -%}
          :root,
        {%- endif %}
        .color-{{ scheme.id }} {
          --color-background: {{ scheme.settings.background.red }},{{ scheme.settings.background.green }},{{ scheme.settings.background.blue }};
          {% if scheme.settings.background_gradient != empty %}
            --gradient-background: {{ scheme.settings.background_gradient }};
          {% else %}
            --gradient-background: {{ scheme.settings.background }};
          {% endif %}
          --color-foreground: {{ scheme.settings.text.red }},{{ scheme.settings.text.green }},{{ scheme.settings.text.blue }};
          --color-shadow: {{ scheme.settings.shadow.red }},{{ scheme.settings.shadow.green }},{{ scheme.settings.shadow.blue }};
          --color-button: {{ scheme.settings.button.red }},{{ scheme.settings.button.green }},{{ scheme.settings.button.blue }};
          --color-button-text: {{ scheme.settings.button_label.red }},{{ scheme.settings.button_label.green }},{{ scheme.settings.button_label.blue }};
          --color-secondary-button: {{ scheme.settings.background.red }},{{ scheme.settings.background.green }},{{ scheme.settings.background.blue }};
          --color-secondary-button-text: {{ scheme.settings.secondary_button_label.red }},{{ scheme.settings.secondary_button_label.green }},{{ scheme.settings.secondary_button_label.blue }};
          --color-link: {{ scheme.settings.secondary_button_label.red }},{{ scheme.settings.secondary_button_label.green }},{{ scheme.settings.secondary_button_label.blue }};
          --color-badge-foreground: {{ scheme.settings.text.red }},{{ scheme.settings.text.green }},{{ scheme.settings.text.blue }};
          --color-badge-background: {{ scheme.settings.background.red }},{{ scheme.settings.background.green }},{{ scheme.settings.background.blue }};
          --color-badge-border: {{ scheme.settings.text.red }},{{ scheme.settings.text.green }},{{ scheme.settings.text.blue }};
          --payment-terms-background-color: rgb({{ scheme.settings.background.rgb }});
        }
      {% endfor %}

      {{ scheme_classes | prepend: 'body' }} {
        color: rgba(var(--color-foreground), 0.75);
        background-color: rgb(var(--color-background));
      }

      :root {
        --font-body-family: {{ settings.type_body_font.family }}, {{ settings.type_body_font.fallback_families }};
        --font-body-style: {{ settings.type_body_font.style }};
        --font-body-weight: {{ settings.type_body_font.weight }};
        --font-heading-family: {{ settings.type_header_font.family }}, {{ settings.type_header_font.fallback_families }};
        --font-heading-style: {{ settings.type_header_font.style }};
        --font-heading-weight: {{ settings.type_header_font.weight }};

        --font-body-scale: {{ settings.body_scale | divided_by: 100.0 }};
        --font-heading-scale: {{ settings.heading_scale | times: 1.0 | divided_by: settings.body_scale }};
        --media-padding: {{ settings.media_padding }}px;
        --media-border-opacity: {{ settings.media_border_opacity | divided_by: 100.0 }};
        --media-border-width: {{ settings.media_border_thickness }}px;
        --media-radius: {{ settings.media_radius }}px;
        --media-shadow-opacity: {{ settings.media_shadow_opacity | divided_by: 100.0 }};
        --media-shadow-horizontal-offset: {{ settings.media_shadow_horizontal_offset }}px;
        --media-shadow-vertical-offset: {{ settings.media_shadow_vertical_offset }}px;
        --media-shadow-blur-radius: {{ settings.media_shadow_blur }}px;
        --media-shadow-visible: {% if settings.media_shadow_opacity > 0 %}1{% else %}0{% endif %};

        --page-width: {{ settings.page_width | divided_by: 10 }}rem;
        --page-width-margin: {% if settings.page_width == '1600' %}2{% else %}0{% endif %}rem;

        --product-card-image-padding: {{ settings.card_image_padding | divided_by: 10.0 }}rem;
        --product-card-corner-radius: {{ settings.card_corner_radius | divided_by: 10.0 }}rem;
        --product-card-text-alignment: {{ settings.card_text_alignment }};
        --product-card-border-width: {{ settings.card_border_thickness | divided_by: 10.0 }}rem;
        --product-card-border-opacity: {{ settings.card_border_opacity | divided_by: 100.0 }};
        --product-card-shadow-opacity: {{ settings.card_shadow_opacity | divided_by: 100.0 }};
        --product-card-shadow-visible: {% if settings.card_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --product-card-shadow-horizontal-offset: {{ settings.card_shadow_horizontal_offset | divided_by: 10.0 }}rem;
        --product-card-shadow-vertical-offset: {{ settings.card_shadow_vertical_offset | divided_by: 10.0 }}rem;
        --product-card-shadow-blur-radius: {{ settings.card_shadow_blur | divided_by: 10.0 }}rem;

        --collection-card-image-padding: {{ settings.collection_card_image_padding | divided_by: 10.0 }}rem;
        --collection-card-corner-radius: {{ settings.collection_card_corner_radius | divided_by: 10.0 }}rem;
        --collection-card-text-alignment: {{ settings.collection_card_text_alignment }};
        --collection-card-border-width: {{ settings.collection_card_border_thickness | divided_by: 10.0 }}rem;
        --collection-card-border-opacity: {{ settings.collection_card_border_opacity | divided_by: 100.0 }};
        --collection-card-shadow-opacity: {{ settings.collection_card_shadow_opacity | divided_by: 100.0 }};
        --collection-card-shadow-visible: {% if settings.collection_card_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --collection-card-shadow-horizontal-offset: {{ settings.collection_card_shadow_horizontal_offset | divided_by: 10.0 }}rem;
        --collection-card-shadow-vertical-offset: {{ settings.collection_card_shadow_vertical_offset | divided_by: 10.0 }}rem;
        --collection-card-shadow-blur-radius: {{ settings.collection_card_shadow_blur | divided_by: 10.0 }}rem;

        --blog-card-image-padding: {{ settings.blog_card_image_padding | divided_by: 10.0 }}rem;
        --blog-card-corner-radius: {{ settings.blog_card_corner_radius | divided_by: 10.0 }}rem;
        --blog-card-text-alignment: {{ settings.blog_card_text_alignment }};
        --blog-card-border-width: {{ settings.blog_card_border_thickness | divided_by: 10.0 }}rem;
        --blog-card-border-opacity: {{ settings.blog_card_border_opacity | divided_by: 100.0 }};
        --blog-card-shadow-opacity: {{ settings.blog_card_shadow_opacity | divided_by: 100.0 }};
        --blog-card-shadow-visible: {% if settings.blog_card_shadow_opacity > 0 %}1{% else %}0{% endif %};
        --blog-card-shadow-horizontal-offset: {{ settings.blog_card_shadow_horizontal_offset | divided_by: 10.0 }}rem;
        --blog-card-shadow-vertical-offset: {{ settings.blog_card_shadow_vertical_offset | divided_by: 10.0 }}rem;
        --blog-card-shadow-blur-radius: {{ settings.blog_card_shadow_blur | divided_by: 10.0 }}rem;

        --badge-corner-radius: {{ settings.badge_corner_radius | divided_by: 10.0 }}rem;

        --spacing-sections-desktop: {{ settings.spacing_sections }}px;
        --spacing-sections-mobile: {% if settings.spacing_sections < 24 %}{{ settings.spacing_sections }}{% else %}{{ settings.spacing_sections | divided_by: 10.0 }}rem{% endif %};

        --header-height: {{ settings.header_height }}px;
        --footer-height: {{ settings.footer_height }}px;
      }

      {% if settings.page_width == '1600' %}
        @media (min-width: 1600px) {
          .page-width-container {
            max-width: {{ settings.page_width }}px;
          }
        }
      {% endif %}
    {% endstyle %}

    {% script %}
      var theme_settings = {{ settings | json }};
      var color_scheme_classes = "{{ scheme_classes | split: ',' | join: ' ' }}";
    {% endscript>

    {% render 'preload' %}

    {%- render 'font-variables' -%}
</head>
<body class="template-{{ template }} {{ scheme_classes }}">
    <header class="site-header">
        <div class="header-content">
            <div class="header-logo">
                <a href="{{ shop.url }}">
                    <img src="{{ settings.header_logo | image_url: width: 200 }}" alt="{{ shop.name }}">
                </a>
            </div>
            <nav class="header-navigation">
                <ul>
                    <li><a href="/collections">Collections</a></li>
                    <li><a href="/blogs">Blogs</a></li>
                    <li><a href="/about-us">About Us</a></li>
                    <li><a href="/contact">Contact</a></li>
                </ul>
            </nav>
        </div>
    </header>

    <main class="main-content">
        {{ content_for_layout }}
    </main>

    <footer class="site-footer">
        <div class="footer-content">
            <div class="footer-links">
                <a href="/privacy-policy">Privacy Policy</a>
                <a href="/terms-of-service">Terms of Service</a>
                <a href="/refund-policy">Refund Policy</a>
            </div>
            <div class="footer-social">
                <a href="https://facebook.com/{{ shop.facebook }}" target="_blank">Facebook</a>
                <a href="https://twitter.com/{{ shop.twitter }}" target="_blank">Twitter</a>
                <a href="https://instagram.com/{{ shop.instagram }}" target="_blank">Instagram</a>
            </div>
            <div class="footer-bottom">
                <p>&copy; {{ 'now' | date: '%Y' }} {{ shop.name }}. All rights reserved.</p>
            </div>
        </div>
    </footer>

    {% render 'scripts' %}
</body>
</html>
