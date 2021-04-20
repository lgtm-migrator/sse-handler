# sse-handler

:boom: [ESM] The server-sent events (sse) handler for Node.js according to w3c and whatwg

---

![npm](https://img.shields.io/david/awesomeorganization/sse-handler)
![npm](https://img.shields.io/npm/v/@awesomeorganization/sse-handler)
![npm](https://img.shields.io/npm/dt/@awesomeorganization/sse-handler)
![npm](https://img.shields.io/npm/l/@awesomeorganization/sse-handler)
![npm](https://img.shields.io/bundlephobia/minzip/@awesomeorganization/sse-handler)
![npm](https://img.shields.io/bundlephobia/min/@awesomeorganization/sse-handler)

---

## Example

Full example in `/example` folder.

```
import { http } from '@awesomeorganization/servers'
import { rewriteHandler } from '@awesomeorganization/rewrite-handler'
import { sseHandler } from '@awesomeorganization/sse-handler'
import { staticHandler } from '@awesomeorganization/static-handler'

const example = async () => {
  const rewriteMiddleware = rewriteHandler({
    rules: [
      {
        pattern: '(.*)/$',
        replacement: '$1/index.html',
      },
    ],
  })
  const sseMidleware = sseHandler()
  const staticMiddleware = await staticHandler({
    directoryPath: './static',
  })
  http({
    listenOptions: {
      host: '127.0.0.1',
      port: 3000,
    },
    onListening() {
      setInterval(() => {
        const timestamp = new Date().toISOString()
        sseMidleware.push({
          data: `${timestamp}: Hi!`,
        })
        sseMidleware.push({
          data: [timestamp, 'This is multiline', 'string with event.'].join('\n'),
          event: 'someEvent',
        })
      }, 3e3)
    },
    onRequest(request, response) {
      switch (request.method) {
        case 'GET': {
          switch (request.url) {
            case '/sse': {
              sseMidleware.handle({
                request,
                response,
              })
              return
            }
            default: {
              rewriteMiddleware.handle({
                request,
                response,
              })
              staticMiddleware.handle({
                request,
                response,
              })
              return
            }
          }
        }
      }
      response.end()
    },
  })
  // TRY
  // http://127.0.0.1:3000/
}

example()
```
