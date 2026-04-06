# Guía de Configuración Definitiva: Formulario con WPForms

Para mantener el diseño premium de INGEDACA en tu WordPress usando **WPForms Lite**, necesitas replicar la estructura de campos y aplicarle el estilo "Glassmorphism" oscuro con detalles en naranja mediante un CSS especializado para este plugin.

---

## 🛠️ Paso 1: Crea la estructura en WPForms

1. **Añadir Nuevo:** Ve a tu panel de WordPress > `WPForms > Añadir nuevo` y elige "Formulario en blanco" (Blank Form).
2. **Agrega los campos:** Arrastra desde el panel izquierdo los siguientes campos en este orden estricto:
    - **Texto de una línea** (Single Line Text)
    - **Correo electrónico** (Email)
    - **Texto de una línea** (Single Line Text)
    - **Desplegable** (Dropdown)
    - **Texto de párrafo** (Paragraph Text)

---

## ⚙️ Paso 2: Configura cada campo (El truco del "Placeholder")

Como nuestro diseño no tiene etiquetas de texto por fuera (Labels), sino que el texto va por dentro de la caja (Placeholders), debemos configurarlo así:

1. **Haz clic en el primer campo (Texto de una línea):**
    - Pestaña *General* > Etiqueta: Escribe "Nombre Completo".
    - Pestaña *General* > Marca la casilla "Obligatorio" (Required).
    - Pestaña *Avanzado* > Texto del campo (Placeholder): Escribe "Nombre Completo".
    - Pestaña *Avanzado* > Ocultar etiqueta (Hide Label): Marca esta opción.
    
2. **Segundo campo (Correo):**
    - Etiqueta y Placeholder: "Correo Corporativo". Obligatorio y Ocultar etiqueta.
    
3. **Tercer campo (Empresa):**
    - Etiqueta y Placeholder: "Empresa / Organización". Obligatorio y Ocultar etiqueta.
    
4. **Cuarto campo (Desplegable):**
    - Opciones: "Energía & Petróleo", "Alimentos & Bebidas", "Ciencias de la Vida", "Infraestructura".
    - Pestaña *Avanzado* > En "Texto por defecto" (Placeholder), escribe "Sector Industrial". Oculta la etiqueta.
    
5. **Quinto campo (Texto de párrafo):**
    - Etiqueta y Placeholder: "¿Cómo podemos ayudarte?". Ocultar etiqueta.

6. **El Botón:** 
    - Ve a `Ajustes > General` a la izquierda. En "Texto del botón de enviar", escribe: **Registrar Interés**. Guarda todo y cierra.

---

## 🔌 Paso 3: Insértalo en Elementor (El Truco)

Aquí está la clave para que quede *dentro* de la caja con bordes redondeados:

1. Copia el **Código Corto (Shortcode)** que WPForms te da al guardar (ejemplo: `[wpforms id="15"]`).
2. Ve a Editar con Elementor la Landing Page y haz clic en la sección "Hablemos del Futuro" (al final de la página).
3. Verás que esa sección es un solo **Widget HTML**. En el cuadro de texto del widget, busca esta línea:
   `<!-- Aqui pegaria el shortcode del formulario de WP -->`
4. **Borra esa línea y pega tu shortcode `[wpforms id="15"]` exactamente ahí.**
5. ¡Listo! Elementor es inteligente y procesará el formulario directamente dentro de nuestro contenedor premium (`form-centered-wrapper`). **No necesitas arrastrar un widget nuevo.**

---

## 🎨 Paso 4: El Código CSS Mágico para WPForms

WPForms tiene sus propias "clases" CSS. Para que se vea con el diseño Glassmorphism de nuestra landing preium, **Copia este bloque y pégalo al final de tu Widget HTML en Elementor:**

```css
/* =========================================
   ESTILOS PREMIUM PARA WPFORMS (INGEDACA)
   ========================================= */

/* Estilo para los campos de texto y listas */
div.wpforms-container-full .wpforms-form .wpforms-field input,
div.wpforms-container-full .wpforms-form .wpforms-field select,
div.wpforms-container-full .wpforms-form .wpforms-field textarea {
    width: 100% !important;
    padding: 1rem 1.5rem !important;
    margin-bottom: 0.5rem !important;
    background: rgba(255, 255, 255, 0.05) !important;
    border: 1px solid rgba(255, 255, 255, 0.1) !important;
    border-radius: 12px !important;
    color: white !important;
    font-family: 'Outfit', sans-serif !important;
    font-size: 1rem !important;
    transition: all 0.3s ease !important;
}

/* El menú desplegable (corrección de fondo gris) */
div.wpforms-container-full .wpforms-form .wpforms-field select option {
    background-color: #0f0f0f;
    color: white;
}

/* Efecto al hacer clic (Foco naranja Premium) */
div.wpforms-container-full .wpforms-form .wpforms-field input:focus,
div.wpforms-container-full .wpforms-form .wpforms-field select:focus,
div.wpforms-container-full .wpforms-form .wpforms-field textarea:focus {
    outline: none !important;
    border-color: #f43f07 !important;
    background: rgba(255, 255, 255, 0.08) !important;
    box-shadow: 0 0 15px rgba(244, 63, 7, 0.2) !important;
}

/* Estilo del botón "Registrar Interés" (Botón Pastilla) */
div.wpforms-container-full .wpforms-form .wpforms-submit-container button {
    width: 100% !important;
    padding: 1.2rem 2rem !important;
    background-color: #f43f07 !important;
    color: white !important;
    border: none !important;
    border-radius: 50px !important;
    font-family: 'Zen Dots', sans-serif !important;
    font-size: 1rem !important;
    text-transform: uppercase !important;
    cursor: pointer !important;
    transition: all 0.3s ease !important;
    margin-top: 1rem !important;
}

/* Efecto Hover del Botón */
div.wpforms-container-full .wpforms-form .wpforms-submit-container button:hover {
    background-color: #d13504 !important;
    transform: translateY(-3px);
    box-shadow: 0 10px 20px rgba(244, 63, 7, 0.3) !important;
}

/* Ocultar etiquetas forzadamente (Por Si Acaso) */
div.wpforms-container-full .wpforms-form .wpforms-field-label {
    display: none !important;
}
```

¡Con esto tu formulario WPForms abandonará su aburrido color blanco clásico y adoptará toda la personalidad digital de Ingedaca!

---

## 📬 Paso 5: ¿Dónde veo las respuestas que llenen mis clientes?

**¡Atención aquí!** Si estás usando la versión gratuita de WPForms (**WPForms Lite**), el plugin **NO** guarda las respuestas dentro de una base de datos en el panel de WordPress. 

En su lugar, WPForms funciona enviando automáticamente un **Correo Electrónico** cada vez que alguien llena el formulario.

**Para asegurarte de que te lleguen los correos:**
1. Ve a `WPForms > Todos los formularios` y edita tu formulario "Hablemos del Futuro".
2. En el panel izquierdo, ve a **Ajustes > Avisos** (Settings > Notifications).
3. Asegúrate de que la opción "Activar los avisos" esté **encendida**.
4. En el campo "Enviar a la dirección de correo" (Send To Email Address), escribe el correo corporativo (ej. *ventas@ingedaca.com*), o déjalo como `{admin_email}` si quieres que le llegue al correo del administrador de WordPress.
5. Haz clic en Guardar.

*(Nota: Si más adelante deseas ver un registro de todos los correos guardados directamente dentro de WordPress, necesitarás adquirir WPForms Pro, o usar un plugin gratuito alternativo como "Fluent Forms").*
