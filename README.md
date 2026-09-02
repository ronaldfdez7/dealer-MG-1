# Total Trading Limited — sitio web

Sitio de una sola página para **Total Trading Limited** (Jamaica), un servicio de
pedidos especiales de vehículos: el cliente dice qué auto quiere, año y presupuesto,
y la empresa lo busca en subastas y redes de distribuidores de Japón, Europa y
Norteamérica, y lo entrega en Jamaica.

No es un concesionario con inventario en piso, así que el sitio está construido
alrededor de la solicitud de búsqueda, no de una parrilla de autos en stock.

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

## Pendiente antes de salir a producción

Esto es una **vista previa de diseño**. La página lo dice abiertamente en varios
sitios, y esos avisos deben quitarse solo cuando lo de abajo esté resuelto:

- [ ] **Números de cuenta bancaria** — NCB, Scotiabank y JN Bank dicen "pendiente"
- [ ] **Montos de depósito reales** — los de US$500 / 1.000 / 2.500 / 5.000 son supuestos
- [ ] **Pasarela de pago** — hace falta cuenta de comercio (First Atlantic Commerce
      o WiPay) y un servidor que autorice cada cobro. La página **no pide datos de
      tarjeta en ningún punto**, y así debe seguir hasta que exista ese servidor.
- [ ] **Formulario de solicitud** — hoy no envía nada; falta conectarlo a correo o WhatsApp
- [ ] **Tasa de cambio** — fija en 158 JMD/USD, etiquetada como indicativa. En producción
      debería venir de un servicio de tasas. Se cambia en la constante `USD_TO_JMD`
      dentro de `dist/index.html`.
- [ ] **Video de portada** — es vertical (540×960); en escritorio se recorta y se ve
      suave. Una versión horizontal mejoraría bastante.
- [ ] **Fotos propias** — la portada usa una imagen de referencia; conviene sustituirla
      por fotos de entregas reales con derechos de la empresa.
