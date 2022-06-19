---
pcx-content-type: configuration
title: Runtime APIs
---

# R2

The in-Worker R2 API is accessed by binding an R2 bucket to a [Worker](/workers). The Worker you write can expose external access to  buckets via a route or manipulate R2 objects internally.

The R2 API includes some extensions and semantic differences from the S3 API. If you need S3 compatibility, consider using the [S3-compatible API](/r2/platform/s3-compatibility/).

## Concepts

R2 organizes the data you store, called objects, into containers, called buckets. Buckets are the fundamental unit of performance, scaling, and access within R2.

## Create a binding

{{<Aside type="note" header="Bindings">}}

A binding is a how your Worker interacts with external resources such as [KV Namespaces](/workers/runtime-apis/kv/), [Durable Objects](/workers/runtime-apis/durable-objects/), or [R2 Buckets](#api). A binding is a runtime variable that the Workers runtime provides to your code. You can declare a variable name in your `wrangler.toml` file that will be bound to these resources at runtime, and interact with them through this variable. Every binding's variable name and behavior is determined by you when deploying the Worker. Refer to [Environment Variables](/workers/platform/environment-variables) for more information.

A binding is defined in the `wrangler.toml` file of your Worker project's directory.

{{</Aside>}}

To bind your R2 bucket to your Worker, add the following to your `wrangler.toml` file. Update the `binding` property to a valid JavaScript variable identifier and `bucket_name` to the name of your R2 bucket:

```toml
r2_buckets = [
  { binding = 'MY_BUCKET', bucket_name = 'BUCKET_NAME' }
]
```

Within your Worker, your bucket binding is now available under the `MY_BUCKET` variable and you can begin interacting with it using the [bucket methods](#bucket-methods) described below.

## Bucket method definitions

The following methods are available on the bucket binding object injected into your code.

For example, to issue a `PUT` object request using the binding above:

```js
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const key = url.pathname.slice(1);

    switch (request.method) {
      case "PUT": {
        await env.MY_BUCKET.put(key, request.body);
        return new Response(`Put ${key} successfully!`);
      }

      default: {
        return new Response(`${request.method} is not allowed.`, {
          status: 405,
          headers: {
            Allow: 'PUT'
          }
        })
      }
    }
  }
}
```

{{<definitions>}}

- {{<code>}}head(key{{<param-type>}}string{{</param-type>}}){{</code>}} {{<type>}}Promise\<R2Object | null\>{{</type>}}

  - Retrieves the {{<type-link href="#r2object-definition">}}R2Object{{</type-link>}} for the given key containing only object metadata, if the key exists, and null if the key does not exist.

- {{<code>}}get(key{{<param-type>}}string{{</param-type>}}, options{{<param-type>}}R2GetOptions{{</param-type>}}{{<prop-meta>}}optional{{</prop-meta>}}) {{<type>}}Promise\<{{<param-type>}}R2ObjectBody{{</param-type>}}|{{<param-type>}}R2Object{{</param-type>}}|{{<param-type>}}null{{</param-type>}}>{{</type>}}{{</code>}}

  - Retrieves the {{<type-link href="#r2object-definition">}}R2Object{{</type-link>}} for the given key containing object metadata and the object body as a {{<type-link href="https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream">}}ReadableStream{{</type-link>}}, if the key exists, and `null` if the key does not exist.
  - In the event that a precondition specified in `options` fails, `get()` returns an {{<type-link href="#r2object-definition">}}R2Object{{</type-link>}} with `body` undefined.

- {{<code>}}put(key{{<param-type>}}string{{</param-type>}}, value{{<param-type>}}ReadableStream{{</param-type>}}|{{<param-type>}}ArrayBuffer{{</param-type>}}|{{<param-type>}}ArrayBufferView{{</param-type>}}|{{<param-type>}}string{{</param-type>}}|{{<param-type>}}null{{</param-type>}}|{{<param-type>}}Blob{{</param-type>}}, options{{<param-type>}}R2PutOptions{{</param-type>}}{{<prop-meta>}}optional{{</prop-meta>}}){{</code>}} {{<type>}}Promise\<R2Object\>{{</type>}}

  - Stores the given `value` and metadata under the associated `key`. Once the write succeeds, returns an {{<type-link href="#r2object-definition">}}R2Object{{</type-link>}} containing metadata about the stored Object.
  - R2 writes are strongly consistent. Once the Promise resolves, all subsequent read operations will see this key value pair globally.

- {{<code>}}delete(key{{<param-type>}}string{{</param-type>}}){{</code>}} {{<type>}}Promise\<void\>{{</type>}}

  - Deletes the given `value` and metadata under the associated `key`. Once the delete succeeds, returns `void`.
   - R2 deletes are strongly consistent. Once the Promise resolves, all subsequent read operations will no longer see this key value pair globally.

- {{<code>}}list(options {{<type-link href="#r2listoptions">}}R2ListOptions{{</type-link>}}{{<prop-meta>}}optional{{</prop-meta>}}){{</code>}} {{<type>}}Promise\<R2Objects | null\>{{</type>}}

  - Returns an {{<type-link href="#r2objects">}}R2Objects{{</type-link>}} containing a list of {{<type-link href="#r2object-definition">}}R2Object{{</type-link>}} contained within the bucket. By default, returns the first 1000 entries.

{{</definitions>}}

## `R2Object` definition

`R2Object` is created when you `PUT` an object into an R2 bucket. `R2Object` represents the metadata of an object based on the information provided by the uploader. Every object that you `PUT` into an R2 bucket will have an `R2Object` created.

{{<definitions>}}

- {{<code>}}key{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - The object's key.

- {{<code>}}body{{</code>}}{{<param-type>}}ReadableStream{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - The object's value.

- {{<code>}}bodyUsed{{</code>}}{{<param-type>}}boolean{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - Whether the object's value has been consumed or not.

- {{<code>}}arrayBuffer(){{</code>}} {{<type>}}Promise\<ArrayBuffer\>{{</type>}}

  - Returns a Promise that resolves to an `ArrayBuffer` containing the object's value.

- {{<code>}}text(){{</code>}} {{<type>}}Promise\<string\>{{</type>}}

  - Returns a Promise that resolves to a string containing the object's value.

- {{<code>}}json<T>(){{</code>}} {{<type>}}Promise\<T\>{{</type>}}

  - Returns a Promise that resolves to the given object containing the object's value.

- {{<code>}}blob(){{</code>}} {{<type>}}Promise\<Blob\>{{</type>}}

  - Returns a Promise that resolves to a binary Blob containing the object's value.

- {{<code>}}version{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - Random unique string associated with a specific upload of a key.

- {{<code>}}size{{</code>}}{{<param-type>}}number{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - Size of the object in bytes.

- {{<code>}}etag{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - The etag associated with the object upload.

- {{<code>}}httpEtag{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - The object's etag, in quotes so as to be returned as a header.

- {{<code>}}uploaded{{</code>}}{{<param-type>}}Date{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - A Date object representing the time the object was uploaded.

- {{<code>}}httpMetadata{{</code>}} {{<type-link href="#http-metadata">}}R2HTTPMetadata{{</type-link>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - Various HTTP headers associated with the object. Refer to [HTTP Metadata](#http-metadata).

- {{<code>}}customMetadata{{</code>}}{{<param-type>}}Record\<string, string>{{</param-type>}} {{<prop-meta>}}readonly{{</prop-meta>}}

  - A map of custom, user-defined metadata associated with the object.

- {{<code>}}writeHttpMetadata(headers{{<param-type>}}Headers{{</param-type>}}){{</code>}} {{<type>}}void{{</type>}}

  -  Retrieves the {{<type-link href="#http-metadata">}}R2HTTPMetadata{{</type-link>}} from the {{<type-link href="#r2object">}}R2Object{{</type-link>}} and applies their corresponding HTTP headers to the `Headers` input object. Refer to [HTTP Metadata](#http-metadata).

{{</definitions>}}

## Method-specific types

### R2GetOptions

{{<definitions>}}

- {{<code>}}onlyIf{{</code>}} {{<type-link href="#conditional-operations">}}R2Conditional{{</type-link>}}

  - Specifies that the object should only be returned given satisfaction of certain conditions in the {{<type-link href="#conditional-operations">}}R2Conditional{{</type-link>}}.

- {{<code>}}range{{</code>}} {{<type-link href="#ranged-reads">}}R2Range{{</type-link>}}

  - Specifies that only a specific length (from an optional offset) or suffix of bytes from the object should be returned. Refer to [Ranged reads](#ranged-reads).

{{</definitions>}}

#### Ranged reads

`R2GetOptions` accepts a `range` parameter, which can be used to restrict the data returned in `body`.

There are 3 variations of arguments that can be used in a range:

* An offset with an optional length.
* An optional offset with a length.
* A suffix.

{{<definitions>}}

- {{<code>}}offset{{</code>}}{{<param-type>}}number{{</param-type>}}

  - The byte to begin returning data from, inclusive.

- {{<code>}}length{{</code>}}{{<param-type>}}number{{</param-type>}}

  - The number of bytes to return. If more bytes are requested than exist in the object, fewer bytes than this number may be returned.

- {{<code>}}suffix{{</code>}}{{<param-type>}}number{{</param-type>}}

  - The number of bytes to return from the end of the file, starting from the last byte. If more bytes are requested than exist in the object, fewer bytes than this number may be returned.

{{</definitions>}}

### R2PutOptions

{{<definitions>}}

- {{<code>}}httpMetadata{{</code>}}{{<param-type>}}R2HTTPMetadata | Headers{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - Various HTTP headers associated with the object. Refer to [HTTP Metadata](#http-metadata).

- {{<code>}}customMetadata{{</code>}}{{<param-type>}}Record\<string, string\>{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - A map of custom, user-defined metadata that will be stored with the object.

- {{<code>}}md5{{</code>}}{{<param-type>}}ArrayBuffer | string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - A md5 hash to use to check the received object's integrity.

{{</definitions>}}

### R2ListOptions

{{<definitions>}}

- {{<code>}}limit{{</code>}}{{<param-type>}}number{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}
  - The number of results to return. Defaults to `1000`, with a maximum of `1000`.

- {{<code>}}prefix{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}
  - The prefix to match keys against. Keys will only be returned if they start with given prefix.

- {{<code>}}cursor{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}
  - An opaque token that indicates where to continue listing objects from. A cursor can be retrieved from a previous list operation.

- {{<code>}}delimiter{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}
  - The character to use when grouping keys.

- {{<code>}}include{{</code>}}{{<param-type>}}Array\<string\>{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}
  - Can include `httpMetadata` and/or `customMetadata`. If included, items returned by the list will include the specified metadata.

  - Note that there is a limit on the total amount of data that a single `list` operation can return. If you request data, you may receive fewer than `limit` results in your response to accommodate metadata.

  This means applications must be careful to avoid code like the following:

  ```js
  while (listed.length < limit) {
    listed = env.MY_BUCKET.list({ limit, include: ['customMetadata'] })
  }
  ```
  Instead, use the `truncated` property to determine if the `list` request has more data to be returned.

{{</definitions>}}

### R2Objects

An object containing an `R2Object` array, returned by `BUCKET_BINDING.list()`.

{{<definitions>}}

- {{<code>}}objects{{</code>}}{{<param-type>}}Array\<R2Object\>{{</param-type>}}

  - An array of objects matching the `list` request.

- {{<code>}}truncated{{</code>}}{{<param-type>}}boolean{{</param-type>}}

  - If true, indicates there are more results to be retrieved for the current `list` request.

- {{<code>}}cursor{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - A token that can be passed to future `list` calls to resume listing from that point. Only present if truncated is true.

- {{<code>}}delimitedPrefixes{{</code>}}{{<param-type>}}Array\<string\>{{</param-type>}}

  - If a delimiter has been specified, contains all prefixes between the specified prefix and the next occurence of the delimiter.

  - For example, if no prefix is provided and the delimiter is `/`, `foo/bar/baz` would return `foo` as a delimited prefix. If `foo/` was passed as a prefix with the same structure and delimiter, `foo/bar` would be returned as a delimited prefix.

{{</definitions>}}

### Conditional operations

You can pass an `R2Conditional` object to `R2GetOptions`.  If the condition check fails, the body will not be returned. This will make `get()` have lower latency.

{{<definitions>}}

- {{<code>}}etagMatches{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - Performs the operation if the object's etag matches the given string.

- {{<code>}}etagDoesNotMatch{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - Performs the operation if the object's etag does not match the given string.

- {{<code>}}uploadedBefore{{</code>}}{{<param-type>}}Date{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - Performs the operation if the object was uploaded before the given date.

- {{<code>}}uploadedAfter{{</code>}}{{<param-type>}}Date{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

  - Performs the operation if the object was uploaded after the given date.

 {{</definitions>}}

For more information about conditional requests, refer to [RFC 7232](https://datatracker.ietf.org/doc/html/rfc7232).

### HTTP Metadata

Generally, these fields match the HTTP metadata passed when the object was created.  They can be overridden when issuing `GET` requests, in which case, the given values will be echoed back in the response.

{{<definitions>}}

- {{<code>}}contentType{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

- {{<code>}}contentLanguage{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

- {{<code>}}contentDisposition{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

- {{<code>}}contentEncoding{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

- {{<code>}}cacheControl{{</code>}}{{<param-type>}}string{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

- {{<code>}}cacheExpiry{{</code>}}{{<param-type>}}Date{{</param-type>}} {{<prop-meta>}}optional{{</prop-meta>}}

 {{</definitions>}}
