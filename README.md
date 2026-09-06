# Total Trading Limited — sitio web

Sitio de una sola página para **Total Trading Limited** (Jamaica), un servicio de
pedidos especiales de vehículos: el cliente dice qué auto quiere, año y presupuesto,
y la empresa lo busca en subastas y redes de distribuidores de Japón, Europa y
Norteamérica, y lo entrega en Jamaica.

Además del pedido especial, el negocio ahora también va a tener un pequeño inventario
propio de autos ya en Jamaica, listos para venta inmediata — el sitio está creciendo
para cubrir ambos caminos: "búscamelo" y "cómpralo ya".

## Estructura

Sitio estático, sin build ni dependencias. `dist/` es el sitio completo.

```
dist/
  index.html    página completa: HTML, CSS y JS en un archivo
  hero.mp4      video de portada (entra 5 s después de la pantalla de carga)
  seal.png      sello de marca, panel de pedido especial
  seal-sm.png   sello reducido, barra superior / pie / favicon
  og.png        imagen de vista previa al compartir el enlace
  _headers      cabeceras de seguridad y caché
netlify.toml    configuración de despliegue
```

Todo el JavaScript es propio, sin librerías externas. Las únicas peticiones de red
son las tipografías de Google Fonts (Archivo y Plus Jakarta Sans).

## Qué hace

- **Pantalla de carga** de 3 s con un auto girando, dibujado en canvas
- **Portada** con foto de póster y el video entrando con fundido a los 5 s
- **Globo 3D interactivo** (canvas, arrastrable, con inercia) que muestra las rutas
  de búsqueda mundial convergiendo en Kingston
- **Proceso de 4 pasos** en scroll horizontal en escritorio, apilado en móvil
- **Inventario** (`#inventario`): parrilla de autos en stock con foto, specs, precio y
  estado, **leída en vivo desde la base de datos** (Supabase). Los precios siguen el
  selector USD/JMD igual que la sección de depósito
- **Cuentas de cliente**: registro, inicio de sesión, cierre de sesión y recuperación
  de contraseña. La sesión sobrevive a recargas y se renueva sola
- **Autos guardados**: con sesión iniciada, el corazón de cada auto lo guarda en la
  cuenta, y un filtro deja ver solo los guardados
- **Formulario de solicitud**: marca, modelo, año y presupuesto
- **Sección de depósito** con tres vías de pago y conversor de moneda
- **Bilingüe** inglés / español, y **modo día / noche** que respeta el ajuste del
  sistema y se puede forzar con el botón
- Respeta `prefers-reduced-motion`: sin animaciones automáticas si el visitante
  las tiene desactivadas

## Despliegue

Proyecto en Netlify: `total-trading-limited`
(site id `46601eab-8855-4222-908e-d35dd1277a29`) → https://total-trading-limited.netlify.app

Desde una máquina con Node:

```sh
npx netlify deploy --prod --dir=dist --site=46601eab-8855-4222-908e-d35dd1277a29
```

O manualmente: arrastrar el contenido de `dist/` a la zona de despliegue en
https://app.netlify.com/projects/total-trading-limited/deploys

## Base de datos (Supabase)

Proyecto: `ronaldfdez Dealer MJ` → https://uampqldgiditxueqmtlj.supabase.co

### Cómo agregar o editar autos (sin tocar código)

1. Entra a https://supabase.com → tu proyecto → **Table Editor** → tabla `cars`.
2. **Insert row** y llena los campos:

   | Campo | Qué poner |
   |---|---|
   | `make` | Marca: Toyota, BMW… (obligatorio) |
   | `model` | Modelo: Land Cruiser, X5… (obligatorio) |
   | `year` | Año (obligatorio) |
   | `price_usd` | Precio **en dólares**. El sitio convierte solo a JMD |
   | `fuel` | Petrol, Diesel, Hybrid… |
   | `transmission` | Automatic / Manual |
   | `mileage` | Millaje, solo el número |
   | `status` | `in_stock`, `reserved` o `sold` |
   | `photo_url` | Enlace a la foto. Si se deja vacío sale un ícono con "Foto próximamente" |
   | `published` | **Déjalo en `false` mientras lo preparas.** Ponlo en `true` para que aparezca en el sitio |

3. Los cambios salen en la web al recargar la página. No hay que desplegar nada.

**`published` es el interruptor de publicación**: un auto con `published = false` es
invisible para el sitio, aunque exista en la tabla. Sirve para preparar una ficha con
calma sin que salga a medio llenar.

### ⚠️ Falta configurar el envío de correos antes de abrir el registro al público

Supabase trae un servicio de correo incluido, pero **solo envía a direcciones
preautorizadas** (las del equipo del proyecto), tiene un límite bajo por hora y su
propia documentación dice que **no es para producción**.

Traducido: un cliente real que se registre **no recibirá** el correo de confirmación
ni el de recuperar contraseña. Funciona para probar con tu propio correo, nada más.

Para abrirlo al público hay que conectar un SMTP propio en
**Authentication → SMTP Settings**. Hay opciones gratuitas suficientes para este
volumen (Brevo, Resend, Mailgun). Es una configuración de una sola vez, ~15 minutos.

También conviene revisar en **Authentication → URL Configuration** que el *Site URL*
apunte a https://total-trading-limited.netlify.app, porque de ahí salen los enlaces
de confirmación y de recuperación de contraseña.

### Seguridad

La llave que va en `dist/index.html` es la **llave publicable**, hecha para ser pública.
Lo que realmente protege los datos es el row-level security de las tablas:

- **`cars`**: con la llave publicable solo se pueden **leer** los autos publicados.
  Insertar, modificar y borrar están prohibidos. Editar solo se puede desde el panel
  de Supabase, con la cuenta del dueño.
- **`favourites`**: cada política está atada a `auth.uid()`, así que un cliente con
  sesión solo alcanza **sus propias** filas. Comprobado contra la base: un usuario no
  ve los guardados de otro, no puede guardar en su nombre ni borrarle nada, y alguien
  sin sesión no ve ninguno.

> La llave publicable viaja **solo en la cabecera `apikey`**. Supabase intenta leer la
> cabecera `Authorization: Bearer` como un JWT y rechaza la petición con "Invalid JWT"
> si le llega ahí. En `Authorization` solo va el token de sesión del propio usuario.

> La llave `service_role` / `sb_secret_...` **nunca** debe ir en el sitio ni compartirse.

## Próxima fase: pago en línea

1. ~~Inventario navegable conectado a base de datos~~ — **hecho**.
2. ~~Cuentas de cliente~~ — **hecho**, pendiente solo el SMTP (ver aviso arriba).
3. **Pago en línea** — bloqueado hasta decidir con el cliente qué procesador va a usar
   en Jamaica (WiPay, First Atlantic Commerce, Stripe, PayPal Business…). Sin cuenta de
   comercio no hay nada real que conectar. Cuando exista, el cobro debe autorizarse en
   un servidor (Netlify Functions), nunca desde el navegador.

## Pendiente antes de salir a producción

Esto es una **vista previa de diseño**. La página lo dice abiertamente en varios
sitios, y esos avisos deben quitarse solo cuando lo de abajo esté resuelto:

- [ ] **Números de cuenta bancaria** — NCB, Scotiabank y JN Bank dicen "pendiente"
- [ ] **Montos de depósito reales** — los de US$500 / 1.000 / 2.500 / 5.000 son supuestos
- [ ] **Pasarela de pago** — hace falta cuenta de comercio (First Atlantic Commerce
      o WiPay) y un servidor que autorice cada cobro. La página **no pide datos de
      tarjeta en ningún punto**, y así debe seguir hasta que exista ese servidor.
- [x] **Formulario de solicitud** — conectado a Netlify Forms; llega al correo configurado
      en el panel de Netlify (Site settings → Forms → Form notifications) y también queda
      guardado ahí bajo la pestaña "Forms"
- [ ] **Tasa de cambio** — fija en 158 JMD/USD, etiquetada como indicativa. En producción
      debería venir de un servicio de tasas. Se cambia en la constante `USD_TO_JMD`
      dentro de `dist/index.html`.
- [ ] **Video de portada** — es vertical (540×960); en escritorio se recorta y se ve
      suave. Una versión horizontal mejoraría bastante.
- [ ] **Fotos propias** — la portada usa una imagen de referencia; conviene sustituirla
      por fotos de entregas reales con derechos de la empresa.
- [ ] **Autos reales en el inventario** — la conexión a la base ya funciona, pero los
      6 autos cargados son ficticios y sin foto. Falta que el negocio cargue los autos
      reales (ver "Cómo agregar o editar autos" arriba) y borre los de muestra. El
      aviso de "autos de muestra" bajo la parrilla debe quitarse en ese momento.
- [ ] **SMTP para las cuentas** — el registro y la recuperación de contraseña ya
      funcionan, pero los correos solo llegan a direcciones preautorizadas hasta
      conectar un SMTP propio. Ver el aviso en la sección de la base de datos.
- [ ] **Pago en línea** — todavía no existe; depende de que se decida el procesador.
      Ver "Próxima fase".
