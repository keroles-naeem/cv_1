it send type vlaue map it should be string

Ah — I get what you’re saying now.  
Right now, your `convertAdditionalParamToAPI` is returning a **Go `map[string]string`**, but when JSON marshalling it for the API, it’s being sent as a JSON object (a “map” type in JSON), whereas you actually want it to be sent as a **string**.  

That means we shouldn’t be keeping it as a `map[string]string` in the API model — we should convert it to the exact type/format the API expects **before** putting it into the struct.

***

## 1️⃣ Why it’s happening

Right now:

```go
type ServerApiModel struct {
    AdditionalParam map[string]string `json:"additionalParam"`
}
```

When marshalled to JSON, Go turns it into something like:

```json
"additionalParam": {
  "env": "production",
  "version": "8.4"
}
```

That’s a JSON object (map), *not* a string.

***

## 2️⃣ If you need `AdditionalParam` as a single string

If your API field should be a single string **(e.g., JSON-encoded or comma‑separated)**, then you have to **serialize** it before putting it in `ServerApiModel`.

### Option A — Store it directly as a JSON string

```go
type ServerApiModel struct {
    AdditionalParam string `json:"additionalParam"`
}
```

Conversion:

```go
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) string {
    if tfMap.IsNull() || tfMap.IsUnknown() {
        return ""
    }
    apiMap := make(map[string]string)
    _ = tfMap.ElementsAs(ctx, &apiMap, false)

    // Convert the map into a JSON string
    b, _ := json.Marshal(apiMap)
    return string(b)
}
```

If Terraform had this:

```hcl
additional_param = {
  env     = "production"
  version = "8.4"
}
```

Then `AdditionalParam` in JSON would become:

```json
"additionalParam": "{\"env\":\"production\",\"version\":\"8.4\"}"
```

***

### Option B — Store as a comma-separated string

```go
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) string {
    if tfMap.IsNull() || tfMap.IsUnknown() {
        return ""
    }
    apiMap := make(map[string]string)
    _ = tfMap.ElementsAs(ctx, &apiMap, false)

    parts := []string{}
    for k, v := range apiMap {
        parts = append(parts, fmt.Sprintf("%s=%s", k, v))
    }
    return strings.Join(parts, ",")
}
```

That would result in:

```json
"additionalParam": "env=production,version=8.4"
```

***

## 3️⃣ Which to choose?

- **Option A** → if your API expects a JSON *string* containing key-value pairs.
- **Option B** → if your API expects a plain text format like `key1=value1,key2=value2`.
- **Keep `map[string]string`** only if your API really supports JSON objects directly.

***

If you tell me exactly how your API expects `additionalParam` in the request JSON (stringified JSON, plain text, etc.), I can rewrite your mapper so it **outputs exactly that format** and your values don’t get sent as a JSON map.  

How does your API expect it — JSON object or plain string?

