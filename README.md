# strapi-provider-upload-local-custom-path

## Configurations

This provider has two parameters: `path` and `sizeLimit`.

**Example**

`./config/plugins.js`

```js
module.exports = ({ env }) => ({
  upload: {
    provider: 'local-custom-path',
    providerOptions: {
      path: env('UPLOADS_PATH') || '.tmp/media',
      sizeLimit: 100000
    },
  },
});
```

The `sizeLimit` parameter must be a number. Be aware that the unit is in bytes, and the default is 1000000. When setting this value high, you should make sure to also configure the body parser middleware `maxFileSize` so the file can be sent and processed. Read more [here](https://strapi.io/documentation/v3.x/plugins/upload.html#configuration)

**Note**

You will likely need to add `./middlewares/upload/index.js`

```js
'use strict';

const { resolve } = require('path');
const range = require('koa-range');
const koaStatic = require('koa-static');
const get = require('lodash/get')

module.exports = strapi => ({
  initialize() {
    const configPublicPath = strapi.config.get(
      'middleware.settings.public.path',
      strapi.config.paths.static
    );

    const staticDir = get(strapi, "config.plugins.upload.providerOptions.path", resolve(strapi.dir, configPublicPath));

    strapi.app.on('error', err => {
      if (err.code === 'EPIPE') {
        return;
      }

      strapi.app.onerror(err);
    });

    strapi.router.get('/uploads/(.*)', range, koaStatic(staticDir, { defer: true }));
  },
});
```

## Resources

- [License](LICENSE)

## Links

- [Strapi website](http://strapi.io/)
- [Strapi community on Slack](http://slack.strapi.io)
- [Strapi news on Twitter](https://twitter.com/strapijs)
