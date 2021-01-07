# How to Setup Apple Sign In with Keycloak
For now keycloak does not provide a ready to use identity provider for Apple like Google or Facebook.
Since the Apple Sign in is mandatory to publish apps in App store (only applicable if your app has other social logins),
this document contains how to integrate Apple Sign in with keycloak.

##Pre-requisites
* Keycloak - 11.0.2 (Tested in this version, should work in the later versions. Older versions might not support)
* Apple Developer Account (Should have the capability to publish apps)

## How to configure
### Apple Developer Account
Refer [this blog](https://developer.okta.com/blog/2019/06/04/what-the-heck-is-sign-in-with-apple) on how to configure the Apple Developer Account if below steps are not clear.

1. Sign in to the Apple Developer account and go to https://developer.apple.com/account/resources/identifiers/list/bundleId
2. If the app does not have an App Id, first create one and enable `Sign in with Apple` for the app.
3. Then create a Service Id and configure the service id as below.
    * Select the App Id which is created in Step 2 as the `Primary App Id`
    * Add a Description. This will be displayed to users when they login.
    * Add an identifier. This can be anything and will later be used in Keycloak as the `client_id`.
    * Tick the `Sign in with Apple` button and click `configure` button next to it.
    * Provide the domain which the Keycloak is hosted and provide the redirect url. The Redirect url is like this `https://{KEYCLOAK_DOMAIN}/auth/realms/{REALM_NAME}/broker/apple/endpoint`.
    
4. Then in the main "Certificates, Identifiers & Profiles" screen, choose Keys from the side tab. This key will be used for client authentication.
    * Add a new key by clicking the plus icon.
    * Provide a `Key Name`.
    * Tick the `Sign in with Apple` button and click `configure` button next to it.
    * select your primary App ID you created earlier in Step 2.
    ```
   Apple will generate a new private key for you and let you download it only once. 
   Make sure you save this file, because you won’t be able to get it back again later! 
   The file you download will end in .p8, but it’s just text inside. 
   So get a copy of the original file and rename it to key.txt so that it’s easier to use in the next steps.
    ```
   * Lastly, go back and view the key information to find your Key ID which you’ll need in the next step.
    
### Keycloak
* To configure the Apple Sign In with keycloak, we need to use an extension.
* Get the extension file (a .jar file) from [here](https://github.com/BenjaminFavre/keycloak-apple-social-identity-provider).
* Install the provider JAR file following Keycloak instructions [here](https://www.keycloak.org/docs/latest/server_development/index.html#using-the-keycloak-deployer)
* For this project, we use a standalone keycloak. So go to `/hms/installs/keycloak/standalone/deployments` and copy the jar file to that location. This way Keycloak will automatically deploy the extension. This method works for `docker` as well.
* In the Keycloak Admin Console, go to Identity Providers and you will see Apple is listed as an Identity Provider.

5. Select the Apple Identity Provider.
6. Configure the Identity Provider as below.
    * Client ID - Service Id from the Apple Developer Account
    * Key ID - Key Id from the Apple Developer Account
    * Team ID - Team Id from the Apple Developer Account
    * Default Scopes - openid%20name%20email
        * This is to get the first_name and last_name of the user and email from the Apple Account
        * If the above parameters are not required, keep `Default Scope` as blank.

7. `Client Secret` needs to added after modifying the content of the file `key.txt` generated in Step 4 as below.
    * The content of the file is like this.
    ```
    -----BEGIN PRIVATE KEY-----
    MIGTAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBHkwdwIBAQQggATCYtPlHWIL2Q7f
    /jVPevNjKTw8gRNOJf7uy6mNrwigCgYIKoZIzj0DAQehRANCAATIkf+dpToIIrSp
    alTz0L5+tWPE9KSzEYG1Gq8pnQkFhu9OSJ6EVSd4OOIbWuln4cbVxaJ3TpCZ05tm
    suMzu5RU
    -----END PRIVATE KEY-----
    ```
   * Copy the base64 encode value in the middle and remove new lines and delimiters if there are any.
   * For example after modifying the above it is as this.
   ```
   MIGTAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBHkwdwIBAQQggATCYtPlHWIL2Q7f/jVPevNjKTw8gRNOJf7uy6mNrwigCgYIKoZIzj0DAQehRANCAATIkf+dpToIIrSpalTz0L5+tWPE9KSzEYG1Gq8pnQkFhu9OSJ6EVSd4OOIbWuln4cbVxaJ3TpCZ05tmsuMzu5RU
   ```
   * Copy the modified base64 content and paste it as the `Client ID` in Keycloak
    
8. Save the configurations and test.
9. When publishing the apps to App Store, the Apple Signin Button in Keycloak Theme should be styled 
   to meet the Apple's styles. So check this before publishing.
