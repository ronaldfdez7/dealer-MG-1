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
  estado (disponible/vendido). Hoy son 6 autos de muestra, sin conexión a datos reales
  todavía — ver "Próxima fase" abajo
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

## Próxima fase: cuentas de cliente, inventario real y pago en línea

El cliente pidió tres cosas nuevas: login para clientes, inventario con fotos reales,
y pago en línea. Ninguna de las tres se puede hacer con un sitio estático — necesitan
base de datos y autenticación de verdad. Se van a construir en este orden, porque cada
una depende de la anterior:

1. **Inventario navegable** (en progreso) — hoy son autos de muestra en HTML fijo.
   El siguiente paso es conectar la parrilla a una base de datos real (propuesta:
   Supabase — plan gratis, incluye base de datos + login + almacenamiento de fotos)
   para que el inventario se pueda actualizar sin tocar código.
2. **Cuentas de cliente** — login/registro real con recuperación de contraseña,
   usando el mismo Supabase.
3. **Pago en línea** — bloqueado hasta decidir con el cliente qué procesador va a
   usar en Jamaica (WiPay, First Atlantic Commerce, Stripe, PayPal Business…). Sin
   cuenta de comercio no hay nada real que conectar.

Para arrancar la fase 2 en serio hace falta que el dueño del negocio cree una cuenta
gratuita en Supabase (igual que con Netlify) y comparta la URL del proyecto y la llave
pública — son datos seguros de exponer, no son credenciales secretas.

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
- [ ] **Inventario real** — los 6 autos en `#inventario` son de muestra (ficticios,
      con ícono en vez de foto). Falta conectar a Supabase (o similar) y cargar los
      autos, fotos y precios reales del negocio. Ver "Próxima fase" arriba.
- [ ] **Login de clientes y pago en línea** — todavía no existen; dependen de que se
      cree la cuenta de Supabase y se decida el procesador de pago. Ver "Próxima fase".
