but it's
AdditionalParam types.Map `tfsdk:"additional_param

Ah — now I see.  
You’re clarifying that in your Go **Terraform Plugin Framework** model, the `AdditionalParam` field itself is defined as a **`types.Map`**, not as two separate `types.String` values.  

That changes how your conversion function should be implemented — because if the field is `types.Map` in the provider model, you can’t just assign `types.StringValue(...)` to it. You need to **construct a `types.Map` of nested objects** that matches your HCL type definition.

***

## 1️⃣ Your Terraform HCL type
From earlier conversation, it sounds like your schema is something like:

```hcl
variable "additional_param" {
  type = map(object({
    key   = string
    value = string
  }))
}
```

***

## 2️⃣ Matching Go model
To represent this as a **`types.Map` of nested objects** in your Go Plugin Framework model:

```go
type AdditionalParamModel struct {
    AdditionalParam types.Map `tfsdk:"additional_param"`
}
```

***

## 3️⃣ Schema definition in provider
In your provider resource or data source schema:

```go
"additional_param": schema.MapNestedAttribute{
    NestedObject: schema.NestedAttributeObject{
        Attributes: map[string]schema.Attribute{
            "key": schema.StringAttribute{
                Required: true,
            },
            "value": schema.StringAttribute{
                Required: true,
            },
        },
    },
    Optional: true,
},
```

***

## 4️⃣ Converting from API model
Let’s say your API sends you this Go type:

```go
type AdditionalParamApiModel struct {
    Key   string `json:"key"`
    Value string `json:"value"`
}
```

And you have `map[string]AdditionalParamApiModel` from the API.

You’d build the `types.Map` like:

```go
func convertAdditionalParamToTFSdk(apiMap map[string]AdditionalParamApiModel) types.Map {
    // Build a map of attribute values
    tfMap := make(map[string]attr.Value)

    for k, v := range apiMap {
        objValue, _ := types.ObjectValue(
            map[string]attr.Type{
                "key":   types.StringType,
                "value": types.StringType,
            },
            map[string]attr.Value{
                "key":   types.StringValue(v.Key),
                "value": types.StringValue(v.Value),
            },
        )
        tfMap[k] = objValue
    }

    // Convert to Terraform Plugin Framework MapValue
    result, _ := types.MapValue(
        types.ObjectType{
            AttrTypes: map[string]attr.Type{
                "key":   types.StringType,
                "value": types.StringType,
            },
        },
        tfMap,
    )

    return result
}
```

Then your overall conversion into your model is:

```go
model := AdditionalParamModel{
    AdditionalParam: convertAdditionalParamToTFSdk(apiAdditionalParamMap),
}
```

***

✅ **Key point:**  
Because `AdditionalParam` is a `types.Map` in your Go model, you **must** build a `types.MapValue()` from `map[string]attr.Value`, where each value is itself a Terraform `ObjectValue` representing `{ key, value }`.

***

If you want, I can give you **the inverse function** too — to convert from `types.Map` coming from Terraform into your `map[string]AdditionalParamApiModel` for sending back to the API.  

Do you want me to prepare that as well? That way your mapping works in both directions.

