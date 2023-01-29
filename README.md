## ExpressJS x OpenTelemetry instrumentation

Creating a blank application to auto-instrument OpenTelemetry-JS and Express JS to output trace data to the console.

### Goals

- To understand how raw tracing works in basic application.
- To see the output and the different properties of a trace.

Although it is not as interesting in the console as compared to using a visualization tool, we can still gather valuable information by just looking at the raw trace.

### Helpful links

- [opentelemetry-js github repository](https://github.com/open-telemetry/opentelemetry-js)
- [official opentelemetry docs](https://opentelemetry.io/docs/instrumentation/js/)

### Step 1

Initialize a folder to home your repository. I named it `express-optel`.

### Step 2

Install `express-generator` using the terminal.
`express-generator` helps to quickly create an application skeleton.

```
npx express-generator
```

### Step 3

Install the application using `npm install`.

```
npm install

// to address all issues, run
npm audit fix --force
```

### Step 4

Install `opentelemetry-js` dependencies. Refer to the [opentelemetry-js readme](https://github.com/open-telemetry/opentelemetry-js)for more information.

```
npm install --save @opentelemetry/api
npm install --save @opentelemetry/sdk-node
npm install --save @opentelemetry/auto-instrumentations-node
```

### Step 5

Instantiate tracing by creating `tracing.js`.

```
// tracing.js

'use strict'

const process = require('process');
const opentelemetry = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { ConsoleSpanExporter } = require('@opentelemetry/sdk-trace-base');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');

// configure the SDK to export telemetry data to the console
// enable all auto-instrumentations from the meta package
const traceExporter = new ConsoleSpanExporter();
const sdk = new opentelemetry.NodeSDK({
  resource: new Resource({
    [SemanticResourceAttributes.SERVICE_NAME]: 'my-service',
  }),
  traceExporter,
  instrumentations: [getNodeAutoInstrumentations()]
});

// initialize the SDK and register with the OpenTelemetry API
// this enables the API to record telemetry
sdk.start()
  .then(() => console.log('Tracing initialized'))
  .catch((error) => console.log('Error initializing tracing', error));

// gracefully shut down the SDK on process exit
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('Tracing terminated'))
    .catch((error) => console.log('Error terminating tracing', error))
    .finally(() => process.exit(0));
});
```

In `tracing.js`, you should be able to view `const traceExporter = new ConsoleSpanExporter()` function where the purpose of this is to see what is going on in the console without using any observability software such as Datadog, yet.

### Step 6

In `package.json`, add a reference to `./tracing.js` in the start-up script.

```
"scripts": {
"start": "node -r ./tracing.js ./bin/www"
}

```

### Step 7

Run the application by typing `npm run start` in the terminal. You should be able to see `Tracing initialized` in the console upon successful start-up of tracing.

<p align="center">
  <img src="https://github.com/legendkong/express-optel/blob/master/mdImages/traceSuccess.png?raw=true" width="800"> <br>
</p>

### Step 8

Open another terminal tab. Use `curl` to send a request to the web application.

```
$curl http://localhost:3000/
```

<p align="center">
  <img src="https://github.com/legendkong/express-optel/blob/master/mdImages/htmlOutput.png?raw=true" width="800"> <br>
  It should return a html page in your console.
</p>
<br>

In your main terminal tab, you should see the calls to the server and the tracing information, such as `traceId`, `parentId`, `name`, `id`, `kind`, `timestamp`, and many more.

<p align="center">
  <img src="https://github.com/legendkong/express-optel/blob/master/mdImages/traceInfo.png?raw=true" width="800"> <br>
</p>
<br>

Auto-instrumentation complete!
<br>

### What's next

What we can do next is to send all these info in the console into an OpenTelemetry collector, which can be locally run or externally hosted.
<br>
<br>
Based on the OpenTelemetry docs: <br>
When to use an OpenTelemetry collector?
For most language specific instrumentation libraries you have exporters for popular backends and OTLP.
<br>You might wonder,
under what circumstances does one use a collector to send data, as opposed to having each service send directly to the backend?
For trying out and getting started with OpenTelemetry, sending your data directly to a backend is a great way to get value quickly. Also, in a development or small-scale environment you can get decent results without a collector. <br><br>
However, in general we recommend using a collector alongside your service, since it allows your service to offload data quickly and the collector can take care of additional handling like retries, batching, encryption or even sensitive data filtering.
<br><br>
It is also easier to [setup a collector](https://opentelemetry.io/docs/collector/getting-started) than you might think: the default OTLP exporters in each language assume a local collector endpoint, so you’d start up a collector and you’d just start getting telemetry.
