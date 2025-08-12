7so if you mean you want your Terraform config to look exactly like this:

```hcl
additional_param = {
  key   = "sys_id"
  value = "255b..."
}
```

…then **that’s not a `map` type — it’s a single object** (one set of attributes, `key` and `value`), not a map of multiple objects.

***

## 1️⃣ Terraform variable type

You should declare it as just an **object**, not a map:

```hcl
variable "additional_param" {
  type = object({
    key   = string
    value = string
  })
}
```

***

## 2️⃣ Example usage in `.tfvars`

```hcl
additional_param = {
  key   = "sys_id"
  value = "255b..."
}
```

***

## 3️⃣ Go plugin framework model

If your provider is using the Terraform Plugin Framework, your Go model should be:

```go
type AdditionalParamModel struct {
    AdditionalParamKey   types.String `tfsdk:"key"`
    AdditionalParamValue types.String `tfsdk:"value"`
}
```

Or, if it’s directly under the resource schema:

```go
type AdditionalParamModel struct {
    AdditionalParam types.Object `tfsdk:"additional_param"`
}
```
…but in most cases for a flat form like this, you’ll just have both `key` and `value` as string attributes.

***

## 4️⃣ Schema for one object field

If `"additional_param"` itself is the object:

```go
"additional_param": schema.SingleNestedAttribute{
    Attributes: map[string]schema.Attribute{
        "key": schema.StringAttribute{
            Required: true,
        },
        "value": schema.StringAttribute{
            Required: true,
        },
    },
    Required: true,
},
```

***

## 5️⃣ Converting from API → TF model

If your API model is:

```go
type TupleParam struct {
    Key   string `json:"key"`
    Value string `json:"value"`
}
```

Then conversion is:

```go
func convertAdditionalParamFromApiModel(api TupleParam) types.Object {
    attrTypes := map[string]attr.Type{
        "key":   types.StringType,
        "value": types.StringType,
    }

    objValue, _ := types.ObjectValue(attrTypes, map[string]attr.Value{
        "key":   types.StringValue(api.Key),
        "value": types.StringValue(api.Value),
    })

    return objValue
}
```

***

✅ **Summary:**
- Your example is a **single object**, so use `object({ key=string, value=string })` type in Terraform, not `map`.
- In Go, either represent it as a struct with two `types.String` fields or as a `types.Object`.
- No need for `types.Map` unless you want **multiple key/value entries**.

********************


additional_param = {
  sys_id    = "255b..."
}
or
additional_param = {
  "any key" = "any value"
}
or
additional_param = {
  "any keeey" = "any vaaalue"
}

Yes ✅ — all three of those examples are completely valid Terraform **`map(string)`** values.  

If your variable is declared like this:  

```hcl
variable "additional_param" {
  type = map(string)
}
```

### Then these are valid:

```hcl
additional_param = {
  sys_id = "255b..."
}
```

```hcl
additional_param = {
  "any key" = "any value"
}
```

```hcl
additional_param = {
  "any keeey" = "any vaaalue"
}
```

***

### 💡 Notes
- If your map **keys** have spaces, special characters, or start with a number, you must wrap them in quotes — just like in your `"any key"` example.
- The values are strings, so they must always be quoted.
- You can mix multiple pairs in one map:

```hcl
additional_param = {
  sys_id     = "255b..."
  "any key"  = "any value"
  "special!" = "value with spaces"
}
```

***

### In your Go provider model
Given our earlier schema:

```go
"additional_param": schema.MapAttribute{
    ElementType: types.StringType,
    Required:    true,
},
```

Plugin Framework will map this directly to:
```go
map[string]string
```

via:
```go
apiMap := make(map[string]string)
diags := tfMap.ElementsAs(ctx, &apiMap, false)
```

***

If you’d like, I can now show you the **round-trip** example for these inputs — from Terraform config → provider → API → provider → Terraform — so you can see they stay identical.  
Do you want me to do that?



