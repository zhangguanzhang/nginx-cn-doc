# Installing NGINX Plus on the Google Cloud Platform

[NGINX Plus](https://www.nginx.com/products/nginx/), the high‑performance application delivery platform, load balancer, and web server, is available on the Google Cloud Platform as a virtual machine \(VM\) image. The VM image contains the latest version of NGINX Plus, optimized for use with the Google Cloud Platform Compute Engine.

### Installing the NGINX Plus VM

To quickly set up an NGINX Plus environment on the Google Cloud Platform, perform the following steps.

**Note:** The Google Cloud Platform interface is under active development. Although we make every effort to provide accurate instructions, they are subject to change. Also, the exact options you see depend on whether or not you have existing projects.

1. Access the [NGINX Plus page](https://console.cloud.google.com/marketplace/details/nginx-public/nginx-plus) in Google Marketplace and click the **LAUNCH ON COMPUTE ENGINE** button.

![](https://www.nginx.com/wp-content/uploads/2018/10/gcp-nginx-plus-page-oct2018.png)

1. The **Select or create a project** window opens. Enter a project name and select a value from the **Organization** drop‑down, then click the **Create** button.

   ![](https://www.nginx.com/wp-content/uploads/2018/10/gcp-create-select-project-oct2018.png)

2. In the **Configure & Deploy** window, enter or select appropriate values for zone, machine type, and so on. Click the **Deploy** button.

   **Note:** In the **Firewall** section, be sure the **Allow HTTP traffic** checkbox is checked. For more information on controlling incoming traffic, see the [Firewall Rules Overview](https://cloud.google.com/vpc/docs/firewalls) in the Google Cloud Platform documentation.

   ![](https://www.nginx.com/wp-content/uploads/2018/02/gcp-configure-deploy-feb2018.png)

3. Click the **Create** button. The Google Developers Console confirms that NGINX Plus was deployed.

   ![](https://www.nginx.com/wp-content/uploads/2018/02/gcp-deploy-confirmation-feb2018.png)

As soon as the project deploys and the new virtual machine \(VM\) instance starts running, NGINX Plus starts automatically and serves a default **index.html** page. To verify that NGINX Plus is working properly, use a web browser to access the public DNS name of the new VM and view the page.

You can also check the status of the NGINX Plus server by logging into the VM and running this command:

```text
# /etc/init.d/nginx status
```

### What If I Need Help?

If you encounter any problems with NGINX Plus configuration, documentation is available at [nginx.org](https://nginx.org/en/docs/) and in the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/admin-guide/).

Customers who purchase an NGINX Plus VM image on the Google Cloud Platform are eligible for the Google Cloud Platform VM support provided by the NGINX, Inc. engineering team. To activate support, submit the [Google Cloud Platform Support Activation](https://www.nginx.com/gcp-support-activation/) form.

#### Accessing the Open Source Licenses for NGINX Plus

NGINX Plus includes open source software written by NGINX, Inc. and other contributors. The text of the open source licenses is provided in Appendix B of the _NGINX Plus Reference Guide_. To access the guide included with the NGINX Plus VM instance, run this command:

```text
# less /usr/share/nginx/html/nginx-modules-reference.pdf
```

The _NGINX Plus Reference Guide_ is also [available online](https://cdn-1.wp.nginx.com/wp-content/files/nginx-pdfs/nginx-modules-reference.pdf) on the NGINX, Inc. website.  


