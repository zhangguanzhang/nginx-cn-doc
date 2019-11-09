# Enabling Single Sign-On for Proxied Applications with OneLogin, NGINX Plus, and OpenID Connect

This guide explains how to enable single sign-on \(SSO\) for applications being proxied by NGINX Plus. The solution uses OpenID Connect as the authentication mechanism, with [OneLogin](https://www.onelogin.com/) as the identity provider \(IdP\), and NGINX Plus as the relying party.

For more information about integrating OpenID Connect with NGINX Plus, see the [documentation](https://github.com/nginxinc/nginx-openid-connect#nginx-openid-connect) for NGINX’s reference implementation on GitHub.

* [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#prereqs)
* [Configuring OneLogin](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#onelogin)
* [Configuring NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#nginx-plus)
* [Testing](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#testing)
* [Troubleshooting](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#troubleshooting)

### Prerequisites

The instructions assume you have the following:

* A OneLogin subscription.
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

### Configuring OneLogin

**Note:** The following procedure reflects the OneLogin GUI at the time of publication, but the GUI is subject to change. Use this guide as a reference and adapt to the current OneLogin GUI as necessary.

Create a new application for NGINX Plus in the OneLogin GUI:

1. Log in to your OneLogin account at **https://**_domain_**.onelogin.com**, where _domain_ is the domain you chose when you created your account.
2. Click  Applications  in the title bar and then click the  Add App  button in the upper right corner of the window that opens.

   ![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-SSO_Add-App-button.png)

3. On the **Find Applications** page that opens, type OpenID Connect in the search box. Click on the **OpenID Connect \(OIDC\)** row that appears.

   ![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-SSO_Find-Applications-page.png)

4. On the **Add OpenId Connect \(OIDC\)** page that opens, change the value in the **Display Name** field to NGINX Plus and click the  Save  button.

   ![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-SSO_Add-OIDC-page.png)

5. When the save completes, a new set of choices appears in the left navigation bar. Click **Configuration**. In the **Redirect URI’s** field, type the URI of the NGINX Plus instance including the port number, and ending in /\_codexch \(in this guide it is https://my-nginx.example.com:443/\_codexch\). Then click the  Save  button.

   **Notes:**

   * For production, we strongly recommend that you use SSL/TLS \(port 443\).
   * The port number is mandatory even when you’re using the default port for HTTP \(80\) or HTTPS \(443\).

   ![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-SSO_Configuration-tab.png)

6. When the save completes, click **SSO** in the left navigation bar. Click Show client secret below the **Client Secret** field. Record the values in the **Client ID** and **Client Secret** fields. You will add them to the NGINX Plus configuration in Step 4 of the [next section](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#nginx-plus).

   ![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-SSO_SSO-tab.png)

7. Assign users to the application \(in this guide, NGINX Plus\) to enable them to access it for SSO. OneLogin recommends using [roles](https://onelogin.service-now.com/kb_view_customer.do?sysparm_article=KB0010606) for this purpose. You can access the **Roles** page under  Users  in the title bar.

   ![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-SSO_Roles-page.png)

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
3. Get the URLs for the authorization endpoint, token endpoint, and JSON Web Key \(JWK\) file from the OneLogin configuration. Run the following `curl` command in a terminal, piping the output to the indicated `python` command to output the entire configuration in an easily readable format. We’ve abridged the output to show only the relevant fields.

   ```text
   $ curl https://<OneLogin-server-address>.onelogin.com/oidc/.well-known/openid-configuration | python -m json.tool
   ...
   {
       "authorization_endpoint": "https://<domain>.onelogin.com/oidc/auth",
       ...
       "jwks_uri": "https://<domain>.onelogin.com/oidc/certs",
       ...
       "token_endpoint": "https://<domain>.onelogin.com/oidc/token",
    ...
    }
   ```

4. In your preferred text editor, open **/etc/nginx/conf.d/frontend.conf**. In the following [`set`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set) directives, change the variable values to the appropriate values for your environment:
   * `set $oidc_authz_endpoint` – Value of `authorization_endpoint` from Step 3 \(in this guide, `https://<domain>.onelogin.com/oidc/auth`\)
   * `set $oidc_token_endpoint` – Value of `token_endpoint` from Step 3 \(in this guide, `https://<domain>.onelogin.com/oidc/token`\)
   * `set $oidc_client` – Value in the **Client ID** field from Step 6 of [Configuring OneLogin](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#onelogin) \(in this guide, `168d5600-9224-0137-3b2b-0acf<xxx>`\)
   * `set $oidc_client_secret` – Value in the **Client secret** field from Step 6 of [Configuring OneLogin](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#onelogin) \(in this guide, `c9210a67d09e85<xxx>`\)
   * `set $oidc_token_type` – `id_token`
   * `set $oidc_hmac_key` – A unique, long, and secure phrase
5. Configure the JWK file. The method depends on which version of NGINX Plus you are using.
   * In NGINX Plus R17 and later, NGINX Plus can read the JWK file directly from the URL reported as `jwks_uri` in Step 3 of this section. Change **/etc/nginx/conf.d/frontend.conf** as follows:
     1. Comment out \(or remove\) the [`auth_jwt_key_file`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_file) directive.
     2. Uncomment the [`auth_jwt_key_request`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_request) directive and replace `_jwks_uri` with the value from the `jwks_uri` field reported in Step 3 \(in this guide, `https://<domain>.onelogin.com/oidc/certs`\).
   * In NGINX Plus R16 and earlier, the JWK file must be on the local disk. \(You can also use this method with NGINX Plus R17 and later if you wish.\)
     1. Copy the JSON contents from the JWK file named in the `jwks_uri` field in Step 3 \(in this guide, `https://<domain>.onelogin.com/oidc/certs`\) to a local file \(for example, `/etc/nginx/my_onelogin_jwk.json`\).
     2. In **/etc/nginx/conf.d/frontend.conf**, change the variable value in the `set $oidc_jwt_keyfile` directive to the local file path.
6. Confirm that the user named by the [`user`](https://nginx.org/en/docs/ngx_core_module.html#user) directive in **/etc/nginx/conf** has read permission on the JWK file.

### Testing

In a browser, enter the address of your NGINX Plus instance and try to log in using the credentials of a user assigned to the application \(see Step 7 in [Configuring OneLogin](https://docs.nginx.com/nginx/deployment-guides/single-sign-on/onelogin/#onelogin)\).

![](https://www.nginx.com/wp-content/uploads/2019/07/OneLogin-login-window.png)

### Troubleshooting

See the [**Troubleshooting**](https://github.com/nginxinc/nginx-openid-connect#troubleshooting) section at the nginx-openid-connect repository on GitHub.

