in gitlab

You asked about "in gitlab" in the context of our previous discussion on Terraform debug logging and provider development. Here’s a concise explanation of how Terraform debug logging and troubleshooting relates to GitLab, especially if you are running Terraform within GitLab CI/CD pipelines or using GitLab’s Terraform integration.

***

## Debugging Terraform in GitLab CI/CD

When running Terraform commands inside GitLab pipelines, you can enable debug logging like you do locally by setting environment variables:

- Set the following **in your `.gitlab-ci.yml`** file under the job that runs Terraform:

```yaml
variables:
  TF_LOG: DEBUG
  TF_LOG_PATH: terraform.log  # Optional: store logs in file inside the job workspace
```

This will activate detailed debug logs during your pipeline runs, which are printed to the job log or saved to a file if `TF_LOG_PATH` is set.

***

## Viewing Logs in GitLab

- The Terraform debug output will be part of the **job logs** you see in GitLab’s web UI.
- If you set `TF_LOG_PATH`, the log file will be created as an artifact or within the workspace which you can upload or inspect.

***

## Additional Debugging Tips for GitLab & Terraform Integration

- GitLab has a [Terraform integration](https://docs.gitlab.com/user/infrastructure/iac/terraform_state/) that manages state files. You can check their built-in troubleshooting guide for common issues.
- For **pipeline debugging** in general, GitLab’s docs on [Debugging CI/CD pipelines](https://docs.gitlab.com/ci/debugging/) may help.
- Be sure to also check permissions and tokens if your Terraform provider interacts with API endpoints (like your custom provider).

***

## Summary for Terraform Debug Logging in GitLab

- Use environment variables in GitLab CI jobs:

```yaml
variables:
  TF_LOG: DEBUG
  TF_LOG_PATH: /path/to/terraform-debug.log
```

- Logs appear in GitLab job output or files.
- Use `TRACE` instead of `DEBUG` for more verbose logs.
- Combine with printed debug info inside your custom Terraform provider (e.g., `fmt.Println` or `tflog`).

***

If you want, I can help you with a GitLab CI example `.gitlab-ci.yml` snippet that includes Terraform debug logging configuration relevant to your provider development. Would you like that?

Citations:
[1] How to Debug & Troubleshoot Terraform Projects: Tutorial https://spacelift.io/blog/terraform-debug
[2] Enable logs to debug Terraform https://developer.hashicorp.com/terraform/internals/debugging
[3] How to configure logging for Terraform - Jamie Phillips https://www.phillipsj.net/posts/how-to-configure-logging-for-terraform/
[4] Troubleshooting the Terraform integration with GitLab https://docs.gitlab.com/user/infrastructure/iac/troubleshooting/
[5] Debugging CI/CD pipelines https://docs.gitlab.com/ci/debugging/
[6] Using Terraform with GitLab https://scalr.com/learning-center/using-terraform-with-gitlab/
[7] How to debug terraform https://stackoverflow.com/questions/71774656/how-to-debug-terraform
[8] Log system | GitLab Docs https://docs.gitlab.com/administration/logs/
[9] GitLab-managed Terraform/OpenTofu state https://docs.gitlab.com/user/infrastructure/iac/terraform_state/



export tf_log="debug"

Ah — I see what you’re trying to do.  
You want to **enable Terraform’s debug logging** so you can see what’s happening inside the provider (including the variables you print) when you run Terraform.  

***

## **How to enable `TF_LOG`**

In your terminal, before running `terraform plan` or `terraform apply`, set the `TF_LOG` environment variable, like:

### **Linux / macOS (bash/zsh)**
```bash
export TF_LOG=DEBUG
```
(You can also use `TRACE` for even more details.)

Then run:
```bash
terraform apply
```

### **Windows PowerShell**
```powershell
$env:TF_LOG="DEBUG"
terraform apply
```

***

## **Optional — save logs to a file**
To avoid flooding your terminal:

```bash
export TF_LOG=DEBUG
export TF_LOG_PATH=./terraform.log
terraform apply
```
Now all logs go into `terraform.log` in the current directory.

***

## **In your provider**
Since we added `fmt.Println()` and `json.MarshalIndent()` calls in `Create/Update`, they’ll also appear in the same log output.

**Pro tip:** Terraform sends your provider’s stdout/stderr into its own log. So with `TF_LOG=DEBUG` enabled, you will see both:
- Terraform core debug messages
- Your provider’s print statements

***

If you want, I can show you how to make your **Go provider respect `TF_LOG`** and use the built‑in `tflog.Debug()` instead of raw `fmt.Println()` so your debug messages integrate neatly with Terraform’s logging system.  

Do you want me to give you that logging upgrade?
json imported and not used even i use it?

The error "json imported and not used" in Go occurs when you import the `encoding/json` package but do not actually use it anywhere in your code. Go requires that all imported packages be used; otherwise, it raises a compile-time error.

Here is how you can fix or handle this:

1. **Make sure you use the `json` package where needed:**
   - For example, if you want to print the API request as JSON for debugging:
     ```go
     import (
       "encoding/json"
       "fmt"
       // other imports
     )
     
     // somewhere in your code, before sending API request:
     b, err := json.MarshalIndent(apiReq, "", "  ")
     if err != nil {
         fmt.Println("Error marshalling API request:", err)
     } else {
         fmt.Println("DEBUG API request JSON:\n", string(b))
     }
     ```

2. **If you have the import but don't need to use JSON yet:**
   - You can remove the import to satisfy the compiler.
   - Or, if you want to keep the import for later but avoid errors, you can temporarily do:
     ```go
     import _ "encoding/json"
     ```
     but this is not recommended for `json` as an unused side effect import.

3. **Check if you have commented out or missed the usage of `json` in your debug prints.**
   
4. **In Terraform provider development, you often use JSON marshalling to pretty-print structs or request bodies for debugging, so using `json.MarshalIndent` as shown above is common.**





i want to print all variable send to api

Gotcha — you now want to **print every variable you are sending to the API** so you can debug the exact payload being sent from your Terraform provider.

Because you’re working with the Terraform Plugin Framework, you usually have a **plan** or **state** struct (e.g. `MyResourceModel`) that you pass through your conversion functions before calling your API client.

***

## 1️⃣ Simple case — your API request struct
If you’ve already built your API request struct, you can just `fmt.Printf` it:

```go
fmt.Printf("DEBUG API request payload: %+v\n", apiReq)
```

The `%+v` verb shows **field names and values**.

If you want **JSON** so it looks exactly like what you’d send over the wire:

```go
b, _ := json.MarshalIndent(apiReq, "", "  ")
fmt.Println("DEBUG API request JSON:")
fmt.Println(string(b))
```

***

## 2️⃣ Extracting variables from `plan` (Terraform model)
If you want *all* variables a user set in Terraform before converting to API, just debug‑print the `plan` struct:

```go
fmt.Printf("DEBUG Terraform plan: %+v\n", plan)
```

The catch: `types.Map`, `types.String`, `types.Int64`, etc. don’t pretty‑print as their raw values — you’ll get `Value = {...}`.

So for each `types.Map`, you must extract to a Go type:

```go
if !plan.AdditionalParam.IsNull() {
    var addParams map[string]string
    diags := plan.AdditionalParam.ElementsAs(ctx, &addParams, false)
    if diags.HasError() {
        fmt.Println("Error converting AdditionalParam:", diags)
    }
    fmt.Println("DEBUG AdditionalParam:", addParams)
}
```

***

## 3️⃣ Put it all together — debug in Create/Update before API call

```go
func (r *MyResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan MyResourceModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Extract AdditionalParam map for printing
    var addParams map[string]string
    if !plan.AdditionalParam.IsNull() {
        diags := plan.AdditionalParam.ElementsAs(ctx, &addParams, false)
        resp.Diagnostics.Append(diags...)
    }

    fmt.Println("DEBUG: AdditionalParam to send to API:", addParams)

    // Convert whole plan to API model
    apiMap, _ := convertAdditionalParamToAPI(ctx, plan.AdditionalParam)
    apiReq := ApiModel{
        AdditionalParam: apiMap,
        // other fields...
    }

    // Print full API request as JSON
    b, _ := json.MarshalIndent(apiReq, "", "  ")
    fmt.Println("DEBUG: Full API request JSON:\n", string(b))

    // Call API
    apiResp, err := r.client.CreateResource(apiReq)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    plan.AdditionalParam = convertAdditionalParamToTFSdk(ctx, apiResp.AdditionalParam)

    resp.State.Set(ctx, &plan)
}
```

***

## ✅ Key points

- **Before** calling your API client, log the struct (or JSON) so you see exactly what’s being sent.
- Use `%+v` for field names or `json.MarshalIndent` for pretty JSON.
- For `types.Map` in Terraform Plugin Framework, use `.ElementsAs()` to get a regular Go map for logging/debugging.
- This works the same way in `Create`, `Update`, or even `Delete` if you want to debug deletions.

***

If you want, I can write you a **drop‑in debug utility** that takes *any* `types.Object`/`types.Map` Terraform model and prints all fields recursively as a clean JSON before API calls — that way you always see the exact payload.  

Do you want me to prepare that?

give me the whole code with function and structure

Alright — since you want **the whole working code** for your `additional_param` handling — including the Terraform Plugin Framework structs, schema, and both conversion functions — I’ll give you the complete self‑contained Go code ready to drop into your provider.  

This will allow Terraform configs like:

```hcl
additional_param = {
  sys_id    = "255b..."
  "any key" = "any value"
}
```

…to flow **TF → API → TF** without changing shape.

***

## 📄 **Full Example**

```go
package provider

import (
    "context"

    "github.com/hashicorp/terraform-plugin-framework/diag"
    "github.com/hashicorp/terraform-plugin-framework/resource"
    "github.com/hashicorp/terraform-plugin-framework/schema"
    "github.com/hashicorp/terraform-plugin-framework/types"
)

//
// -------------------
// API Model
// -------------------
//
type ApiModel struct {
    // This matches your API payload exactly
    AdditionalParam map[string]string `json:"AdditionalParam"`
}

//
// -------------------
// Terraform Model
// -------------------
//
type MyResourceModel struct {
    AdditionalParam types.Map `tfsdk:"additional_param"`
}

//
// -------------------
// Terraform Schema
// -------------------
//
func (r *MyResource) Schema(ctx context.Context, req resource.SchemaRequest, resp *resource.SchemaResponse) {
    resp.Schema = schema.Schema{
        Attributes: map[string]schema.Attribute{
            "additional_param": schema.MapAttribute{
                ElementType: types.StringType,
                Required:    true, // or Optional: true
                Description: "A map of arbitrary key-value strings to send to the API.",
            },
        },
    }
}

//
// -------------------
// Conversion Functions
// -------------------
//

// TF -> API
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string, diag.Diagnostics) {
    apiMap := make(map[string]string)
    diags := tfMap.ElementsAs(ctx, &apiMap, false) // false = don't allow nulls
    return apiMap, diags
}

// API -> TF
func convertAdditionalParamToTFSdk(ctx context.Context, apiMap map[string]string) types.Map {
    tfMap, _ := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap
}

//
// -------------------
// Example Resource CRUD
// -------------------
//

type MyResource struct {
    client *MyAPIClient
}

func (r *MyResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan MyResourceModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // TF -> API conversion
    apiMap, diags := convertAdditionalParamToAPI(ctx, plan.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Call API
    apiReq := ApiModel{AdditionalParam: apiMap}
    apiResp, err := r.client.CreateResource(apiReq)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    // API -> TF conversion (store in state)
    plan.AdditionalParam = convertAdditionalParamToTFSdk(ctx, apiResp.AdditionalParam)

    // Save to state
    diags = resp.State.Set(ctx, &plan)
    resp.Diagnostics.Append(diags...)
}

func (r *MyResource) Read(ctx context.Context, req resource.ReadRequest, resp *resource.ReadResponse) {
    var state MyResourceModel
    diags := req.State.Get(ctx, &state)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Call API
    apiResp, err := r.client.GetResource()
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    // API -> TF
    state.AdditionalParam = convertAdditionalParamToTFSdk(ctx, apiResp.AdditionalParam)

    // Save state
    diags = resp.State.Set(ctx, &state)
    resp.Diagnostics.Append(diags...)
}

func (r *MyResource) Update(ctx context.Context, req resource.UpdateRequest, resp *resource.UpdateResponse) {
    var plan MyResourceModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // TF -> API
    apiMap, diags := convertAdditionalParamToAPI(ctx, plan.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    apiReq := ApiModel{AdditionalParam: apiMap}
    apiResp, err := r.client.UpdateResource(apiReq)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    // API -> TF
    plan.AdditionalParam = convertAdditionalParamToTFSdk(ctx, apiResp.AdditionalParam)

    // Save state
    diags = resp.State.Set(ctx, &plan)
    resp.Diagnostics.Append(diags...)
}

func (r *MyResource) Delete(ctx context.Context, req resource.DeleteRequest, resp *resource.DeleteResponse) {
    var state MyResourceModel
    diags := req.State.Get(ctx, &state)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    apiMap, diags := convertAdditionalParamToAPI(ctx, state.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    err := r.client.DeleteResource(apiMap)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    // Terraform will remove it from state automatically after Delete returns
}
```

***

## 💡 How it works
- **In Terraform config**: Any number of `key = value` pairs are allowed; keys can be quoted if needed.
- **In Go**: Stored as `types.Map` in the TF model, converted to/from `map[string]string` for the API.
- **In API**: Sent and received as a regular map in JSON (`object` type).
- **Round‑trip safe**: Whatever user configures goes into state and comes back the same.

***

If you want, I can now add the **minimal `main.go` and provider registration** so you can compile this whole thing as a standalone Terraform provider and test it right away.  

Do you want me to extend it with that?

it expected one argument 
serverResourceModel.CaasRole.ValueString(),

int(serverResourceModel.StorageSize.ValueInt64()),

serverResourceModel.OrganisationUnit.ValueString(),

InstanceName: serverResourceModel. InstanceName. ValueString(),

MysqlVersion: serverResourceModel.MysqlVersion.ValueString(),

CaasRole:

StorageSize:

DomainJoin: serverResourceModel.DomainJoin. ValueBool(),

OrganisationUnit:

OwnedByGroup: convertGroupToApiModel(serverResourceModel.OwnedByGroup), UsedByGroup: convertGroupToApiModel(serverResourceModel.UsedByGroup), ManagedByGroup: convertGroupToApiModel(serverResourceModel.ManagedByGroup), AdditionalParam: convertAdditionalParamToAPI(ctx, serverResourceModel.AdditionalParam), SupportedByGroup: convertGroupToApiModel(serverResourceModel. SupportedByGroup), Request: convertRequestToApiModel(serverResourceModel.Request),

func convertGroupToApiModel(group GroupModel) GroupApiModel {

return GroupApiModel{

SysId: group.SysId.ValueString(), Name: group. Name.ValueString(),

func convertAdditionalParamToAPI(additional_param AdditionalParamModel) AdditionalParamApiMode

// return AdditionalParamApiModel{

Key: additional_param.Key.ValueString(),

Value: additional param. Value.ValueString(),

func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string) {

apiMap := make(map[string]string) diags := tfMap.ElementsAs(ctx, &apiMap, false) // false = don't allow null values

return diags

Ah — I see exactly what’s going wrong here.  
In your snippet you’ve actually got **two different `convertAdditionalParamToAPI` functions** with different signatures, and that’s causing mismatches and “expected one argument” type errors.

***

## **What’s Happening**

From your pasted code bits:

```go
// Version 1 — takes a model
func convertAdditionalParamToAPI(additional_param AdditionalParamModel) AdditionalParamApiModel {
    return AdditionalParamApiModel{
        Key:   additional_param.Key.ValueString(),
        Value: additional_param.Value.ValueString(),
    }
}
```

…and later:

```go
// Version 2 — takes a ctx and a Terraform types.Map
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string) {
    apiMap := make(map[string]string)
    diags := tfMap.ElementsAs(ctx, &apiMap, false)
    return diags
}
```

That’s a **duplicate function name** with **different parameter lists**, which Go does **not** allow (no function overloading).  
Also, your second one is returning `diags` (diagnostics), **not** the `apiMap` it built — so it won’t compile or work the way you expect.

***

## **How to fix this**

First, decide what `AdditionalParam` **really is** and how you want to send it to the API:

### Case A — It’s a `map[string]string` in API
Then the `convertAdditionalParamToAPI` should be only **this version**:

```go
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string, diag.Diagnostics) {
    apiMap := make(map[string]string)
    diags := tfMap.ElementsAs(ctx, &apiMap, false)
    return apiMap, diags
}
```

**Usage:**
```go
apiMap, diags := convertAdditionalParamToAPI(ctx, serverResourceModel.AdditionalParam)
resp.Diagnostics.Append(diags...)
```

***

### Case B — It’s a single Key/Value pair in API  
Then it would look like:

```go
func convertAdditionalParamToAPI(additionalParam AdditionalParamModel) AdditionalParamApiModel {
    return AdditionalParamApiModel{
        Key:   additionalParam.Key.ValueString(),
        Value: additionalParam.Value.ValueString(),
    }
}
```

**Usage:**
```go
apiReq.AdditionalParam = convertAdditionalParamToAPI(serverResourceModel.AdditionalParam)
```

***

## **Why you’re getting “expected one argument”**
Right now in your `InstanceName: ..., CaasRole: ..., AdditionalParam: ...` struct you’re calling:

```go
AdditionalParam: convertAdditionalParamToAPI(ctx, serverResourceModel.AdditionalParam)
```

…but in your file, you have **two** functions named `convertAdditionalParamToAPI` and Go is probably resolving to the wrong one (or failing to compile altogether).

Also, your second version’s signature doesn’t match your call:  
- **Your code calls**: `(ctx, types.Map)` → expects `(context.Context, types.Map)`  
- **Your broken version** returns `diags` instead of the built `map[string]string`.

***

## ✅ Recommended cleanup

1. **Pick one canonical implementation**.  
   Since all earlier discussion was about `map[string]string` (not Key/Value tuple), keep:
   ```go
   func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string, diag.Diagnostics) {
       apiMap := make(map[string]string)
       diags := tfMap.ElementsAs(ctx, &apiMap, false) 
       return apiMap, diags
   }
   ```

2. **Call it properly** in your API request assembly:
   ```go
   apiMap, diags := convertAdditionalParamToAPI(ctx, serverResourceModel.AdditionalParam)
   resp.Diagnostics.Append(diags...)
   apiReq.AdditionalParam = apiMap
   ```

3. **Remove the other `convertAdditionalParamToAPI`** version to avoid compile errors.

***

If you want, I can rewrite your **whole API request builder** (`InstanceName`, `CaasRole`, `StorageSize`, `OrganisationUnit`, etc.) into a clean, compiling block that calls `convertGroupToApiModel` and `convertAdditionalParamToAPI` correctly so you stop getting “expected 1 argument” and mismatched types.  

Do you want me to prepare that final corrected code?




what it will be if we use additionalParamApiModel the whole code

Got it — you want to see **the full working code** if we actually *do* introduce an `AdditionalParamApiModel` type, instead of just using `map[string]string`.  

That means your API would expect something like this JSON structure:

```json
"AdditionalParam": {
    "Key": "sys_id",
    "Value": "255b..."
}
```
…instead of a free‑form map.  
So we’d make a small Go struct for that, and adjust the conversions accordingly.

***

## 1️⃣ API Model Types

```go
// The API uses a single key/value tuple
type AdditionalParamApiModel struct {
    Key   string `json:"Key"`
    Value string `json:"Value"`
}

type ApiModel struct {
    InstanceName     string                   `json:"InstanceName"`
    MysqlVersion     string                   `json:"MysqlVersion"`
    CaasRole         string                   `json:"CaasRole"`
    StorageSize      int                      `json:"StorageSize"`
    DomainJoin       bool                     `json:"DomainJoin"`
    OrganisationUnit string                   `json:"OrganisationUnit"`
    OwnedByGroup     GroupApiModel            `json:"OwnedByGroup"`
    UsedByGroup      GroupApiModel            `json:"UsedByGroup"`
    ManagedByGroup   GroupApiModel            `json:"ManagedByGroup"`
    AdditionalParam  AdditionalParamApiModel  `json:"AdditionalParam"`
    SupportedByGroup GroupApiModel            `json:"SupportedByGroup"`
    Request          RequestApiModel          `json:"Request"`
}
```

***

## 2️⃣ Terraform Model

In TF we still want the HCL to be:

```hcl
additional_param = {
  sys_id = "255b..."
}
```

…but because the API format supports only *one* key/value pair, in your TF **state model** you could either:

- represent `additional_param` as a `map(string)` (and enforce only one key in validation), or  
- represent it directly as two `string` attributes (`key` and `value`).

Here, I’ll keep `map(string)` so your HCL is unchanged:

```go
type MyResourceModel struct {
    AdditionalParam types.Map `tfsdk:"additional_param"`
}
```

***

## 3️⃣ Conversion Functions

### TF → API (map → key/value struct)
Because your map will only have one entry:

```go
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (AdditionalParamApiModel, diag.Diagnostics) {
    apiModel := AdditionalParamApiModel{}
    goMap := make(map[string]string)

    diags := tfMap.ElementsAs(ctx, &goMap, false)
    if diags.HasError() {
        return apiModel, diags
    }

    // Take first key/value from the map
    for k, v := range goMap {
        apiModel.Key = k
        apiModel.Value = v
        break
    }
    return apiModel, diags
}
```

***

### API → TF (key/value struct → map)
```go
func convertAdditionalParamFromAPI(ctx context.Context, apiModel AdditionalParamApiModel) (types.Map, diag.Diagnostics) {
    m := map[string]string{
        apiModel.Key: apiModel.Value,
    }
    tfMap, diags := types.MapValueFrom(ctx, types.StringType, m)
    return tfMap, diags
}
```

***

## 4️⃣ Example usage in a Resource Create

```go
func (r *MyResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan MyResourceModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Convert AdditionalParam: TF -> API
    addParamApi, diags := convertAdditionalParamToAPI(ctx, plan.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Build API request
    apiReq := ApiModel{
        InstanceName:     plan.InstanceName.ValueString(),
        MysqlVersion:     plan.MysqlVersion.ValueString(),
        CaasRole:         plan.CaasRole.ValueString(),
        StorageSize:      int(plan.StorageSize.ValueInt64()),
        DomainJoin:       plan.DomainJoin.ValueBool(),
        OrganisationUnit: plan.OrganisationUnit.ValueString(),
        OwnedByGroup:     convertGroupToApiModel(plan.OwnedByGroup),
        UsedByGroup:      convertGroupToApiModel(plan.UsedByGroup),
        ManagedByGroup:   convertGroupToApiModel(plan.ManagedByGroup),
        AdditionalParam:  addParamApi, // key/value struct
        SupportedByGroup: convertGroupToApiModel(plan.SupportedByGroup),
        Request:          convertRequestToApiModel(plan.Request),
    }

    // Call Create API
    apiResp, err := r.client.CreateResource(apiReq)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    // API -> TF conversion for AdditionalParam
    plan.AdditionalParam, diags = convertAdditionalParamFromAPI(ctx, apiResp.AdditionalParam)
    resp.Diagnostics.Append(diags...)

    diags = resp.State.Set(ctx, &plan)
    resp.Diagnostics.Append(diags...)
}
```

***

## 5️⃣ Key Differences from `map[string]string` version

- `AdditionalParam` in API model is now a **struct** with exactly one `Key` and `Value`.
- The conversion functions wrap/unwrap between `map(string)` (Terraform) and `AdditionalParamApiModel` (API).
- This means **Terraform can still configure it as `{ key = "value" }`**, but your Go code enforces that only one is sent to the API.

***

If you want, I can also give you the **matching `Read()` method** so API → TF is handled in the same style — that way the round‑trip between TF config ↔ API JSON will work perfectly with this `AdditionalParamApiModel` design.

show me the whole code if we don't have additionalParamApiModel

Alright ✅ — if we **don’t** have an `AdditionalParamApiModel` and we’re just treating `AdditionalParam` as a plain `map[string]string` in the API, then the whole thing becomes much simpler.

Here’s the **complete working code** for the API model, TF model, schema, conversion functions, and example CRUD methods, using only a `map[string]string` for `AdditionalParam`.

***

## 📄 Full Example — `map[string]string` version

```go
package provider

import (
    "context"
    "encoding/json"
    "fmt"

    "github.com/hashicorp/terraform-plugin-framework/diag"
    "github.com/hashicorp/terraform-plugin-framework/resource"
    "github.com/hashicorp/terraform-plugin-framework/schema"
    "github.com/hashicorp/terraform-plugin-framework/types"
)

//
// ----------------
// API Models
// ----------------
//
type ApiModel struct {
    InstanceName     string            `json:"InstanceName"`
    MysqlVersion     string            `json:"MysqlVersion"`
    CaasRole         string            `json:"CaasRole"`
    StorageSize      int               `json:"StorageSize"`
    DomainJoin       bool              `json:"DomainJoin"`
    OrganisationUnit string            `json:"OrganisationUnit"`
    OwnedByGroup     GroupApiModel     `json:"OwnedByGroup"`
    UsedByGroup      GroupApiModel     `json:"UsedByGroup"`
    ManagedByGroup   GroupApiModel     `json:"ManagedByGroup"`
    AdditionalParam  map[string]string `json:"AdditionalParam"` // <- simple map
    SupportedByGroup GroupApiModel     `json:"SupportedByGroup"`
    Request          RequestApiModel   `json:"Request"`
}

type GroupApiModel struct {
    SysId string `json:"sys_id"`
    Name  string `json:"name"`
}

type RequestApiModel struct {
    ID   string `json:"id"`
    Type string `json:"type"`
}

//
// ----------------
// Terraform Model
// ----------------
//
type MyResourceModel struct {
    InstanceName     types.String `tfsdk:"instance_name"`
    MysqlVersion     types.String `tfsdk:"mysql_version"`
    CaasRole         types.String `tfsdk:"caas_role"`
    StorageSize      types.Int64  `tfsdk:"storage_size"`
    DomainJoin       types.Bool   `tfsdk:"domain_join"`
    OrganisationUnit types.String `tfsdk:"organisation_unit"`
    OwnedByGroup     GroupModel   `tfsdk:"owned_by_group"`
    UsedByGroup      GroupModel   `tfsdk:"used_by_group"`
    ManagedByGroup   GroupModel   `tfsdk:"managed_by_group"`
    AdditionalParam  types.Map    `tfsdk:"additional_param"` // <- TF map
    SupportedByGroup GroupModel   `tfsdk:"supported_by_group"`
    Request          RequestModel `tfsdk:"request"`
}

type GroupModel struct {
    SysId types.String `tfsdk:"sys_id"`
    Name  types.String `tfsdk:"name"`
}

type RequestModel struct {
    ID   types.String `tfsdk:"id"`
    Type types.String `tfsdk:"type"`
}

//
// ----------------
// Schema
// ----------------
//
func (r *MyResource) Schema(ctx context.Context, req resource.SchemaRequest, resp *resource.SchemaResponse) {
    resp.Schema = schema.Schema{
        Attributes: map[string]schema.Attribute{
            "instance_name": schema.StringAttribute{
                Required: true,
            },
            "mysql_version": schema.StringAttribute{
                Required: true,
            },
            "caas_role": schema.StringAttribute{
                Required: true,
            },
            "storage_size": schema.Int64Attribute{
                Required: true,
            },
            "domain_join": schema.BoolAttribute{
                Optional: true,
            },
            "organisation_unit": schema.StringAttribute{
                Optional: true,
            },
            "additional_param": schema.MapAttribute{
                ElementType: types.StringType,
                Optional:    true,
                Description: "Arbitrary key-value pairs sent to the API",
            },
        },
    }
}

//
// ----------------
// Conversion Functions
// ----------------
//

// TF → API
func convertAdditionalParamToAPI(ctx context.Context, tfMap types.Map) (map[string]string, diag.Diagnostics) {
    apiMap := make(map[string]string)
    diags := tfMap.ElementsAs(ctx, &apiMap, false) // false = disallow nulls
    return apiMap, diags
}

// API → TF
func convertAdditionalParamToTFSdk(ctx context.Context, apiMap map[string]string) types.Map {
    tfMap, _ := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap
}

//
// ----------------
// Example CRUD usage
// ----------------
//

func (r *MyResource) Create(ctx context.Context, req resource.CreateRequest, resp *resource.CreateResponse) {
    var plan MyResourceModel
    diags := req.Plan.Get(ctx, &plan)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Convert AdditionalParam for API
    addParams, diags := convertAdditionalParamToAPI(ctx, plan.AdditionalParam)
    resp.Diagnostics.Append(diags...)
    if resp.Diagnostics.HasError() {
        return
    }

    // Build API request
    apiReq := ApiModel{
        InstanceName:     plan.InstanceName.ValueString(),
        MysqlVersion:     plan.MysqlVersion.ValueString(),
        CaasRole:         plan.CaasRole.ValueString(),
        StorageSize:      int(plan.StorageSize.ValueInt64()),
        DomainJoin:       plan.DomainJoin.ValueBool(),
        OrganisationUnit: plan.OrganisationUnit.ValueString(),
        OwnedByGroup:     convertGroupToApiModel(plan.OwnedByGroup),
        UsedByGroup:      convertGroupToApiModel(plan.UsedByGroup),
        ManagedByGroup:   convertGroupToApiModel(plan.ManagedByGroup),
        AdditionalParam:  addParams, // simple map
        SupportedByGroup: convertGroupToApiModel(plan.SupportedByGroup),
        Request:          convertRequestToApiModel(plan.Request),
    }

    // Debug print
    if b, err := json.MarshalIndent(apiReq, "", "  "); err == nil {
        fmt.Println("DEBUG API request:\n", string(b))
    }

    // Send to API
    apiResp, err := r.client.CreateResource(apiReq)
    if err != nil {
        resp.Diagnostics.AddError("API Error", err.Error())
        return
    }

    // Set state from API response
    plan.AdditionalParam = convertAdditionalParamToTFSdk(ctx, apiResp.AdditionalParam)
    diags = resp.State.Set(ctx, &plan)
    resp.Diagnostics.Append(diags...)
}

func convertGroupToApiModel(g GroupModel) GroupApiModel {
    return GroupApiModel{
        SysId: g.SysId.ValueString(),
        Name:  g.Name.ValueString(),
    }
}

func convertRequestToApiModel(rq RequestModel) RequestApiModel {
    return RequestApiModel{
        ID:   rq.ID.ValueString(),
        Type: rq.Type.ValueString(),
    }
}
```

***

### **Key Points in This Version**
- `AdditionalParam` in both `ApiModel` and `terraform` model is just a **map** (no extra struct).
- Two small helpers handle **TF ↔ API map conversion** (`convertAdditionalParamToAPI` and `convertAdditionalParamToTFSdk`).
- Inside Create/Update/Delete you just call `convertAdditionalParamToAPI(ctx, plan.AdditionalParam)` before sending, and `convertAdditionalParamToTFSdk(ctx, apiResp.AdditionalParam)` when reading.
- Debugging is easy — you can `json.MarshalIndent(apiReq)` to see exactly what’s getting sent.

***

If you want, I can now give you the **matching Read() and Update() methods** so the TF → API → TF round trip is complete with this `map[string]string` approach.  

Do you want me to extend the code with those?

