# Fast Api based IDP with react Front-End

This is Fast API Based IDP

## How it works

 - Front end requests authorization Url from back-end proxy which manages communication between API resources and front-end
 - Proxy back end calls prepared keycloak instance and brings authorization code to front-end (Directly from Keycloak instance).
 - Front end sends authorization `code` and `session_state` (passed from Keycloak instance) to proxy back-end
 - Proxy back-end exchanges these parameters to access_token which will be used for protected API calls with API resources placed on "main back-end"
 - Once access_token is stored in proxy back-end, this one will set HTTP-Only cookies to front-end.
 - Final, Cookies will allow authorized operations which will be routed through proxy back-end to "main" back-end and response will be forwarded to front end too.
 - On log out proxy back-end will destroy session on Keycloak instance, remove cookies from front end, black list access_token.
 
 
 ## Keycloak configuration
 
 
 ### Local setup
 
 Pull docker image from here

[https://hub.docker.com/r/jboss/keycloak/](https://hub.docker.com/r/jboss/keycloak/)
 
 run docker image
 
 `docker run -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin jboss/keycloak`
 
 Access instance on [http://localhost:8080](http://localhost:8080) using admin as username and admin as password
 
### Create a new realm. 
 
Apply following settings (Save name somewhere)

   - General: Enabled, OpenID Endpoint Configuration
   - Login: User registration enabled
    
### Create a new client. 
 
(Save this name also somewhere)
 
 - Access Type: Select confidential
 - Settings: Enabled, Direct Access Granted, Service Accounts Enabled, Authorization Enabled
 - Scope: Full Scope Allowed (Will automatically grant all available roles to all users using this client, you may want to disable this and assign the roles to the client manually)
 - Valid Redirect URIs: [http://localhost:3000/callback](http://localhost:3000/callback) and [http://localhost:3000](http://localhost:3000) __This one needed to return on page after log out__ 
 - In credentials tab please copy secret
 - Go to Secrets and generate new certificate turning on **Use JWKS** (Save contents this is very important thing to decode token on back-end)
    
 Modify the admin-cli client
    - Access Type: Select confidential
    - Settings: Service Accounts Enabled
    - Scope: Full Scope Allowed
    - Service Account Roles: Select all Client Roles available for account and realm_management
    - In credentials tab please copy secret

 
 So finally you need following values:

        server_url="http://localhost:8080/auth", # This is your keycloak server url
        client_id="fapy", # You client id 
        client_secret="CPbW5J7Ge9pwgFatK4TuV71GVwiLgamv", # your client secret
        admin_client_secret="l1iBN1Ddv21kfwcCsYM5D2GaxvZmRA2O", # your admin secret
        realm="fapy", # realm id
        callback_uri="http://localhost:3000/callback" # front end url
    
 You can set then es env variables or hardcode (Do not use in production)
 
## Proxy auth back-end configuration

Just clone repository: 

`https://github.com/labadze/fastapi_auth_proxy_middleware`

and place values above follow README.md there to run database migrations and start application successfully.
Note you need postgres database server running (You can use docker or local server installation).

## Main back-end configuration

Just clone repository:

`https://github.com/labadze/fastapi_back-end_up`

and place values above follow `README.md` there to run database migrations and start application successfully.
In this case you can place same credentials but preferable use another in production
Note you need postgres database server running (You can use docker or local server installation).

## Run front end

You need just node.js and npm installed so clone repository: 

`https://github.com/labadze/proxy-auth-react-app`

and run npm install, then `npm start`.

You're ready to go do not forget to create user in keycloak to pass authorization process.

