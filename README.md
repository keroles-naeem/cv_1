fir map[string]string

Got it ✅ — you’ve decided you want **`AdditionalParam`** to use a simple  
`map[string]string` in your **API model**.  

That’s the cleanest and most flexible approach, so we’ll remove all `AdditionalParamApiModel` structs and make all conversions use `map[string]string`.

***

## **Final Working Setup — map[string]string version**

### **1️⃣ API model**
```go
type ServerApiModel struct {
    Environment      string            `json:"environment"`
    ServerCode       string            `json:"serverCode"`
    AdditionalParam  map[string]string `json:"additionalParam"` // simple map
    // ... other fields ...
}
```

***

### **2️⃣ Terraform model**
```go
type ServerResourceModel struct {
    Environment     types.String `tfsdk:"environment"`
    ServerCode      types.String `tfsdk:"server_code"`
    AdditionalParam types.Map    `tfsdk:"additional_param"` // TF map(string)
    // ... other fields ...
}
```

***

### **3️⃣ Conversion helpers**

#### **TF → API**
```go
import (
    "context"

    "github.com/hashicorp/terraform-plugin-framework/diag"
    "github.com/hashicorp/terraform-plugin-framework/types"
)

func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string, diag.Diagnostics) {
    apiMap := make(map[string]string)
    diags := tfMap.ElementsAs(ctx, &apiMap, false) // false = disallow nulls
    return apiMap, diags
}
```

#### **API → TF**
```go
func convertAdditionalParamFromAPI(ctx context.Context, apiMap map[string]string) (types.Map, diag.Diagnostics) {
    tfMap, diags := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap, diags
}
```

***

### **4️⃣ Example usage in ToApiModel**
```go
func ToApiModel(ctx context.Context, tfModel ServerResourceModel) (ServerApiModel, diag.Diagnostics) {
    apiMap, diags := convertAdditionalParamToAPI(ctx, tfModel.AdditionalParam)

    apiModel := ServerApiModel{
        Environment:     tfModel.Environment.ValueString(),
        ServerCode:      tfModel.ServerCode.ValueString(),
        AdditionalParam: apiMap,
    }

    return apiModel, diags
}
```

***

### **5️⃣ Example usage in UpdateModelFromServerAPI (API → TF)**
```go
func UpdateModelFromServerAPI(ctx context.Context, api ServerApiModel, model *ServerResourceModel) diag.Diagnostics {
    var diags diag.Diagnostics

    model.Environment = types.StringValue(api.Environment)
    model.ServerCode = types.StringValue(api.ServerCode)

    tfMap, d := convertAdditionalParamFromAPI(ctx, api.AdditionalParam)
    diags.Append(d...)
    model.AdditionalParam = tfMap

    return diags
}
```

***

### **6️⃣ Example Create() flow**
```go
func (r *MyResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan ServerResourceModel
    resp.Diagnostics.Append(req.Plan.Get(ctx, &plan)...)
    if resp.Diagnostics.HasError() {
        return
    }

    apiMap, diags := convertAdditionalParamToAPI(ctx, plan.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    apiReq := ServerApiModel{
        Environment:     plan.Environment.ValueString(),
        ServerCode:      plan.ServerCode.ValueString(),
        AdditionalParam: apiMap,
    }

    // Call your API...
    apiResp, err := r.client.CreateServer(apiReq)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    tfMap, diags := convertAdditionalParamFromAPI(ctx, apiResp.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    plan.AdditionalParam = tfMap

    resp.Diagnostics.Append(resp.State.Set(ctx, &plan)...)
}
```

***

✅ **Key takeaways:**
- One single definition for `AdditionalParam` as `map[string]string` in API + `types.Map` in TF.
- Two small symmetric helpers: **`convertAdditionalParamToAPI`** & **`convertAdditionalParamFromAPI`**.
- No more `AdditionalParamApiModel` struct — type mismatch problem gone.
- Safer: always propagate `diag.Diagnostics` when converting.

***

If you want, I can also **go through your mapper.go and replace all old `AdditionalParamApiModel` uses** so it’s 100% consistent with this `map[string]string` approach.

Do you want me to prepare that cleaned-up mapper file for you?

