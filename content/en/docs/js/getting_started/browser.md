---
title: "Browser"
weight: 2
---

This guide uses the example application in HTML & javascript provided below, but the steps to instrument your own application should be broadly the same. Here is an overview of what we will be doing.

- Install the required OpenTelemetry libraries
- Initialize a global [tracer](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#tracer)
- Initialize and register a [span exporter](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/sdk.md#span-exporter)

This is a very simple guide, if you'd like to see more complex examples go to [examples/tracer-web](https://github.com/open-telemetry/opentelemetry-js/tree/main/examples/tracer-web).

Copy the following file into an empty directory and call it `index.html`.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Document Load Instrumentation Example</title>
  <base href="/">
  <!--
    https://www.w3.org/TR/trace-context/
    Set the `traceparent` in the server's HTML template code. It should be
    dynamically generated server side to have the server's request trace Id,
    a parent span Id that was set on the server's request span, and the trace
    flags to indicate the server's sampling decision
    (01 = sampled, 00 = notsampled).
    '{version}-{traceId}-{spanId}-{sampleDecision}'
  -->
  <meta name="traceparent" content="00-ab42124a3c573678d4d8b21ba52df3bf-d21f7bc17caa5aba-01">
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  Example of using Web Tracer with document load plugin with console exporter and collector exporter
</body>
</html>
```

# Installation

To create traces in the browser, you will need `@opentelemetry/web`, and the instrumentation `@opentelemetry/instrumentation-document-load`:

```shell
npm install @opentelemetry/web @opentelemetry/instrumentation-document-load
```

In the following we will use parcel as web application bundler, but you can of course also use any other build tool:

```shell
npm install -g parcel
```

# Initialization and Configuration

Create a empty file called `document-load.js` and add the following code to your html right before the body end tag:

```html
<script type="text/javascript" src="document-load.js"></script>
```

We will add some code that will trace the document load timings and output those as OpenTelemetry Spans.


## Creating a Tracer Provider

Add the following code to the `document-load.js` to create a tracer provider, which brings the plugin to trace document load:

```javascript
import { registerInstrumentations } from '@opentelemetry/instrumentation';
import { DocumentLoadInstrumentation } from '@opentelemetry/instrumentation-document-load';
import { WebTracerProvider } from '@opentelemetry/web';

// Minimum required setup - supports only synchronous operations
const provider = new WebTracerProvider();
provider.register();

registerInstrumentations({
  instrumentations: [
    new DocumentLoadInstrumentation(),
  ],
  tracerProvider: provider,
});
```

Run `parcel index.html` and open the development webserver (e.g. at http://localhost:1234) to see if your code works.

There will be no output of traces yet, for this we need to add an exporter.

## Creating a Console Exporter

To export traces, modify `document-load.js` so that it matches the following code snippet:

```javascript
import { registerInstrumentations } from '@opentelemetry/instrumentation';
import { DocumentLoadInstrumentation } from '@opentelemetry/instrumentation-document-load';
import { WebTracerProvider } from '@opentelemetry/web';
import { ConsoleSpanExporter, SimpleSpanProcessor } from '@opentelemetry/tracing';

// Minimum required setup - supports only synchronous operations
const provider = new WebTracerProvider();
provider.addSpanProcessor(new SimpleSpanProcessor(new ConsoleSpanExporter()));
provider.register();

registerInstrumentations({
  instrumentations: [
    new DocumentLoadInstrumentation(),
  ],
  tracerProvider: provider,
});
```

After saving the file, head back to the browser. By default Parcel should automatically rebuild the application and refresh the page. In the console of the developer toolbar you should start seeing traces being exported:

```js
{traceId: "ab42124a3c573678d4d8b21ba52df3bf", parentId: "af0261e69b996866", name: "documentFetch", id: "a1fbbd7ba725be4f", kind: 0, ...}
{traceId: "ab42124a3c573678d4d8b21ba52df3bf", parentId: "af0261e69b996866", name: "resourceFetch", id: "530cb89307cf4972", kind: 0, ...}
{traceId: "ab42124a3c573678d4d8b21ba52df3bf", parentId: "d21f7bc17caa5aba", name: "documentLoad", id: "af0261e69b996866", kind: 0, ...}
```
