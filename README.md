1. Create and deploy apps
   -  cd frontend
   -  az webapp up --resource-group rg-raptor-test --name raptor-client --plan myPlan --sku FREE --location "East US2" --os-type Linux --runtime "NODE:18-lts" --debug
   -  cd ../backend
   -  az webapp up --resource-group rg-raptor-test --name raptor-server --plan myPlan --sku FREE --location "East US2" --runtime "NODE:18-lts"
2. Configure app setting
   - az webapp config appsettings set --resource-group rg-raptor-test --name raptor-client --settings BACKEND_URL="https://raptor-server.azurewebsites.net" 
3. Frontend calls the backend
   - Open the frontend web app in a browser, https://raptor-client.azurewebsites.net
4. Configure authentication
   -  raptor-server: Add the MS as ID provider and copy the client ID from raptor-server -> Authentication
   -  raptor-client: Add the MS as ID provider and copy the client ID from raptor-server -> Authentication
6. Grant frontend app access to backend. Technically, you give the frontend's AD application the permissions to access the backend's AD application on the user's behalf
   -  raptor-client => Authentication -> Identity Provider section -> Select the app registration by name raptor-client -> Manage -> API permissions -> Add a permission -> My APIs -> select "raptor-server" -> select "Delegated Permissions" -> Under permissions section select "user_impersonation" and then Add Permissions 
7. Configure App Service to return a usable access token i.e, configure App Service authentication and authorization to give you a usable access token for accessing the back end.
   - In the Cloud Shell, run the following commands on the front-end app to add the scope parameter to the authentication setting. Replace <back-end-client-id> with raptor-server client id.
   - ```
     az extension add --name authV2
     authSettings=$(az webapp auth show -g rg-raptor-test -n raptor-client)
     authSettings=$(echo "$authSettings" | jq '.properties' | jq '.identityProviders.azureActiveDirectory.login += {"loginParameters":["scope=openid offline_access api://<back-end-client-id>/user_impersonation"]}')
     az webapp auth set --resource-group rg-raptor-test --name raptor-client --body "$authSettings"
     ```
   - After executing the above commands, webapp configurion would be something like this.
   - ```      
      arif [ ~ ]$ az webapp auth show -g rg-reptor-test -n raptor-client
      {
        "aadClaimsAuthorization": "{\"allowed_groups\":null,\"allowed_client_applications\":[\"2a69e0a2-5712-4423-92a7-1456b8a7e8fd\"]}",
        "additionalLoginParams": [
          "scope=openid offline_access api://258529a8-d959-4d0f-a222-98d7639236ce/user_impersonation"
        ],
        "allowedAudiences": [
          "api://2a69e0a2-5712-4423-92a7-1456b8a7e8fd"
        ],
        ...
        ... 
     ```
8. Configure backend App Service to accept a token only from the front-end App Service
   - Not doing this configuration results in a 403: Forbidden error when you pass the token from the front end to the back end
   - Get the appId of the front-end App Service. You can get this value on the Authentication page of the front-end App Service
   - Run the following Azure CLI, substituting the <front-end-app-id>
   - ```
     authSettings=$(az webapp auth show -g rg-raptor-test -n raptor-server)
     authSettings=$(echo "$authSettings" | jq '.properties' | jq '.identityProviders.azureActiveDirectory.validation.defaultAuthorizationPolicy.allowedApplications += ["<front-end-app-id>"]')
     az webapp auth set --resource-group rg-raptor-test --name raptor-server --body "$authSettings"     
     ```
10. 


---
page_type: sample
name: Authenticate and authorize users end-to-end in Azure App Service with JavaScript
description: This tutorial shows how to secure two App Services (frontend and backend), passing user auth from the frontend app to the backend. 
languages:
- javascript
products:
- azure-app-service
---

# Authenticate and authorize users end-to-end in Azure App Service

Azure App Service provides a highly scalable, self-patching web hosting service. In addition, App Service has built-in support for [user authentication and authorization](https://learn.microsoft.com/azure/app-service/overview-authentication-authorization). This tutorial shows how to secure your apps with App Service authentication and authorization. It uses an Express.js with views frontend as an example. App Service authentication and authorization support all language runtimes, and you can learn how to apply it to your preferred language by following the tutorial.

## Features

In the tutorial, you learn:

> [!div class="checklist"]
> * Enable built-in authentication and authorization
> * Secure apps against unauthenticated requests
> * Use Azure Active Directory as the identity provider
> * Access a remote app on behalf of the signed-in user
> * Secure service-to-service calls with token authentication
> * Use access tokens from server code
> * Use access tokens from client (browser) code

## Read the tutorial

[Read the tutorial](https://learn.microsoft.com/azure/app-service/tutorial-auth-aad) to understand how to deploy this scenario to App Service.
