From here:
https://learn.microsoft.com/en-us/azure/azure-functions/functions-deploy-container-apps?tabs=acr%2Cbash&pivots=programming-language-csharp

docker build --tag azurefunctionsimage:v1.0.0 .
az acr login --name asdfasdf22123ds
docker tag azurefunctionsimage:v1.0.0 asdfasdf22123ds.azurecr.io/azurefunctionsimage:v1.0.0
docker push asdfasdf22123ds.azurecr.io/azurefunctionsimage:v1.0.0

az containerapp env create --name poc-environment --enable-workload-profiles --resource-group test-mi-permissions --location australiaeast

az storage account create --name arkstpoccontainerapps --location eastus --resource-group test-mi-permissions --sku Standard_LRS

az functionapp create --name azurefunctionsimage --storage-account arkstpoccontainerapps --environment poc-environment --workload-profile-name "Consumption" --resource-group test-mi-permissions --functions-version 4 --runtime dotnet-isolated --image asdfasdf22123ds.azurecr.io/azurefunctionsimage:v1.0.0 --assign-identity

$FUNCTION_APP_ID=$(az functionapp identity assign --name azurefunctionsimage --resource-group test-mi-permissions --query principalId --output tsv)
$ACR_ID=$(az acr show --name asdfasdf22123ds --query id --output tsv)
az role assignment create --assignee $FUNCTION_APP_ID --role AcrPull --scope $ACR_ID

# Test the function
az functionapp function show --resource-group  test-mi-permissions --name azurefunctionsimage --function-name HttpExample --query invokeUrlTemplate