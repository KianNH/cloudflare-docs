---
_build:
publishResources: false
render: never
list: never
---

{{<details summary="<code>head(key: string): Promise&lt;R2Object | null</code>">}}

**Parameters**

- `key` {{<param-type>}}string{{</param-type>}}

	- The name of the object.

Retrieves the {{<type-link href="#R2Object">}}R2Object{{</type-link>}} for the given `key`, containing only object metadata.

If `key` does not exist in the bucket, this method will return `null`.

**Examples**

{{<details summary="Reading an object's metadata">}}

```js
const object = env.R2.head('dog.png');

console.log(object.etag)
// 9dc673463c0e68b0d7eb86708309f232
```

{{</details>}}

{{</details>}}