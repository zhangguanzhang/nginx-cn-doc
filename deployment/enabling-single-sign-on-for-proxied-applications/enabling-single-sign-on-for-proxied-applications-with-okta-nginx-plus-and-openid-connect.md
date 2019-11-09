# Enabling Single Sign-On for Proxied Applications with Okta, NGINX Plus, and OpenID Connect

This guide explains how to enable single sign-on \(SSO\) for applications being proxied by NGINX Plus. The solution uses OpenID Connect as the authentication mechanism, with Okta as the identity provider \(IdP\), and NGINX Plus as the relying party.

For more information about integrating OpenID Connect with NGINX Plus, see the [documentation](https://github.com/nginxinc/nginx-openid-connect#nginx-openid-connect) for NGINX’s reference implementation on GitHub.

* [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#prereqs)
* [Configuring Okta](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#okta)
* [Configuring NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#nginx-plus)
* [Testing](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#testing)
* [Troubleshooting](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#troubleshooting)

### Prerequisites

The instructions assume you have the following:

* An Okta account.
* An NGINX Plus subscription and NGINX Plus R15 or later. For installation instructions, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/).
* The [NGINX JavaScript module](https://www.nginx.com/blog/introduction-nginscript/) \(njs\), required for handling the interaction between NGINX Plus and the IdP. After installing NGINX Plus, install the module with the command for your operating system.

  For Debian and Ubuntu:

  ```text
  $ sudo apt install nginx-plus-module-njs 
  ```

  For CentOS, RHEL, and Oracle Linux:

  ```text
  $ sudo yum install nginx-plus-module-njs
  ```

* The following directive included in the top-level \(“main”\) configuration context in **/etc/nginx/nginx.conf**, to load the NGINX JavaScript module:

  ```text
  load_module modules/ngx_http_js_module.so;
  ```

### Configuring Okta

**Note:** The following procedure reflects the Okta GUI at the time of publication, but the GUI is subject to change. Use this guide as a reference and adapt to the current Okta GUI as necessary.

Create a new application for NGINX Plus in the Okta GUI:

1. Log in to your Okta account at [**okta.com**](https://www.okta.com/).
2. Click your username at the right side of the banner, then **Your Org** on the drop-down menu.

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Your-Org.png)

3. Click the  Admin  button at the right end of the banner.

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Admin-button.png)

4. On the **Dashboard** page that opens, click **Add Applications** in the Shortcuts column on the right side. \(You can tell you’re acting as Admin because the URL in your browser is now **https://**_username_**-admin.okta.com/admin/dashboard**.\)

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Dashboard.png)

5. On the **Add Application** page that opens, click the  Create New App  button in the left navigation column.

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Add-Application.png)

6. In the Create a New Application Integration window that pops up, select these values and then click the  Create  button:

   * **Platform** – Web
   * **Sign on method** – OpenID Connect

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Create-New-Application-Integration.png)

7. On the Create OpenID Connect Integration page that opens, fill in the following two fields and then click the  Save  button:

   * **Application name** – The name of the OpenID Connect relying party \(Okta refers to it as the “application”\). Here we’re using NGINX-Plus.
   * **Login redirect URIs** – The URI of the NGINX Plus instance, including the port number, and ending in /\_codexch\*\* \(in this guide it is https://my-nginx.example.com:443/\_codexch\).

     **Notes:**

     * For production, we strongly recommend that you use SSL/TLS \(port 443\).
     * The port number is mandatory even when you’re using the default port for HTTP \(80\) or HTTPS \(443\).

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Create-OpenID-Connect-Integration.png)

8. On the confirmation page that opens, make note of the values in the **Client ID** and **Client secret** fields in the **Client Credentials** section \(click the “eye” icon in the **Client secret** field to display the actual character string\). You will add them to the NGINX Plus configuration in Step 4 of the [next section](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#nginx-plus).

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Client-Credentials.png)

9. Navigate to the **Applications** page and click the **Assign Applications** button.

   ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Applications.png)

10. On the **Assign Applications** page, select the relying party \(the application from Step 7\) in the **Application** column. In the **People** column, select each user and group who you want to enable to use SSO.

    ![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-Assign-Applications.png)

### Configuring NGINX Plus

Configure NGINX Plus as the OpenID Connect relying party:

1. Create a clone of the [nginx-openid-connect](https://github.com/nginxinc/nginx-openid-connect) GitHub repository.

   ```text
   $ git clone https://github.com/nginxinc/nginx-openid-connect
   ```

2. Copy these files from the clone to **/etc/nginx/conf.d**:
   * **frontend.conf**
   * **openid\_connect.js**
   * **openid\_connect.server\_conf**
3. Get the URLs for the authorization endpoint, token endpoint, and JSON Web Key \(JWK\) file from the Okta configuration. Run the following `curl` command in a terminal, piping the output to the indicated `python` command to output the entire configuration in an easily readable format. We’ve abridged the output to show only the relevant fields.

   ```text
   $ curl https://<username>-admin.okta.com/.well-known/openid-configuration | python -m json.tool
   ...
   {
       "authorization_endpoint": "https://<username>.okta.com/oauth2/v1/authorize",
       ...
       "jwks_uri": "https://<username>.okta.com/oauth2/v1/keys",
       ...
       "token_endpoint": "https://<username>.okta.com/oauth2/v1/token",
    ...
    }
   ```

4. In your preferred text editor, open **/etc/nginx/conf.d/frontend.conf**. In the following [`set`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set) directives, change the variable values to the appropriate values for your environment:
   * `set $oidc_authz_endpoint` – Value of `authorization_endpoint` from Step 3 \(in this guide, `https://<username>.okta.com/oauth2/v1/authorize`\)
   * `set $oidc_token_endpoint` – Value of `token_endpoint` from Step 3 \(in this guide, `https://<username>.okta.com/oauth2/v1/token`\)
   * `set $oidc_client` – Value in the **Client ID** field from Step 8 of [Configuring Okta](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#okta) \(in this guide, `Ooaegfcqd9t1n5ubk356`\)
   * `set $oidc_client_secret` – Value in the **Client secret** field from Step 8 of [Configuring Okta](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#okta) \(in this guide, `wbBEIES5KUnwHixDyyAZcDT_ioVVNNDvCDWX9Ubu`\)
   * `set $oidc_token_type` – `id_token`
   * `set $oidc_hmac_key` – A unique, long, and secure phrase
5. Configure the JWK file. The procedure depends on which version of NGINX Plus you are using.
   * In NGINX Plus R17 and later, NGINX Plus can read the JWK file directly from the URL reported as `jwks_uri` in Step 3 of this section. Change **/etc/nginx/conf.d/frontend.conf** as follows:
     1. Comment out \(or remove\) the [`auth_jwt_key_file`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_file) directive.
     2. Uncomment the [`auth_jwt_key_request`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_request) directive and replace `_jwks_uri` with the value from the `jwks_uri` field reported in Step 3 \(in this guide, `https://<username>.okta.com/oauth2/v1/keys`\).
   * In NGINX Plus R16 and earlier, the JWK file must be on the local disk. \(You can also use this method with NGINX Plus R17 and later if you wish.\)
     1. Copy the JSON contents from the JWK file named in the `jwks_uri` field in Step 3 \(in this guide, `https://username.okta.com/oauth2/v1/keys`\) to a local file \(for example, `/etc/nginx/my_okta_jwk.json`\).
     2. In **/etc/nginx/conf.d/frontend.conf**, change the variable value in the `set $oidc_jwt_keyfile` directive to the local file path.
6. Confirm that the user named by the [`user`](https://nginx.org/en/docs/ngx_core_module.html#user) directive in **/etc/nginx/conf** has read permission on the JWK file.

### Testing

In a browser, enter the address of your NGINX Plus instance and try to log in using the credentials of a user assigned to the application \(see Step 10 in [Configuring Okta](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/okta/#okta)\).

![](https://www.nginx.com/wp-content/uploads/2019/04/Okta-login-window.png)

### Troubleshooting

See the [**Troubleshooting**](https://github.com/nginxinc/nginx-openid-connect#troubleshooting) section at the nginx-openid-connect repository on GitHub.

