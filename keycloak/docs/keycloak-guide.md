# Keycloak Auth Provider Guide

This guide explains how to integrate a local Keycloak instance with the Red Hat Developer Hub (RHDH) local setup in the `rhdh-local` repository. By default, `rhdh-local` uses the guest authentication provider. To switch to the Keycloak provider for testing integration, adding users, and evaluating RBAC policies, follow the steps below.

## Prerequisites
- A local Keycloak instance is preconfigured in the `keycloak` folder of the repository. It includes:
  - A realm named `rhdh-local`.
  - A sample user: username `bob`, password `pass@1234`.
- Keycloak admin credentials: username `admin`, password `admin_pass`.
- Ensure you have access to edit the `/etc/hosts` file (requires sudo privileges on Unix-like systems).

## Steps to Integrate Keycloak Auth Provider

1. **Update Environment Variables**  
   Uncomment the following environment variable in your configuration file (e.g., `.env` or equivalent):  
   ```
   # Base url with keycloak
   BASE_URL=http://dev.rhdh-local.com:8080
   ```  
   This sets the base URL to work with the Keycloak-integrated setup.

2. **Configure the Auth Provider**  
   In your `app-config.local.yaml` , add the authentication section to use the Keycloak OIDC provider. Use the following default configuration:  
   ```yaml
   auth:
     environment: development
     session:
       # secret: <add a session secret here> 
     providers:
       oidc:
         development:
           metadataUrl: http://example.keycloak.com:8080/realms/rhdh-local
           clientId: rhdh-local-client
           clientSecret: lnwq4YeBcYJRxdmfjrjyBqaT0w1Dhqj0
           prompt: auto
           signIn: 
             resolvers:
               - resolver: preferredUsernameMatchingUserEntityName
   signInPage: oidc
   ```  
   Replace `<add a session secret here>` with a secure secret for session management.

3. **Configure the Catalog Provider (Optional)**  
   To provision users and groups from Keycloak into the RHDH software catalog, add the following to your `app-config.local.yaml`:  
   ```yaml
   catalog:
     providers:
       keycloakOrg:
         default:
           baseUrl: http://example.keycloak.com:8080
           loginRealm: rhdh-local
           clientId: rhdh-local-client
           clientSecret: lnwq4YeBcYJRxdmfjrjyBqaT0w1Dhqj0
           realm: rhdh-local
           schedule:
             frequency: { minutes: 1 }
   ```  
   This schedules periodic syncing of users from Keycloak.

4. **Enable the Keycloak Auth Provider Plugin**  
   To enable the Keycloak authentication provider plugin, add the following configuration to your `dynamic-plugins.override.yaml`:  
   ```yaml
   - package: ./dynamic-plugins/dist/backstage-community-plugin-catalog-backend-module-keycloak-dynamic
     disabled: false
   ```

5. **Update Hosts File for Proxy Redirection**  
   Add the following entries to your `/etc/hosts` file to ensure the NGINX proxy redirects traffic correctly:  
   ```
   127.0.0.1        example.keycloak.com dev.rhdh-local.com
   ```  

6. **Start the Services**  
   Run the local setup as below.
   ```bash
   [podman|docker] compose -f compose.yaml -f keycloak/compose.yaml up
   ```
   - This will start a keycloak,rhdh-local and nginx container.
   - Access Keycloak admin console at `http://example.keycloak.com:8080` using admin credentials (`admin` / `admin_pass`).  
   - Test login to RHDH at `http://dev.rhdh-local.com:8080` using the sample user (`bob` / `pass@1234`).  
   - You can add more users in Keycloak to test RBAC policies.

For official documentation on Keycloak authentication in RHDH:
- [Enabling authentication with Red Hat Build of Keycloak (RHBK)](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.6/html/authentication_in_red_hat_developer_hub/assembly-authenticating-with-rhbk#enabling-authentication-with-rhbk)
- [Provisioning users from RHBK to the software catalog](https://docs.redhat.com/en/documentation/red_hat_developer_hub/1.6/html/authentication_in_red_hat_developer_hub/assembly-authenticating-with-rhbk#provisioning-users-from-rhbk-to-the-software-catalog)

Additional Keycloak documentation:
- [Creating a realm](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/26.0/html/getting_started_guide/getting-started-zip-#getting-started-zip-create-a-realm)
- [Securing the application](https://docs.redhat.com/en/documentation/red_hat_build_of_keycloak/26.0/html/getting_started_guide/getting-started-zip-#getting-started-zip-secure-the-first-application)

