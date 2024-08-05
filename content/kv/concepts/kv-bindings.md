---
pcx_content_type: concept
title: KV bindings
weight: 7
---

# KV bindings

KV [bindings](/workers/runtime-apis/bindings/) allow for communication between a Worker and a KV namespace.

Configure KV bindings in the [wrangler.toml file](/workers/wrangler/configuration/#configure-wranglertoml).

## Reference KV from Workers

A [KV namespace](/kv/concepts/kv-namespaces/) is a key-value database replicated to Cloudflare's global network.

To connect to a KV namespace from within a Worker, you must define a binding that points to the namespace's ID.

The name of your binding does not need to match the KV namespace's name. Instead, the binding should be a valid JavaScript identifier, because the identifier will exist as a global variable within your Worker.

A KV namespace will have a name you choose (for example, `My tasks`), and an assigned ID (for example, `06779da6940b431db6e566b4846d64db`).

To execute your Worker, define the binding.

In the following example, the binding is called `TODO`. In the `kv_namespaces` portion of your `wrangler.toml` file, add:

```toml
---
filename: wrangler.toml
---
name = "worker"

# ...

kv_namespaces = [
  { binding = "TODO", id = "06779da6940b431db6e566b4846d64db" }
]
```

With this, the deployed Worker will have a `TODO` field in their environment object (the second parameter of the `fetch()` request handler). Any methods on the `TODO` binding will map to the KV namespace with an ID of `06779da6940b431db6e566b4846d64db` – which you called `My Tasks` earlier.


```js
export default {
  async fetch(request, env, ctx) {
    // Get the value for the "to-do:123" key
    // NOTE: Relies on the `TODO` KV binding that maps to the "My Tasks" namespace.
    let value = await env.TODO.get("to-do:123");

    // Return the value, as is, for the Response
    return new Response(value);
  },
};
```


When using [`wrangler dev`](/workers/wrangler/commands/#dev) to develop locally, rangler will default to using a local version of KV to avoid interfering with any of your live production data in KV. This means that reading keys that you have not written locally will return null.

To have `wrangler dev` connect to your Workers KV namespace running on Cloudflare's global network, call `wrangler dev --remote` instead. This will use the `preview_id` of the KV binding configuration in the `wrangler.toml` file. See the [KV binding docs](/kv/reference/kv-bindings/) for more information.

## Using the KV binding when developing locally

When you use Wrangler to develop locally with the `wrangler dev` command, Wrangler will default to using a local version of KV to avoid interfering with any of your live production data in KV. This means that reading keys that you have not written locally will return null.

To have `wrangler dev` connect to your Workers KV namespace running on Cloudflare's global network, call `wrangler dev --remote` instead. This will use the `preview_id` of the KV binding configuration in the `wrangler.toml` file. This is how a `wrangler.toml` file looks with the `preview_id` specified.

```toml
---
filename: wrangler.toml
---
name = "worker"

# ...

kv_namespaces = [
  { binding = "TODO", id = "06779da6940b431db6e566b4846d64db", preview_id="06779da6940b431db6e566b484a6a769a7a" }
]
```

## Reference KV from Durable Objects and Workers using ES modules format

[Durable Objects](/durable-objects/) use ES modules format. Instead of a global variable, bindings are available as properties of the `env` parameter [passed to the constructor](/durable-objects/get-started/#3-write-a-class-to-define-a-durable-object).

An example might look like:

```js
export class DurableObject {
  constructor(state, env) {
    this.state = state;
    this.env = env;
  }

  async fetch(request) {
    const valueFromKV = await this.env.NAMESPACE.get("someKey");
    return new Response(valueFromKV);
  }
}
```