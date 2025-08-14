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

-----------------------------------------------------------------------------------------------------
--- FAIL: TestToApiModel (0.00s)
panic: runtime error: invalid memory address or nil pointer dereference [recovered]
	panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x30 pc=0x729fd2]
goroutine 47 [running]:
testing.tRunner.func1.2({0x7968a0, 0xae1620})
	/usr/local/go/src/testing/testing.go:1631 +0x24a
testing.tRunner.func1()
	/usr/local/go/src/testing/testing.go:1634 +0x377
panic({0x7968a0?, 0xae1620?})
	/usr/local/go/src/runtime/panic.go:770 +0x132
github.com/hashicorp/terraform-plugin-framework/types/basetypes.MapValue.ToTerraformValue({0x0?, {0x0?, 0x0?}, 0x1c?}, {0x8a7b20, 0xb566a0})
	/go/pkg/mod/github.com/hashicorp/terraform-plugin-framework@v1.13.0/types/basetypes/map_value.go:215 +0x52
github.com/hashicorp/terraform-plugin-framework/types/basetypes.MapValue.ElementsAs({0x0?, {0x0?, 0x0?}, 0x3?}, {0x8a7b20, 0xb566a0}, {0x778700, 0xc000112230}, 0x0)
	/go/pkg/mod/github.com/hashicorp/terraform-plugin-framework@v1.13.0/types/basetypes/map_value.go:186 +0x6f
terraform-provider-oase/internal/provider.convertAdditionalParamToAPI(...)
	/builds/oase/terraform-provider-oase/internal/provider/mapper.go:67
terraform-provider-oase/internal/provider.ToApiModel({_, _}, {{0x0, {0x0, 0x0}}, {0x0, {0x0, 0x0}}, {0x2, {0x80139d, ...}}, ...})
	/builds/oase/terraform-provider-oase/internal/provider/mapper.go:12 +0x116
terraform-provider-oase/internal/provider.TestToApiModel(0xc000117520)
	/builds/oase/terraform-provider-oase/internal/provider/mapper_test.go:76 +0x585
testing.tRunner(0xc000117520, 0x838f58)
	/usr/local/go/src/testing/testing.go:1689 +0xfb
created by testing.(*T).Run in goroutine 1
	/usr/local/go/src/testing/testing.go:1742 +0x390
FAIL	terraform-provider-oase/internal/provider	0.029s
func TestToApiModel(t *testing.T) {
	ctx := context.Background()
	input := ServerResourceModel{
		Environment:             types.StringValue("production"),
		OperatingSystem:         types.StringValue("linux"),
		BusinessServiceOffering: types.StringValue("web"),
		Location:                types.StringValue("us-east"),
		Zone:                    types.StringValue("zone1"),
		Tier:                    types.StringValue("tier1"),
		Cpus:                    types.Int64Value(4),
		Ram:                     types.Int64Value(8),
		ApplicationService:      types.StringValue("app-svc"),
		TshirtSize:              types.StringValue("L"),
		RequestId:               types.StringValue("req-123"),
		RequestUser: RequestUser{
			SysId:   types.StringValue("sys-123"),
			LoginId: types.StringValue("tester"),
		},
		Tenant:            types.StringValue("tenant1"),
		AccessibilityIvbb: types.BoolValue(true),
		Fabric:            types.StringValue("fabric1"),
		Interfaces: []InterfaceItemModel{
			{
				IfType:         types.StringValue("mgmt"),
				NetworkAddress: types.StringValue("10.1.1.0/24"),
			},
			{
				IfType:         types.StringValue("prod"),
				NetworkAddress: types.StringValue("10.2.2.0/24"),
			},
		},
		OwnedByGroup: GroupModel{
			SysId: types.StringValue("og-1"),
			Name:  types.StringValue("OwnedGroup"),
		},
		UsedByGroup: GroupModel{
			SysId: types.StringValue("ug-1"),
			Name:  types.StringValue("UsedGroup"),
		},
		ManagedByGroup: GroupModel{
			SysId: types.StringValue("mg-1"),
			Name:  types.StringValue("ManagedGroup"),
		},
		SupportedByGroup: GroupModel{
			SysId: types.StringValue("sg-1"),
			Name:  types.StringValue("SupportedGroup"),
		},
		Hdd: []HddItemModel{
			{
				Mountpoint: types.StringValue("hdd1"),
				SizeGB:     types.Int64Value(100),
			},
		},
		CatalogItemName: types.StringValue("catalog_item_A"),
		ProjectNumber:   types.StringValue("PRJ0001518"),

		Backup: types.StringValue("Filesicherung"),

		InstanceName:     types.StringValue("TESTYE25"),
		MysqlVersion:     types.StringValue("8.4"),
		CaasRole:         types.StringValue("k8s.workload"),
		StorageSize:      types.Int64Value(100),
		DomainJoin:       types.BoolValue(true),
		OrganisationUnit: types.StringValue("String"),
	}

	result := ToApiModel(ctx,input )

	// Assert basic fields
	assert.Equal(t, "production", result.Environment)
	assert.Equal(t, "linux", result.OperatingSystem)
	assert.Equal(t, "web", result.BusinessServiceOffering)
	assert.Equal(t, "us-east", result.Location)
	assert.Equal(t, "zone1", result.Zone)
	assert.Equal(t, "tier1", result.Tier)
	assert.Equal(t, 4, result.Cpus)
	assert.Equal(t, 8, result.Ram)
	assert.Equal(t, "app-svc", result.ApplicationService)
	assert.Equal(t, "L", result.TSize)
	assert.Equal(t, "sys-123", result.RequestUser.SysId)
	assert.Equal(t, "tester", result.RequestUser.LoginId)
	assert.Equal(t, "tenant1", result.Tenant)
	assert.True(t, result.AccessibilityIvbb)
	assert.Equal(t, "fabric1", result.Fabric)

	// Assert Interfaces
	assert.Len(t, result.Interfaces, 2)
	assert.Equal(t, "mgmt", result.Interfaces[0].IfType)
	assert.Equal(t, "10.1.1.0/24", result.Interfaces[0].NetworkAddress)
	assert.Equal(t, "prod", result.Interfaces[1].IfType)
	assert.Equal(t, "10.2.2.0/24", result.Interfaces[1].NetworkAddress)

	assert.Len(t, result.Hdd, 1)
	assert.Equal(t, "hdd1", result.Hdd[0].Name)
	assert.Equal(t, 100, result.Hdd[0].Size)
	assert.Equal(t, "catalog_item_A", result.CatalogItemName)
	assert.Equal(t, "PRJ0001518", result.ProjectNumber)

	assert.Equal(t, "Filesicherung", result.Backup)

	assert.Equal(t, "TESTYE25", result.InstanceName)
	assert.Equal(t, "8.4", result.MysqlVersion)
	assert.Equal(t, "k8s.workload", result.CaasRole)
	assert.Equal(t, 100, result.StorageSize)
	assert.Equal(t, true, result.DomainJoin)
	assert.Equal(t, "String", result.OrganisationUnit)

	// Test with some empty values
	input_with_some_empty_values := ServerResourceModel{
		Environment:             types.StringValue("production"),
		OperatingSystem:         types.StringValue("linux"),
		BusinessServiceOffering: types.StringValue("web"),
		Location:                types.StringValue("us-east"),
		Zone:                    types.StringValue("zone1"),
		Tier:                    types.StringValue("tier1"),
		Cpus:                    types.Int64Value(4),
		Ram:                     types.Int64Value(2),
		ApplicationService:      types.StringValue(""),
		TshirtSize:              types.StringValue(""),
		RequestId:               types.StringValue(""),
		RequestUser: RequestUser{
			SysId:   types.StringValue(""),
			LoginId: types.StringValue(""),
		},
		Tenant:            types.StringValue(""),
		AccessibilityIvbb: types.BoolValue(false),
		Fabric:            types.StringValue(""),
		Interfaces:        []InterfaceItemModel{},
		Hdd: []HddItemModel{
			{
				Mountpoint: types.StringValue("hdd1"),
				SizeGB:     types.Int64Value(100),
			},
		},
		CatalogItemName: types.StringValue(""),
		ProjectNumber:   types.StringValue(""),

		Backup: types.StringValue(""),

		InstanceName:     types.StringValue(""),
		MysqlVersion:     types.StringValue(""),
		CaasRole:         types.StringValue(""),
		StorageSize:      types.Int64Value(100),
		DomainJoin:       types.BoolValue(true),
		OrganisationUnit: types.StringValue(""),
	}

	result = ToApiModel(ctx,input_with_some_empty_values )

	// Assert basic fields
	assert.Equal(t, "production", result.Environment)
	assert.Equal(t, "linux", result.OperatingSystem)
	assert.Equal(t, "web", result.BusinessServiceOffering)
	assert.Equal(t, "us-east", result.Location)
	assert.Equal(t, "zone1", result.Zone)
	assert.Equal(t, "tier1", result.Tier)
	assert.Equal(t, 4, result.Cpus)
	assert.Equal(t, 2, result.Ram)
	assert.Equal(t, "", result.ApplicationService)
	assert.Equal(t, "", result.TSize)
	assert.Equal(t, "", result.RequestId)
	assert.Equal(t, RequestUserApiModel{SysId: "", LoginId: ""}, result.RequestUser)
	assert.Equal(t, "", result.Tenant)
	assert.False(t, result.AccessibilityIvbb)
	assert.Equal(t, "", result.Fabric)
	assert.Len(t, result.Interfaces, 0)

	assert.Len(t, result.Hdd, 1)
	assert.Equal(t, "hdd1", result.Hdd[0].Name)
	assert.Equal(t, 100, result.Hdd[0].Size)
	assert.Equal(t, "", result.CatalogItemName)
	assert.Equal(t, "", result.ProjectNumber)

	assert.Equal(t, "", result.Backup)

	assert.Equal(t, "", result.InstanceName)
	assert.Equal(t, "", result.MysqlVersion)
	assert.Equal(t, "", result.CaasRole)
	assert.Equal(t, 100, result.StorageSize)
	assert.Equal(t, true, result.DomainJoin)
	assert.Equal(t, "", result.OrganisationUnit)
}

func TestConvertInterfaceModelModel(t *testing.T) {
	input := []InterfaceItemModel{
		{
			IfType:         types.StringValue("mgmt"),
			NetworkAddress: types.StringValue("10.1.1.0/24"),
		},
		{
			IfType:         types.StringValue("prod"),
			NetworkAddress: types.StringValue("10.2.2.0/24"),
		},
	}
	expected := []InterfaceApiModel{
		{
			IfType:         "mgmt",
			NetworkAddress: "10.1.1.0/24",
		},
		{
			IfType:         "prod",
			NetworkAddress: "10.2.2.0/24",
		},
	}
	result := convertInterfacesToApiModel(input)
	assert.Equal(t, expected, result)
	// Test with empty input
	result = convertInterfacesToApiModel([]InterfaceItemModel{})
	assert.Nil(t, result)
}

func TestConvertHddToApiModel(t *testing.T) {
	// Test with non-empty input
	input := []HddItemModel{
		{
			Mountpoint: types.StringValue("disk1"),
			SizeGB:     types.Int64Value(100),
		},
		{
			Mountpoint: types.StringValue("disk2"),
			SizeGB:     types.Int64Value(200),
		},
	}

	result := convertHddToApiModel(input)

	assert.Len(t, result, 2)
	assert.Equal(t, "disk1", result[0].Name)
	assert.Equal(t, 100, result[0].Size)
	// assert.Equal(t, "disk2", result[1].Name)
	// assert.Equal(t, 200, result[1].Size)
	// Test with empty input
	emptyResult := convertHddToApiModel([]HddItemModel{})
	assert.Nil(t, emptyResult)
}
