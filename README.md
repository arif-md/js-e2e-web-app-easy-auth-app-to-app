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
7. Configure App Service to return a usable access token
   - 
9. ssd
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
