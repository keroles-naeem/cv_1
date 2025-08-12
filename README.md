yes everything

Alright — let’s put **everything together** into a complete working round‑trip so your  
`additional_param` behaves exactly as you want:

> ✅ **Terraform user can write any key(s) and any string value(s)** — like  
> ```hcl
> additional_param = {
>   sys_id    = "255b..."
>   "any key" = "any value"
> }
> ```

***

## **1. Terraform variable definition**

If you’re using a variable for this:

```hcl
variable "additional_param" {
  type = map(string)
}
```

If it’s defined directly in a resource, you don’t need the variable block — you just use a map in the resource’s attribute.

***

## **2. Example Terraform usage**

```hcl
resource "my_resource" "example" {
  additional_param = {
    sys_id    = "255b..."
    "any key" = "any value"
    "another" = "hello"
  }
}
```

***

## **3. Go Plugin Framework model**

```go
type AdditionalParamModel struct {
    AdditionalParam types.Map `tfsdk:"additional_param"`
}
```

***

## **4. Schema in your provider**

```go
"additional_param": schema.MapAttribute{
    ElementType: types.StringType,
    Required:    true, // or Optional: true, if not required
},
```

***

## **5. Converting from Terraform → API (map[string]string)**

```go
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string, diag.Diagnostics) {
    apiMap := make(map[string]string)
    diags := tfMap.ElementsAs(ctx, &apiMap, false)
    return apiMap, diags
}
```

***

## **6. Converting from API → Terraform (`types.Map`)**

```go
func convertAdditionalParamFromAPI(ctx context.Context, apiMap map[string]string) types.Map {
    tfMap, _ := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap
}
```

***

## **7. Full round‑trip flow:**

**Terraform config → Provider model → API payload:**

1. User writes in `.tf`:
   ```hcl
   additional_param = {
     sys_id     = "255b..."
     "any key"  = "any value"
     "another"  = "val"
   }
   ```

2. Terraform framework passes this into your model:
   ```go
   AdditionalParam types.Map
   ```

3. You call `convertAdditionalParamToAPI()` to get:
   ```go
   map[string]string{
     "sys_id":    "255b...",
     "any key":   "any value",
     "another":   "val",
   }
   ```
   — ready to send to your API.

***

**API payload → Provider model → Terraform state:**

1. API sends JSON:
   ```json
   {
     "sys_id": "255b...",
     "any key": "any value",
     "another": "val"
   }
   ```

2. You call `convertAdditionalParamFromAPI()` to create a `types.Map` from `map[string]string`.

3. It’s stored back in Terraform state, so next `terraform plan` will show exactly what user entered.

***

✅ This setup **fully supports**:
- Any key names (with or without spaces, quotes if needed).
- Any string values.
- Multiple pairs.
- Round‑trip between TF config → provider → API → provider → TF config unchanged.

***

If you want, I can also give you a **copy‑paste complete Go resource/example** that wires the schema, model, and conversions together so it works in a real Terraform provider.  

Do you want me to prepare that full example?

