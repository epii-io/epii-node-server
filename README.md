# epii-server

[![Build Status](https://travis-ci.org/epiijs/epii-server.svg?branch=master)](https://travis-ci.org/epiijs/epii-server)
[![Coverage Status](https://coveralls.io/repos/github/epiijs/epii-server/badge.svg?branch=master)](https://coveralls.io/github/epiijs/epii-server?branch=master)

A koa-based server with preset MVC model.

- @eggjs/router for controller
- koa-send for static files
- access to .well-known
- koa-bodyparser for body parse ~~& file upload~~
- epii-html5 for main document
- service container (new)

## Features

### MVC pipeline

    (Request)
        => / Static /
      => / Middleware /
      => / Router (Controller) /
        => / Render (Model) => (View) /
    (Response)

    &

    / Service / everywhere

### ASP.net-liked

Different ActionResult makes different response.  

```js
// service/service1.js
module.exports = async ({ context }) => {
  // anything you want to provide
  return {
    example: () => {}
  };
}

// controller
module.exports = [
  {
    path: '/',
    verb: 'get',
    body: async function ({ renders, service1 }) {
      // response text/plain
      return renders.text('text output');

      // response application/json
      return renders.json({ state: true });

      // response text/html by ViewRender
      return renders.view({ name: 'Li Lei' });

      // response application/octet-stream
      return renders.file('dataset.csv');

      // response redirect
      return renders.jump('/target');

      // use service
      return renders.json(service1.example());
    }
  }
];
```

### simple app shell definition

```js
// client/index.meta.js
module.exports = {
  base: 'simple', // inherit simple layout
  head: {
    styles: 'client/index.css'
  },
  body: {
    holder: 'client/index.html',
    scripts: 'client/index.js'
  }
};

// layout/simple.meta.js
module.exports = {
  head: {
    title: 'epii server',
    metas: [],
    styles: 'reset.css',
    icon: 'epii-icon.png'
  },
  body: {
    scripts: 'jquery-2.2.2.min.js'
  }
};
```

Or you maybe want to write HTML directly.

```js
// client/index.meta.js
module.exports = {
  html: 'client/index.html'
}
```

See also [`epii-html5`](https://github.com/epiijs/epii-html5).

## Usage

### project like this

```sh
(root)
├── [layout]
│   └── simple.meta.js
├── client
│   ├── ViewA
│   │   └── index.meta.js
│   └── ViewB
│       └── index.meta.js
├── server
│   ├── service
│   ├── middleware
│   │   └── $order.js
│   └── controller
│       └── index.js
└── static
    ├── (files)
    └── .well-known
        └── (files)
```

### install as dependency
```sh
npm install --save @epiijs/server@latest
```

### use API to start server
```js
const { startServer } = require('@epiijs/server');

startServer({
  name: 'YOUR-APP-NAME',
  port: 8080,
  path: {
    root: __dirname,
    server: {
      controller: 'server/controller',
      middleware: 'server/middleware',
      service: 'server/service',
    },
    client: 'client',
    layout: 'layout',
    static: 'static',
    /* will NEVER support upload */
  },
  static: {
    prefix: '__file',
  },
  expert: {
    'well-known': true, // default false
  }
});
```

### host server by nginx + certbot

Setup your node app in nginx conf.d directory.
```nginx
upstream your-app {
  server 127.0.0.1:your-port;
}

server {
  listen 80;
  server_name your-host;

  root /your-app-static-dir;
  index index.html;

  location / {
    proxy_pass http://your-host;
    proxy_set_header Host $host;
  }
}
```
Use certbot and it will try to validate domain by nginx conf. 

## FAQ

### How to contributing (TODO)

### How to serve static files

You can serve your JS + CSS + Media in CDN and use SSR to render app shell html (index.html) with state.
The recipe can perform balance between maintainability and performance.

#### Why not serve index.html in CDN

1. `index.html` in CDN can not be updated in real time.
2. `epii-html5` can render index.html very fast.
3. App shell HTMLs in CDN will work with tedious NGINX or CDN routes.

## Language (TODO)