what is wrong here ?

	ManagedByGroup TupleSysIdName `json:"managedByGroup"`

	AdditionalParam map[string]string `json:"additionalParam"`

	SupportedByGroup TupleSysIdName `json:"supportedByGroup"`

 
 	AdditionalParam  types.Map    `tfsdk:"additional_param"`

 func ToApiModel(serverResourceModel ServerResourceModel ) ServerApiModel {
	// ctx := context.Background()
	return ServerApiModel{
		Environment:             serverResourceModel.Environment.ValueString(),

		AdditionalParam:  convertAdditionalParamToAPI(serverResourceModel.AdditionalParam),
		SupportedByGroup: convertGroupToApiModel(serverResourceModel.SupportedByGroup),
		Request:          convertRequestToApiModel(serverResourceModel.Request),
	}
}

}
func convertAdditionalParamToAPI(additionalParam AdditionalParamModel) AdditionalParamApiModel {
    return AdditionalParamApiModel{
        Key:   additionalParam.Key.ValueString(),
        Value: additionalParam.Value.ValueString(),
    }
}

func UpdateModelFromServerAPI(api ServerStateModelAPI, model *ServerResourceModel) {
	ctx := context.Background()

	model.AdditionalParam = convertAdditionalParamFromAPI(ctx,api.AdditionalParam)
	model.SupportedByGroup = convertGroupFromApiModel(api.SupportedByGroup)
	model.Request = convertRequestFromApiModel(api.Request)
}
func convertAdditionalParamFromAPI(ctx context.Context, apiMap map[string]string) types.Map {
    tfMap, _ := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap
}

serverTFSdk := ServerResourceModel{
		// Type:            types.StringValue(serverApiModel.Type),
	
		AdditionalParam:         convertAdditionalParamToTFSdk(ctx,serverApiModel.AdditionalParam),
		SupportedByGroup:        convertGroupToTFSdk(serverApiModel.SupportedByGroup),
	}

	return serverTFSdk, nil
}

func convertAdditionalParamToTFSdk(ctx context.Context, apiMap map[string]string) types.Map {
    tfMap, _ := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap
}


type ServerApiModel struct {
	//Hostname         string            `json:"hostname"`
	RequestId               string              `json:"requestId"`

    AdditionalParam  		AdditionalParamApiModel   `json:"AdditionalParam"`
}
type AdditionalParamApiModel struct {
	Key    string `json:"key"`
	Value  string `json:"value"`
}



func (c *Client) CreateServerAndConvertToTFSdk(ctx context.Context, serverApiModel ServerApiModel) (ServerResourceModel, error) {
	// Step 1: Marshal the input JSON model
	model, err := json.Marshal(serverApiModel)
	if err != nil {
		tflog.Error(ctx, "failed to convert model to API model")
		return ServerResourceModel{}, err
	}

	req, err := http.NewRequest("POST", fmt.Sprintf("%s/oase/servers", c.HostURL), bytes.NewReader(model))
	if err != nil {
		tflog.Error(ctx, "failed to create request for servers post")
		return ServerResourceModel{}, err
	}

	respBytes, err := c.doRequest(ctx, req, &c.AuthResponse.AccessToken)
	if err != nil {
		tflog.Error(ctx, "failed to execute request for servers post")
		return ServerResourceModel{}, err
	}

	tflog.Debug(ctx, string(respBytes))

	// Step 3: Unmarshal the response into a Go struct
	var serverCreateApiResp ServerCreateApiResp
	err = json.Unmarshal(respBytes, &serverCreateApiResp)
	if err != nil {
		tflog.Error(ctx, "failed to unmarshal API response")
		return ServerResourceModel{}, err
	}

	// Step 4: Convert the API response (Go struct) into a tfsdk-compatible model
	serverTFSdk := ServerResourceModel{
		// Type:            types.StringValue(serverApiModel.Type),
		Environment:             types.StringValue(serverApiModel.Environment),
		ServerCode:              types.StringValue(serverApiModel.AcmServerType),

		AdditionalParam:         convertAdditionalParamToTFSdk(ctx,serverApiModel.AdditionalParam),

	}

	return serverTFSdk, nil
}

func convertAdditionalParamToTFSdk(ctx context.Context, apiMap map[string]string) types.Map {
    tfMap, _ := types.MapValueFrom(ctx, types.StringType, apiMap)
    return tfMap
}
