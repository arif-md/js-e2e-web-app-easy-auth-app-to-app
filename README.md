1. Create and deploy apps
   -  cd frontend
   -  az webapp up --resource-group rg-raptor-test --name raptor-client --plan myPlan --sku FREE --location "East US2" --os-type Linux --runtime "NODE:18-lts" --debug
   -  cd ../backend
   -  az webapp up --resource-group rg-raptor-test --name raptor-server --plan myPlan --sku FREE --location "East US2" --runtime "NODE:18-lts"
2. Configure app setting
   - az webapp config appsettings set --resource-group rg-raptor-test --name raptor-client --settings BACKEND_URL="https://raptor-server.azurewebsites.net" 
3. Frontend calls the backend
   - Open the frontend web app in a browser, https://raptor-client.azurewebsites.net
4. In a nutshell this is how we are going to make configuration changes in the following steps.
   - Add user_impersonation permission on raptor-client registration app
   - Client : Configure App Service to return a usable access token for raptor-client.
     ```
            "azureActiveDirectory": {
              "enabled": true,
              "login": {
                "disableWWWAuthenticate": false,
                "loginParameters": [
                  "scope=openid offline_access api://<back-end-client-id>/user_impersonation"
                ]
              },
     ```
   - Server : Configure backend App Service to accept a token only from the front-end App Service
     ```
            "azureActiveDirectory": {
              "enabled": true,
              "login": {
                "disableWWWAuthenticate": false
              },
              "registration": {
                "clientId": "8e2bc82f-76db-43ba-b07e-2dc5f4a5769f",
                "clientSecretSettingName": "MICROSOFT_PROVIDER_AUTHENTICATION_SECRET",
                "openIdIssuer": "https://sts.windows.net/60a52712-6791-451d-bdec-5021c5f60a64/v2.0"
              },
              "validation": {
                "allowedAudiences": [],
                "defaultAuthorizationPolicy": {
                  "allowedApplications": [
                    <front-end-client-id>
                  ],
                  "allowedPrincipals": {}
                },
                "jwtClaimChecks": {}
              }
            },     
     ```
   - Client to return the proper access token:
5. This is how the webapp authentication configuration looks like.
   - Auth config for raptor-client
   - ```
      arif [ ~ ]$ az webapp auth show -g rg-raptor-test -n raptor-client
      {
        "aadClaimsAuthorization": null,
        "additionalLoginParams": null,
        "allowedAudiences": null,
        "allowedExternalRedirectUrls": null,
        "authFilePath": null,
        "clientId": null,
        "clientSecret": null,
        "clientSecretCertificateThumbprint": null,
        "clientSecretSettingName": null,
        "configVersion": "v1",
        "defaultProvider": null,
        "enabled": false,
        "facebookAppId": null,
        "facebookAppSecret": null,
        "facebookAppSecretSettingName": null,
        "facebookOAuthScopes": null,
        "gitHubClientId": null,
        "gitHubClientSecret": null,
        "gitHubClientSecretSettingName": null,
        "gitHubOAuthScopes": null,
        "googleClientId": null,
        "googleClientSecret": null,
        "googleClientSecretSettingName": null,
        "googleOAuthScopes": null,
        "id": "/subscriptions/5b489d19-6e0a-45bd-be65-d7d1c40af428/resourceGroups/rg-reptor-test/providers/Microsoft.Web/sites/raptor-client/config/authsettings",
        "isAuthFromFile": null,
        "issuer": null,
        "kind": null,
        "location": "East US 2",
        "microsoftAccountClientId": null,
        "microsoftAccountClientSecret": null,
        "microsoftAccountClientSecretSettingName": null,
        "microsoftAccountOAuthScopes": null,
        "name": "authsettings",
        "resourceGroup": "rg-reptor-test",
        "runtimeVersion": null,
        "tags": {
          "createdOnDate": "2025-07-10T22:59:38.4736321Z"
        },
        "tokenRefreshExtensionHours": null,
        "tokenStoreEnabled": null,
        "twitterConsumerKey": null,
        "twitterConsumerSecret": null,
        "twitterConsumerSecretSettingName": null,
        "type": "Microsoft.Web/sites/config",
        "unauthenticatedClientAction": null,
        "validateIssuer": null
      }     
     ```
   - Auth config for raptor-server
   - ```
      arif [ ~ ]$ az webapp auth show -g rg-raptor-test -n raptor-server
      {
        "aadClaimsAuthorization": null,
        "additionalLoginParams": null,
        "allowedAudiences": null,
        "allowedExternalRedirectUrls": null,
        "authFilePath": null,
        "clientId": null,
        "clientSecret": null,
        "clientSecretCertificateThumbprint": null,
        "clientSecretSettingName": null,
        "configVersion": "v1",
        "defaultProvider": null,
        "enabled": false,
        "facebookAppId": null,
        "facebookAppSecret": null,
        "facebookAppSecretSettingName": null,
        "facebookOAuthScopes": null,
        "gitHubClientId": null,
        "gitHubClientSecret": null,
        "gitHubClientSecretSettingName": null,
        "gitHubOAuthScopes": null,
        "googleClientId": null,
        "googleClientSecret": null,
        "googleClientSecretSettingName": null,
        "googleOAuthScopes": null,
        "id": "/subscriptions/5b489d19-6e0a-45bd-be65-d7d1c40af428/resourceGroups/rg-reptor-test/providers/Microsoft.Web/sites/raptor-server/config/authsettings",
        "isAuthFromFile": null,
        "issuer": null,
        "kind": null,
        "location": "East US 2",
        "microsoftAccountClientId": null,
        "microsoftAccountClientSecret": null,
        "microsoftAccountClientSecretSettingName": null,
        "microsoftAccountOAuthScopes": null,
        "name": "authsettings",
        "resourceGroup": "rg-reptor-test",
        "runtimeVersion": null,
        "tags": {
          "createdOnDate": "2025-07-11T02:16:23.0948190Z"
        },
        "tokenRefreshExtensionHours": null,
        "tokenStoreEnabled": null,
        "twitterConsumerKey": null,
        "twitterConsumerSecret": null,
        "twitterConsumerSecretSettingName": null,
        "type": "Microsoft.Web/sites/config",
        "unauthenticatedClientAction": null,
        "validateIssuer": null
      }     
     ```     
6. Configure authentication
   -  raptor-server: Add the MS as ID provider and copy the client ID from raptor-server -> Authentication
   -  raptor-client: Add the MS as ID provider and copy the client ID from raptor-client -> Authentication
   -  result of above configuration is:
   -  ```
      arif [ ~ ]$ az webapp auth show -g rg-raptor-test -n raptor-server
      {
        "aadClaimsAuthorization": "{\"allowed_groups\":null,\"allowed_client_applications\":[\"8e2bc82f-76db-43ba-b07e-2dc5f4a5769f\"]}",
        "additionalLoginParams": null,
        "allowedAudiences": [
          "api://8e2bc82f-76db-43ba-b07e-2dc5f4a5769f"
        ],
        "clientId": "8e2bc82f-76db-43ba-b07e-2dc5f4a5769f",
        "clientSecretSettingName": "MICROSOFT_PROVIDER_AUTHENTICATION_SECRET",
        "configVersion": "v2",
        "enabled": true,
        "issuer": "https://sts.windows.net/60a52712-6791-451d-bdec-5021c5f60a64/v2.0",
        "tokenStoreEnabled": true,      
        "unauthenticatedClientAction": "RedirectToLoginPage",
        ...
        ...
      }
      arif [ ~ ]$ az webapp auth show -g rg-raptor-test -n raptor-client
      {
        "aadClaimsAuthorization": "{\"allowed_groups\":null,\"allowed_client_applications\":[\"52341723-9883-494f-91fe-755b93e52420\"]}",
        "additionalLoginParams": null,
        "allowedAudiences": [
          "api://52341723-9883-494f-91fe-755b93e52420"
        ],
        "clientId": "52341723-9883-494f-91fe-755b93e52420",
        "clientSecretSettingName": "MICROSOFT_PROVIDER_AUTHENTICATION_SECRET",
        "configVersion": "v2",
        "issuer": "https://sts.windows.net/60a52712-6791-451d-bdec-5021c5f60a64/v2.0",
        "tokenStoreEnabled": true,
        "unauthenticatedClientAction": "RedirectToLoginPage",
        ...
        ...
      }      
      ```
     
7. Grant frontend app access to backend. Technically, you give the frontend's AD application the permissions to access the backend's AD application on the user's behalf. 
   -  raptor-client => Authentication -> Identity Provider section -> Select the app registration by name raptor-client -> Manage -> API permissions -> Add a permission -> My APIs -> select "raptor-server" -> select "Delegated Permissions" -> Under permissions section select "user_impersonation" and then Add Permissions.
   -  **Note** that after performing the above steps, there wont be any change in web app configurations, only change is on app registration side. 
8. Configure App Service to return a usable access token i.e, configure App Service authentication and authorization to give you a usable access token for accessing the back end.
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
      The behavior of this command has been altered by the following extension: authV2
      {
        "id": "/subscriptions/5b489d19-6e0a-45bd-be65-d7d1c40af428/resourceGroups/rg-reptor-test/providers/Microsoft.Web/sites/raptor-client/config/authsettingsV2",
        "location": "East US 2",
        "name": "authsettingsV2",
        "properties": {
          "clearInboundClaimsMapping": "false",
          "globalValidation": {
            "redirectToProvider": "azureactivedirectory",
            "requireAuthentication": true,
            "unauthenticatedClientAction": "RedirectToLoginPage"
          },
          "httpSettings": {
            "forwardProxy": {
              "convention": "NoProxy"
            },
            "requireHttps": true,
            "routes": {
              "apiPrefix": "/.auth"
            }
          },
          "identityProviders": {
            "apple": {
              "enabled": true,
              "login": {},
              "registration": {}
            },
            "azureActiveDirectory": {
              "enabled": true,
              "login": {
                "disableWWWAuthenticate": false,
                "loginParameters": [
                  "scope=openid offline_access api://8e2bc82f-76db-43ba-b07e-2dc5f4a5769f/user_impersonation"
                ]
              },
         ...
         ...
         ...
     ```
9. Configure backend App Service to accept a token only from the front-end App Service
   - Not doing this configuration results in a 403: Forbidden error when you pass the token from the front end to the back end
   - Get the appId of the front-end App Service. You can get this value on the Authentication page of the front-end App Service
   - Run the following Azure CLI, substituting the <front-end-app-id>
   - ```
     authSettings=$(az webapp auth show -g rg-raptor-test -n raptor-server)
     authSettings=$(echo "$authSettings" | jq '.properties' | jq '.identityProviders.azureActiveDirectory.validation.defaultAuthorizationPolicy.allowedApplications += ["<front-end-app-id>"]')
     az webapp auth set --resource-group rg-raptor-test --name raptor-server --body "$authSettings"     
     ```
   - After executing the above commands, webapp configurion would be something like this.
   - ```
      arif [ ~ ]$ az webapp auth show -g rg-reptor-test -n raptor-server
      The behavior of this command has been altered by the following extension: authV2
      {
        "id": "/subscriptions/5b489d19-6e0a-45bd-be65-d7d1c40af428/resourceGroups/rg-reptor-test/providers/Microsoft.Web/sites/raptor-server/config/authsettingsV2",
        "location": "East US 2",
        "name": "authsettingsV2",
        "properties": {
          "clearInboundClaimsMapping": "false",
          "globalValidation": {
            "redirectToProvider": "azureactivedirectory",
            "requireAuthentication": true,
            "unauthenticatedClientAction": "RedirectToLoginPage"
          },
          "httpSettings": {
            "forwardProxy": {
              "convention": "NoProxy"
            },
            "requireHttps": true,
            "routes": {
              "apiPrefix": "/.auth"
            }
          },
          "identityProviders": {
            "apple": {
              "enabled": true,
              "login": {},
              "registration": {}
            },
            "azureActiveDirectory": {
              "enabled": true,
              "login": {
                "disableWWWAuthenticate": false
              },
              "registration": {
                "clientId": "8e2bc82f-76db-43ba-b07e-2dc5f4a5769f",
                "clientSecretSettingName": "MICROSOFT_PROVIDER_AUTHENTICATION_SECRET",
                "openIdIssuer": "https://sts.windows.net/60a52712-6791-451d-bdec-5021c5f60a64/v2.0"
              },
              "validation": {
                "allowedAudiences": [],
                "defaultAuthorizationPolicy": {
                  "allowedApplications": [
                    "8e2bc82f-76db-43ba-b07e-2dc5f4a5769f"
                  ],
                  "allowedPrincipals": {}
                },
                "jwtClaimChecks": {}
              }
            },
            ...
            ...
            ...
     ```
10. cleanup the following resources. Service plan, frontend/backend apps and app registrations. Following command deletes app registration.
     - az ad app delete --id < client-id >
     - az ad app delete --id < server-id >


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
