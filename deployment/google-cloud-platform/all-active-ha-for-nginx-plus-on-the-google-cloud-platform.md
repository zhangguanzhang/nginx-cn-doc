# All-Active HA for NGINX Plus on the Google Cloud Platform



This deployment guide explains how to create and configure the initial components for an all‑active, high‑availability deployment of NGINX Plus on Google Compute Engine \(GCE\), the Google Cloud Platform \(GCP\) product for running workloads on virtual machines \(VMs\). Multiple instances of NGINX Plus in active pairs load balance incoming connections across multiple application environments.

* [Design and Topology](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#design)
* [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#prereqs)
* [Task 1: Creating a Project and Firewall Rules](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#project-firewall)
* [Task 2: Creating Source Instances](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source)
  * [Creating Source Instances from VM Images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm)
  * [Creating Source Instances from Prebuilt NGINX Plus Images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt)
* [Task 3: Creating “Gold” Images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#gold)
* [Task 4: Creating Instance Templates](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#templates)
* [Task 5: Creating Image Health Checks](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#health-checks)
* [Task 6: Creating Instance Groups](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#groups)
* [Task 7: Configuring GCE Network Load Balancer](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#static-ip)
* [Task 8: Testing the All-Active Load Balancing Deployment](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#testing)

**Notes:**

* GCE is a highly dynamic environment where the names and arrangement of GUI elements \(such as menu items, checkboxes, and configuration navigation\) are subject to change. We have made every effort to accurately represent the GCE GUI at the time of original publication, but options and locations might change over time. Use this guide as a reference and adapt to the current GCE working environment as necessary.
* The configuration described in this guide allows anyone from any public IP address to access the NGINX Plus instances directly without restriction. While this most flexibly accommodates common scenarios in a test environment, we do not recommend it for production use. Before deploying the architecture in production, we strongly recommend that you disable HTTP and HTTPS access to the app-1 and app-2 instances over the GCE external IP addresses, or remove the external IP addresses for all application instances so that they’re accessible only on the internal GCE network.

### Design and Topology

The deployment combines the following technologies:

* [NGINX Plus](https://www.nginx.com/products/nginx) – Load balances HTTP connections across multiple instances of two different applications. Instructions are provided both for manual installation on a standard GCE VM image and for setting up the prebuilt NGINX Plus VM image available in the Google Marketplace.
* PHP-FPM – Supports the two sample applications.
* [GCE network load balancer](https://cloud.google.com/compute/docs/load-balancing/network) – Provides TCP connectivity between clients and the NGINX Plus load‑balancing \(LB\) instances in a GCP region, as well as maintaining session persistence for each NGINX Plus instance.
* [GCE instance groups](https://cloud.google.com/compute/docs/instance-groups) – Provide a mechanism for managing a group of VM instances as a unit.
* [GCE health checks](https://cloud.google.com/compute/docs/load-balancing/health-checks) – Maintain high availability of the NGINX Plus LB instances by controlling when GCE creates a new LB instance in the instance group.

![Topology of the all&#x2011;active deployment of NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-all-active-load-balancing-topology.png)

[Session persistence](https://www.nginx.com/products/nginx/load-balancing/#session-persistence) is managed at the network layer by GCE network load balancer \(based on client IP address\) and at the application layer by the NGINX Plus LB instance \(via a session cookie\). When a new client connection enters the GCE network environment, GCE network load balancer assigns it to a specific frontend NGINX Plus LB instance, and the association persists as long as the LB instance is up and functional. The NGINX Plus LB instance forwards the request to a specific application instance in one of the two groups of them, selected using its default Round Robin algorithm. It also issues a cookie to the client to represent the session with that application instance, so that subsequent requests from the client are forwarded to that application instance as long as it is up and running.

This deployment in this guide utilizes two groups of application instances – app-1 and app-2 – to demonstrate [load balancing](https://www.nginx.com/products/nginx/load-balancing/) between different application types, but the application configurations are the same in both groups. The deployment can be very easily adapted to distribute unique connections to different groups of application instances by creating discrete upstream blocks and doing content routing based on URI. Please refer to the reference documentation for details on configuring multiple [upstream server groups](https://nginx.org/en/docs/http/ngx_http_upstream_module.html).

### Prerequisites

This guide assumes that you:

* Have a [Google account](https://accounts.google.com/SignUp) \(a separate GCP or GCE account is not needed\).
* Have enrolled in a [free trial](https://cloud.google.com/free-trial) with available credit or have an established payment account with GCP.
* Have a basic working knowledge of GCE and its GUI control panel:
  * Navigation
  * Creating instances
  * Managing IAM policies
* Understand basic networking.
* Have an NGINX Plus subscription. You can start a [free 30-day trial](https://www.nginx.com/free-trial-request/) if you don’t already have a paid subscription.
* Know how to install NGINX Plus, have a basic understanding of how it performs in load balancing and application delivery modes, and are familiar with its configuration syntax.
* Are familiar with GitHub and know how to [clone a repository](https://help.github.com/articles/cloning-a-repository/).

All component names – projects, instances, templates, instance groups, and so on – are examples only. You can change them as suits your needs.

### Task 1: Creating a Project and Firewall Rules

Create a new GCE project to host the all‑active NGINX Plus deployment.

1. Log into the [GCP Console](https://console.cloud.google.com/) at console.cloud.google.com.
2. The GCP Home &gt; Dashboard tab opens. Its contents depend on whether you have any existing projects.
   * If there are no existing projects, click the  Create a project  button.

     ![Screenshot of the Google Cloud Platform dashboard that appears when there are no existing projects \(creating a project is the first step in configuring NGINX Plus as the Google Cloud load balancer\)](https://www.nginx.com/wp-content/uploads/2016/11/gce-dashboard-no-project.png)

   * If there are existing projects, the name of one of them appears in the upper left of the blue header bar \(in the screenshot, it’s  My Test Project \). Click the project name and select Create project from the menu that opens.

     ![Screenshot of the Google Cloud Platform page that appears when other projects already exist \(creating a project is the first step in configuring NGINX Plus as the Google Cloud load balancer\)](https://www.nginx.com/wp-content/uploads/2016/11/gce-dashboard-existing-project.png)
3. Type your project name in the New Project window that pops up, then click CREATE. We’re naming the project NGINX Plus All-Active-LB.

   ![Screenshot of the New Project pop-up window for naming a new project on the Google Cloud Platform, which is the first step in configuring NGINX Plus as the Google load balancer](https://www.nginx.com/wp-content/uploads/2016/11/gce-new-project-popup.png)

#### Creating Firewall Rules

Create firewall rules that allow access to the HTTP and HTTPS ports on your GCE instances. You’ll attach the rules to all the instances you create for the deployment.

1. Navigate to the Networking &gt; Firewall rules tab and click  +  CREATE FIREWALL RULE. \(The screenshot shows the default rules provided by GCE.\)

   ![Screenshot of the Google Cloud Platform page for defining new firewall rules; when configuring NGINX Plus as the Google Cloud load balancer, we open ports 80, 443, and 8080 for it.](https://www.nginx.com/wp-content/uploads/2016/11/gce-firewall-rules-tab.png)

2. Fill in the fields on the Create a firewall rule screen that opens:

   * **Name** – nginx-plus-http-fw-rule
   * **Description** – Allow access to ports 80, 8080, and 443 on all NGINX Plus instances
   * Source filter – On the drop-down menu, select either Allow from any source \(0.0.0.0/0\), or IP range if you want to restrict access to users on your private network. In the second case, fill in the Source IP ranges field that opens. In the screenshot, we are allowing unrestricted access.
   * Allowed protocols and ports – tcp:80; tcp:8080; tcp:443

     **Note:** As mentioned in the introduction to this guide, opening these ports for your application instances is appropriate only in a test environment. We strongly recommend that before deploying the architecture in production you create a new firewall rule for your application instances that blocks all port access to the external IP address, or disable external IP addresses for the instances to make them accessible only on the internal GCE network.

   * Target tags – nginx-plus-http-fw-rule

   ![Screenshot of the interface for creating a Google Compute Engine \(GCE\) firewall rule, used during deployment of NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-create-firewall-rule.png)

3. Click the  Create  button. The new rule is added to the table on the Firewall rules tab.

### Task 2: Creating Source Instances

Create three GCE source instances that will serve as templates for the instance groups you will create later on: one instance for the NGINX Plus load balancer and two instances for NGINX Plus PHP application servers.

You can create source instances in either of two ways:

* [Based on a standard GCE VM image](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm), in which case you install NGINX Plus manually. This guide uses the Ubuntu LTS image that was most current at the time of publication \(Ubuntu 16.04 LTS\), but you can use any Unix or Linux OS that [NGINX Plus supports](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/nginx/deployment-guides/technical-specs.md).
* [Based on the prebuilt NGINX Plus image](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt) in the Google Marketplace, which at the time of publication runs on Ubuntu 14.04 LTS.

The instructions for the two methods are significantly different, but after you create the source instances all subsequent instructions are the same.

#### Creating Source Instances from VM Images

Create three source VM instances based on a GCE VM image. We’re basing our instances on the Ubuntu 16.04 LTS image.

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the Compute Engine &gt; VM instances tab.
3. Click the  Create instance  button. The Create an instance page opens.

**Creating the First Application Instance from a VM Image**

1. On the Create an instance page, modify or verify the fields and checkboxes as indicated \(a screenshot of the completed page appears in the next step\):
   * **Name** – nginx-plus-app-1
   * **Zone** – The GCP zone that makes sense for your location. We’re using us-west1-a.
   * Machine type – The appropriate size for the level of traffic you anticipate. We’re selecting micro, which is ideal for testing purposes.
   * Boot disk – Click Change. The Boot disk page opens to the OS images subtab. Perform the following steps:

     * Click the radio button for the Unix or Linux image of your choice \(here, Ubuntu 16.04 LTS\).
     * Accept the default values in the Boot disk type and Size \(GB\) fields \(Standard persistent disk and 10 respectively\).
     * Click the  Select  button.

     ![Screenshot of the &apos;Boot disk&apos; page in Google Cloud Platform for selecting the OS on which a VM runs. In the deployment of NGINX Plus as the Google load balancer, we select Ubuntu 16.04 LTS.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-boot-disk.png)

   * Identity and API access – Unless you want more granular control over access, keep the defaults for the Service account field \(Compute Engine default service account\) and Access scopes radio button \(Allow default access\).
   * **Firewall** – Verify that neither check box is checked \(the default\). The firewall rule invoked in the **Tags** field on the Management subtab \(see Step 3 below\) controls this type of access.
2. Click Management, disk, networking, SSH keys to open that set of subtabs. \(The screenshot shows the values entered in the previous step.\)

   ![Screen shot of the &apos;Create an instance&apos; page for an application server in the deployment of NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-create.png)

3. On the Management subtab, modify or verify the fields as indicated:

   * **Description** – NGINX Plus app-1 Image
   * **Tags** – nginx-plus-http-fw-rule
   * **Preemptibility** – Off \(recommended\) \(the default\)
   * Automatic restart – On \(recommended\) \(the default\)
   * On host maintenance – Migrate VM instance \(recommended\) \(the default\)

   ![Screenshot of the Management subtab used during creation of a new VM instance, part of deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-management.png)

4. On the Disks subtab, uncheck the checkbox labeled Delete boot disk when instance is deleted.

   ![Screenshot of the Disks subtab used during creation of a new VM instance, part of deploying NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-disks.png)

5. On the Networking subtab, verify the default settings, in particular Ephemeral for External IP and Off for IP Forwarding.

   ![Screenshot of the Networking subtab used during creation of a new VM instance, part of deploying NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-networking.png)

6. If you are using your own SSH public key instead of the default keys associated with your GCE identity, on the SSH Keys subtab paste the hexadecimal key string into the box that reads Enter entire key data.

   ![Screenshot of the SSH Keys subtab used during creation of a new VM instance, part of deploying NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-ssh-keys.png)

7. Click the  Create  button at the bottom of the Create an instance page.

   The VM instances summary page opens. It can take several minutes for the instance to be created. Wait to continue until the green check mark appears.

   ![Screenshot of the summary page that verifies the creation of a new VM instance, part of deploying NGINX Plus as the load balancer for Google Cloud.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-summary.png)

**Creating the Second Application Instance from a VM Image**

1. On the VM instances summary page, click CREATE INSTANCE.
2. Repeat the steps in [Creating the First Application Instance](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm-app-1) to create the second application instance, specifying the same values as for the first application instance, except:
   * In Step 1, **Name** – nginx-plus-app-2
   * In Step 3, **Description** – NGINX Plus app-2 Image

**Creating the Load-Balancing Instance from a VM Image**

1. On the VM instances summary page, click CREATE INSTANCE.
2. Repeat the steps in [Creating the First Application Instance](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm-app-1) to create the load‑balancing instance, specifying the same values as for the first application instance, except:
   * In Step 1, **Name** – nginx-plus-lb
   * In Step 3, **Description** – NGINX Plus Load Balancing Image

**Configuring PHP and FastCGI on the VM-Based Instances**

Install and configure PHP and FastCGI on the instances.

Repeat these instructions for all three source instances \(nginx-plus-app-1, nginx-plus-app-2, and nginx-plus-lb\).

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Connect to the instance over SSH using the method of your choice. GCE provides a built-in mechanism:

   * Navigate to the Compute Engine &gt; VM instances tab.
   * In the instance’s row in the table, click the triangle icon in the Connect column at the far right and select a method \(for example, Open in browser window\).

   ![Screenshot showing how to connect via SSH to a VM instance, part of deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-ssh.png)

2. Working in the SSH terminal, install PHP 7 \(the default PHP version for Ubuntu 16.04 LTS\) and FastCGI.

   ```text
   $ apt-get install php7.0-fpm
   ```

3. Edit the PHP 7 configuration to bind to a local network port instead of a Unix socket. Using your preferred text editor, remove the following line from /etc/php/7.0/fpm/pool.d:

   ```text
   listen = /run/php/php7.0-fpm.sock
   ```

   and replace it with these two lines:

   ```text
   listen = 127.0.0.1:9000
   listen.allowed_clients = 127.0.0.1
   ```

4. Restart PHP:

   ```text
   $ service php7.0-fpm restart
   ```

5. Leave the SSH connection open for reuse in the next section.

**Installing and Configuring NGINX Plus on the VM-Based Instances**

Now install NGINX Plus and download files that are specific to the all‑active deployment:

* An NGINX Plus configuration file customized for the function performed by the instance \(application server or load balancer\)
* A set of content files \(HTML, images, and so on\) served by the application servers in the deployment

Both the configuration and content files are available at the [NGINX, Inc. GitHub repository](https://github.com/nginxinc/NGINX-Demos/tree/master/gce-nginx-plus-deployment-guide-files).

Repeat these instructions for all three source instances \(nginx-plus-app-1, nginx-plus-app-2, and nginx-plus-lb\).

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Install NGINX Plus. For instructions, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/nginx/deployment-guides/admin-guide/installing-nginx/installing-nginx-plus.md).
2. Clone the GitHub repository for the [all‑active load balancing deployment](https://github.com/nginxinc/NGINX-Demos/tree/master/gce-nginx-plus-deployment-guide-files). \(Instructions for downloading the files directly from the GitHub repository are provided below, in case you prefer not to clone it.\)
3. Copy the contents of the usr\_share\_nginx subdirectory from the cloned repository to the local /usr/share/nginx directory, creating the local directory if necessary. \(If you choose not to clone the repository, you need to download each file from the GitHub repository individually.\)
4. Copy the appropriate configuration file from the etc\_nginx\_conf.d subdirectory of the cloned repository to /etc/nginx/conf.d:
   * On both nginx-plus-app-1 and nginx-plus-app-2, copy gce-all-active-app.conf.

     You can also run the following commands to download the configuration file directly from the GitHub repository:

     ```text
     $ cd /etc/nginx/conf.d/
     $ curl -o gce-all-active-app.conf https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-app.conf
     ```

     or

     ```text
     $ cd /etc/nginx/conf.d/
     $ wget https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-app.conf
     ```

   * On nginx-plus-lb, copy gce-all-active-lb.conf.

     You can also run the following commands to download the configuration file directly from the GitHub repository:

     ```text
     $ cd /etc/nginx/conf.d/
     $ curl -o gce-all-active-lb.conf https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-lb.conf
     ```

     or

     ```text
     $ cd /etc/nginx/conf.d/  
     $ wget https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-lb.conf
     ```
5. On the LB instance \(nginx-plus-lb\), use your preferred text editor to open gce-all-active-lb.conf and change the `server` directives in the `upstream` block to reference the internal IP addresses of the nginx-plus-app-1 and nginx-plus-app-2 instances \(substitute the address for the expression in angle brackets\). No action is required on the two application instances themselves.

   You can look up internal IP addresses in the Internal IP column of the table on the Compute Engine &gt; VM instances summary page.

   ```text
   upstream upstream_app_pool {
       server <internal IP address of nginx-plus-app-1>;
       server <internal IP address of nginx-plus-app-2>;
       zone upstream-apps 64k;
       sticky cookie GCPPersist expires=300;
   }
   ```

   Directive documentation: [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`sticky cookie`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream), [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

6. Rename **default.conf** to **default.conf.bak** so that NGINX Plus does not load it. The configuration files provided for the all‑active deployment include equivalent instructions plus additional function‑specific directives.

   ```text
   $ mv default.conf default.conf.bak
   ```

7. Enable the NGINX Plus [live activity monitoring](https://www.nginx.com/products/nginx/live-activity-monitoring/) dashboard for the instance by copying status.html from the etc\_nginx\_conf.d subdirectory of the cloned repository to /etc/nginx/conf.d.

   You can also run the following commands to download the configuration file directly from the GitHub repository:

   ```text
   $ cd /etc/nginx/conf.d/
   $ curl -o status.conf https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/status.conf
   ```

   or

   ```text
   $ cd /etc/nginx/conf.d/
   $ wget https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/status.conf
   ```

8. Validate the NGINX Plus configuration and restart NGINX Plus:

   ```text
   $ nginx -t
   $ nginx -s reload
   ```

9. Verify the instance is working by accessing it at its external IP address. \(As previously noted, we recommend blocking access to the external IP addresses of the application instances in a production environment.\) The external IP address for the instance appears on the Compute Engine &gt; VM instances summary page, in the External IP column of the table.

   * Access the index.html page either in a browser or by running this `curl` command.

   ```text
   $ curl http://<external-IP-address>
   ```

   * Access its NGINX Plus live activity monitoring dashboard in a browser, at:

     https//:_external-IP-address_:8080/status.html

10. Proceed to [Task 3: Creating “Gold” Images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#gold).

#### Creating Source Instances from Prebuilt NGINX Plus Images

Create three source instances based on a prebuilt NGINX Plus image running on Ubuntu 14.04 LTS, available in the Google Marketplace. Google requires that you provision the first instance in the GCP Marketplace. Then you can clone the additional two instances from the first one.

**Creating the First Application Instance from a Prebuilt Image**

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the GCP Marketplace and search for nginx plus.
3. Click the NGINX Plus box in the results area.

   ![Screenshot of NGINX Plus in the Google Cloud Platform Marketplace; from here, you can create a prebuilt NGINX Plus VM instance when deploying NGINX Plus as the load balancer for Google Cloud.](https://www.nginx.com/wp-content/uploads/2018/07/nginx-plus-in-gcp-marketplace.png)

4. On the NGINX Plus page that opens, click the  Launch on Compute Engine  button.
5. Fill in the fields on the New NGINX Plus deployment page as indicated.

   * Deployment name – nginx-plus-app-1
   * **Zone** – The GCP zone that makes sense for your location. We’re using us-west1-a.
   * Machine type – The appropriate size for the level of traffic you anticipate. We’re selecting micro, which is ideal for testing purposes.
   * Disk type – Standard Persistent Disk \(the default\)
   * Disk size in GB – 10 \(the default and minimum allowed\)
   * Network name – default
   * Subnetwork name – default
   * **Firewall** – Verify that the Allow HTTP traffic checkbox is checked.

   [![Screenshot of the page for creating a prebuilt NGINX Plus VM instance when deploying NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2018/07/gce-marketplace-new-deployment.png)](https://www.nginx.com/wp-content/uploads/2018/07/gce-marketplace-new-deployment.png)

6. Click the  Deploy  button.

   It can take several minutes for the instance to deploy. Wait until the green check mark and confirmation message appear before continuing.

   ![Screenshot of the page that confirms the creation of a prebuilt NGINX Plus VM instance when deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2018/07/gce-marketplace-instance-deployed.png)

7. Navigate to the Compute Engine &gt; VM instances tab and click nginx-plus-app-1-vm in the Name column in the table. \(The -vm suffix is added automatically to the name of the newly created instance.\)

   ![Screenshot showing how to access the page where configuration details for a VM instance can be modified during deployment of NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-app-1-vm-select.png)

8. On the VM instances page that opens, click EDIT at the top of the page. In fields that can be edited, the value changes from static text to text boxes, drop‑down menus, and checkboxes.
9. Modify or verify the indicated editable fields \(non‑editable fields are not listed\):

   * **Tags** – If a default tag appears in the field \(for example, nginx-plus-app-1-tcp-80\), click the X after its name to remove it, and type in nginx-plus-http-fw-rule.
   * External IP – Ephemeral \(the default\)
   * Boot disk and local disks – Uncheck the checkbox labeled Delete boot disk when when instance is deleted.
   * Additional disks – No changes
   * **Network** – We recommend keeping the default settings, but if you need to change them \(if, for example, you’re configuring a production environment\), click default and then EDIT on the Network details page that opens. After making your changes click the  Save  button.
   * **Firewall** – Verify that neither check box is checked \(the default\). The firewall rule named in the **Tags** field that’s above on the current page \(see the first bullet in this list\) controls this type of access.
   * Automatic restart – On \(recommended\) \(the default\)
   * On host maintenance – Migrate VM instance \(recommended\) \(the default\)
   * Custom metadata – No changes
   * SSH Keys – If you are using your own SSH public key instead of the default keys associated with your GCE identity, paste the hexadecimal key string into the box that reads Enter entire key data.
   * Serial port – Verify that the check box labeled Enable connecting to serial ports is not checked \(the default\).

   The screenshot shows the results of your changes, omitting some fields than cannot be edited or for which we recommend retaining the defaults.

   ![Screenshot showing the configuration modifications for a VM instance being deployed as part of setting up NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-app-1-vm-edited.png)

10. Click the  Save  button.

**Creating the Second Application Instance from a Prebuilt Image**

Create the second application instance by cloning the first one.

1. Navigate back to the summary page on the Compute Engine &gt; VM instances tab \(click the arrow that is circled in the following figure\).

   ![Screenshot showing how to return to the VM instance summary page during deployment of NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-vm-instances-back-arrow.png)

2. Click nginx-plus-app-1-vm in the Name column of the table \(shown in the screenshot in Step 7 of [Creating the First Application Instance](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt-app-1)\).
3. On the VM instances page that opens, click CLONE at the top of the page.
4. On the Create an instance page that opens, modify or verify the fields and checkboxes as indicated:
   * **Name** – nginx-plus-app-2-vm. Here we’re adding the -vm suffix to make the name consistent with the first instance; GCE does not add it automatically when you clone an instance.
   * **Zone** – The GCP zone that makes sense for your location. We’re using us-west1-a.
   * Machine type – The appropriate size for the level of traffic you anticipate. We’re selecting f1-micro, which is ideal for testing purposes.
   * Boot disk type – New 10 GB standard persistent disk \(the value inherited from nginx-plus-app-1-vm\)
   * Identity and API access – Set the Access scopes radio button to Allow default access and accept the default values in all other fields. If you want more granular control over access than is provided by these settings, modify the fields in this section as appropriate.
   * **Firewall** – Verify that neither check box is checked \(the default\).
5. Click Management, disk, networking, SSH keys to open that set of subtabs.
6. Verify the following settings on the subtabs, modifying them as necessary:
   * Management – In the **Tags** field: nginx-plus-http-fw-rule
   * Disks – The Deletion rule checkbox \(labeled Delete boot disk when instance is deleted\) is not checked
7. Click the  Create  button.

**Creating the Load-Balancing Instance from a Prebuilt Image**

Create the source load‑balancing instance by cloning the first instance again.

Repeat Steps 2 through 7 of [Creating the Second Application Instance](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt-app-2). In Step 4, specify nginx-plus-lb-vm as the name.

**Configuring PHP and FastCGI on the Prebuilt-Based Instances**

Install and configure PHP and FastCGI on the instances.

Repeat these instructions for all three source instances \(nginx-plus-app-1-vm, nginx-plus-app-2-vm, and nginx-plus-lb-vm\).

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Connect to the instance over SSH using the method of your choice. GCE provides a built‑in mechanism:

   * Navigate to the Compute Engine &gt; VM instances tab.
   * In the row for the instance in the table, click the triangle icon in the Connect column at the far right and select a method \(for example, Open in browser window\).

   The screenshot shows instances based on the prebuilt NGINX Plus images.

   ![Screenshot showing how to connect via SSH to a VM instance, part of deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-prebuilt-instance-ssh.png)

2. Working in the SSH terminal, install PHP 5 \(the default PHP version for Ubuntu 14.04 LTS\) and FastCGI.

   ```text
   $ apt-get install php5-fpm
   ```

3. Edit the PHP 5 configuration to bind to a local network port instead of a Unix socket. Using your preferred text editor, remove the following line from /etc/php5/fpm/pool.d:

   ```text
   Listen = /run/php/php5-fpm.sock
   ```

   and replace it with these two lines:

   ```text
   Listen = 127.0.0.1:9000
   Listen.allowed_clients = 127.0.0.1
   ```

4. Restart PHP:

   ```text
   $ service php5-fpm restart
   ```

5. Leave the SSH connection open for reuse in the next section.

**Configuring NGINX Plus on the Prebuilt-Based Instances**

Now download files that are specific to the all‑active deployment:

* An NGINX Plus configuration file customized for the function the instance performs \(application server or load balancer\)
* A set of content files \(HTML, images, and so on\) served by the application servers in the deployment

Both the configuration and content files are available at the [NGINX, Inc. GitHub repository](https://github.com/nginxinc/NGINX-Demos/tree/master/gce-nginx-plus-deployment-guide-files).

Repeat these instructions for all three source instances \(nginx-plus-app-1-vm, nginx-plus-app-2-vm, and nginx-plus-lb-vm\).

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Clone the GitHub repository for the [all‑active load balancing deployment](https://github.com/nginxinc/NGINX-Demos/tree/master/gce-nginx-plus-deployment-guide-files). \(Instructions for downloading the files directly from the GitHub repository are provided below, in case you prefer not to clone it.\)
2. Copy the contents of the usr\_share\_nginx subdirectory from the cloned repository to the local /usr/share/nginx directory, creating the local directory if necessary. \(If you choose not to clone the repository, you need to download each file from the GitHub repository individually.\)
3. Copy the appropriate configuration file from the etc\_nginx\_conf.d subdirectory of the cloned repository to /etc/nginx/conf.d:
   * On both nginx-plus-app-1-vm and nginx-plus-app-2-vm, copy gce-all-active-app.conf.

     You can also run the following commands to download the configuration file directly from the GitHub repository:

     ```text
     $ cd /etc/nginx/conf.d/
     $ curl -o gce-all-active-app.conf https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-app.conf
     ```

     or

     ```text
     $ cd /etc/nginx/conf.d/
     $ wget https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-app.conf
     ```

   * On nginx-plus-lb-vm, copy gce-all-active-lb.conf.

     You can also run the following commands to download the configuration file directly from the GitHub repository:

     ```text
     $ cd /etc/nginx/conf.d/
     $ curl -o gce-all-active-lb.conf https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-lb.conf
     ```

     or

     ```text
     $ cd /etc/nginx/conf.d/
     $ wget https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/gce-all-active-lb.conf
     ```
4. On the LB instance \(nginx-plus-lb-vm\), use your preferred text editor to open gce-all-active-lb.conf and change the `server` directives in the `upstream` block to reference the internal IP addresses of the nginx-plus-app-1-vm and nginx-plus-app-2-vm instances. \(No action is required on the two application instances themselves.\)

   You can look up internal IP addresses in the Internal IP column of the table on the Compute Engine &gt; VM instances summary page.

   ```text
   upstream upstream_app_pool {
       server <internal IP address of nginx-plus-app-1-vm>;
       server <internal IP address of nginx-plus-app-2-vm>;
       zone upstream-apps 64k;
       sticky cookie GCPPersist expires=300;
   }
   ```

   Directive documentation: [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`sticky cookie`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream), [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

5. Rename **default.conf** to **default.conf.bak** so that NGINX Plus does not load it. The configuration files provided for the all‑active deployment include equivalent instructions plus additional function‑specific directives.

   ```text
   $ mv default.conf default.conf.bak
   ```

6. Enable the NGINX Plus [live activity monitoring](https://www.nginx.com/products/nginx/live-activity-monitoring/) dashboard for the instance by copying status.html from the etc\_nginx\_conf.d subdirectory of the cloned repository to /etc/nginx/conf.d.

   You can also run the following commands to download the configuration file directly from the GitHub repository:

   ```text
   $ cd /etc/nginx/conf.d/
   $ curl -o status.conf https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/status.conf
   ```

   or

   ```text
   $ cd /etc/nginx/conf.d/
   $ wget https://github.com/nginxinc/NGINX-Demos/blob/master/gce-nginx-plus-deployment-guide-files/etc_nginx_conf.d/status.conf
   ```

7. Validate the NGINX Plus configuration and restart NGINX Plus:

   ```text
   $ nginx -t
   $ nginx -s reload
   ```

8. Verify the instance is working by accessing it at its external IP address. \(As previously noted, we recommend blocking access to the external IP addresses of the application instances in a production environment.\) The external IP address for the instance appears on the Compute Engine &gt; VM instances summary page, in the External IP column of the table.
   * Access the index.html page either in a browser or by running this `curl` command.

     ```text
     $ curl http://<external-IP-address-of-NGINX-Plus-server>
     ```

   * Access the NGINX Plus live activity monitoring dashboard in a browser, at:

     https//:_external-IP-address-of-NGINX-Plus-server_:8080/dashboard.html
9. Proceed to [Task 3: Creating “Gold” Images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#gold).

### Task 3: Creating “Gold” Images

Create _gold images_, which are base images that GCE clones automatically when it needs to scale up the number of instances. They are derived from the instances you created in [Creating Source Instances](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source). Before creating the images, you must delete the source instances to break the attachment between them and the disk \(you can’t create an image from a disk that’s attached to a VM instance\).

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the Compute Engine &gt; VM instances tab.
3. In the table, select all three instances:
   * If you created source instances from [VM \(Ubuntu\) images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm): nginx-plus-app-1, nginx-plus-app-2, and nginx-plus-lb
   * If you created source instances from [prebuilt NGINX Plus images](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt): nginx-plus-app-1-vm, nginx-plus-app-2-vm, and nginx-plus-lb-vm
4. Click STOP in the top toolbar to stop the instances.

   ![Screenshot of the toolbar on the Google Compute Engine page that lists VM instances, used when deploying NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-vm-instances-toolbar.png)

5. Click DELETE in the top toolbar to delete the instances.

   **Note:** If the pop‑up confirmation window warns that the boot disk for any instance will also be deleted, cancel the deletion, and perform these steps for each affected instance:

   * Navigate to the Compute Engine &gt; VM instances tab and click the instance in the Name column in the table. \(The screenshot shows nginx-plus-app-1-vm.\)

     ![Screenshot showing how to access the page where configuration details for a VM instance can be modified during deployment of NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-app-1-vm-select.png)

   * On the VM instances page that opens, click EDIT at the top of the page. In fields that can be edited, the value changes from static text to text boxes, drop‑down menus, and checkboxes.
   * In the Boot disk and local disks field, uncheck the checkbox labeled Delete boot disk when when instance is deleted.
   * Click the  Save  button.
   * On the VM instances summary page, select the instance in the table and click DELETE in the top toolbar to delete it.

6. Navigate to the Compute Engine &gt; Images tab.
7. Click \[+\] CREATE IMAGE.
8. On the Create an image page that opens, modify or verify the fields as indicated:
   * **Name** – nginx-plus-app-1-image
   * **Family** – Leave the field empty
   * **Description** – NGINX Plus Application 1 Gold Image
   * **Encryption** – Automatic \(recommended\) \(the default\)
   * **Source** – Disk \(the default\)
   * Source disk – nginx-plus-app-1 or nginx-plus-app-1-vm, depending on the method you used to create source instances \(select the source instance from the drop‑down menu\)
9. Click the  Create  button.
10. Repeat Steps 7 through 9 to create a second image with the following values \(retain the default values in all other fields\):
    * **Name** – nginx-plus-app-2-image
    * **Description** – NGINX Plus Application 2 Gold Image
    * Source disk – nginx-plus-app-2 or nginx-plus-app-2-vm, depending on the method you used to create source instances \(select the source instance from the drop‑down menu\)
11. Repeat Steps 7 through 9 to create a third image with the following values \(retain the default values in all other fields\):
    * **Name** – nginx-plus-lb-image
    * **Description** – NGINX Plus LB Gold Image
    * Source disk – nginx-plus-lb or nginx-plus-lb-vm, depending on the method you used to create source instances \(select the source instance from the drop‑down menu\)
12. Verify that the three images appear at the top of the table on the Compute Engine &gt; Images tab.

### Task 4: Creating Instance Templates

Create _instance templates_, which are the compute workloads that are created in instance groups, either manually or automatically when GCE detects a failure.

#### Creating the First Application Instance Template

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the Compute Engine &gt; Instance templates tab.
3. Click the  Create instance template  button.
4. On the Create an instance template page that opens, modify or verify the fields as indicated:
   * **Name** – nginx-plus-app-1-instance-template
   * Machine type – The appropriate size for the level of traffic you anticipate. We’re selecting micro, which is ideal for testing purposes.
   * Boot disk – Click Change. The Boot disk page opens. Perform the following steps:
     * Open the Custom Images subtab.

       ![Screenshot of the &apos;Boot disk&apos; page in Google Cloud Platform for selecting the source instance of a new instance template, part of deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-instance-template-boot-disk.png)

     * Select NGINX Plus All-Active-LB from the drop-down menu labeled Show images from.
     * Click the nginx-plus-app-1-image radio button.
     * Accept the default values in the Boot disk type and Size \(GB\) fields \(Standard persistent disk and 10 respectively\).
     * Click the  Select  button.
   * Identity and API access – Unless you want more granular control over access, keep the defaults in the Service account field \(Compute Engine default service account\) and Access scopes field \(Allow default access\).
   * **Firewall** – Verify that neither check box is checked \(the default\). The firewall rule invoked in the **Tags** field on the Management subtab \(see Step 6 below\) controls this type of access.
5. Click Management, disk, networking, SSH keys \(indicated with a red arrow in the following screenshot\) to open that set of subtabs.

   ![Screenshot of the interface for creating a Google Compute Engine \(GCE\) instance template, used during deployment of NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-create-instance-template.png)

6. On the Management subtab, modify or verify the fields as indicated:

   * **Description** – NGINX Plus app-1 Instance Template
   * **Tags** – nginx-plus-http-fw-rule
   * **Preemptibility** – Off \(recommended\) \(the default\)
   * Automatic restart – On \(recommended\) \(the default\)
   * On host maintenance – Migrate VM instance \(recommended\) \(the default\)

   ![Screenshot of the Management subtab used during creation of a new VM instance template, part of deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-instance-template-management.png)

7. On the Disks subtab, verify that the checkbox labeled Delete boot disk when instance is deleted is checked.

   Instances created from this template are ephemeral instantiations of the gold image, so we want GCE to reclaim the disk when the instance is terminated. New instances are always based on the gold image, so there is no reason to have the instantiations persist on disk when the instance is deleted.

   ![Screenshot of the Disks subtab used during creation of a new VM instance template, part of deploying NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-instance-template-disks.png)

8. On the Networking subtab, verify the default settings of Ephemeral for External IP and Off for IP Forwarding.

   ![Screenshot of the Networking subtab used during creation of a new VM instance template, part of deploying NGINX Plus as the Google load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-instance-template-networking.png)

9. If you are using your own SSH public key instead of the default keys associated with your GCE identity, on the SSH Keys subtab paste the hexadecimal key string into the box that reads Enter entire key data.

   ![Screenshot of the SSH Keys subtab used during creation of a new VM instance, part of deploying NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-ubuntu-instance-ssh-keys.png)

10. Click the  Create  button.

#### Creating the Second Application Instance Template

1. On the Instance templates summary page, click CREATE INSTANCE TEMPLATE.
2. Repeat Steps 4 through 10 of [Creating the First Application Instance Template](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#templates-app-1) to create a second application instance template, specifying the same values as for the first instance template, except as noted:
   * In Step 4:
     * **Name** – nginx-plus-app-2-instance-template
     * Boot disk – Click the nginx-plus-app-2-image radio button
   * In Step 6, **Description** – NGINX Plus app-2 Instance Template

#### Creating the Load-Balancing Instance Template

1. On the Instance templates summary page, click CREATE INSTANCE TEMPLATE.
2. Repeat Steps 4 through 10 of [Creating the First Application Instance Template](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#templates-app-1) to create the load‑balancing instance template, specifying the same values as for the first instance template, except as noted:
   * In Step 4:
     * **Name** – nginx-plus-lb-instance-template.
     * Boot disk – Click the nginx-plus-lb-image radio button
   * In Step 6, **Description** – NGINX Plus Load‑Balancing Instance Template

### Task 5: Creating Image Health Checks

Define the simple HTTP health check that GCE uses to verify that each NGINX Plus LB image is running correctly \(and to re-create an LB instance that is not running correctly\).

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the Compute Engine &gt; Health checks tab.
3. Click the  Create a health check  button.
4. On the Create a health check page that opens, modify or verify the fields as indicated:
   * **Name** – nginx-plus-http-health-check
   * **Description** – Basic HTTP health check to monitor NGINX Plus instances
   * **Protocol** – HTTP \(the default\)
   * **Port** – 80 \(the default\)
   * Request path – /status-old.html
5. If the Health criteria section is not already open, click More.
6. Modify or verify the fields as indicated:
   * Check interval – 10 seconds
   * **Timeout** – 10 seconds
   * Healthy threshold – 2 consecutive successes \(the default\)
   * Unhealthy threshold – 10 consecutive failures
7. Click the  Create  button.

   ![Screenshot of the interface for creating a health check in Google Compute Engine \(GCE\), which Google network load balancer uses to monitor NGINX Plus as the Google cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-create-health-check.png)

### Task 6: Creating Instance Groups

Create three independent instance groups, one for each type of function-specific instance.

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the Compute Engine &gt; Instance groups tab.
3. Click the  Create instance group  button.

#### Creating the First Application Instance Group

1. On the Create a new instance group page that opens, modify or verify the fields as indicated. Ignore fields that are not mentioned:
   * **Name** – nginx-plus-app-1-instance-group
   * **Description** – Instance group to host NGINX Plus app-1 instances
   * **Location** –
     * Click the Single-zone radio button \(the default\).
     * **Zone** – The GCP zone you specified when you created source instances \(Step 1 of [Creating the First Application Instance from a VM Image](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm-app-1) or Step 5 of [Creating the First Application Instance from a Prebuilt Image](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt)\). We’re using us-west1-a.
   * Creation method – Use instance template radio button \(the default\)
   * Instance template – nginx-plus-app-1-instance-template \(select from the drop-down menu\)
   * **Autoscaling** – Off \(the default\)
   * Number of instances – 2
   * Health check – nginx-plus-http-health-check \(select from the drop-down menu\)
   * Initial delay – 300 seconds \(the default\)
2. Click the  Create  button.

   ![Screenshot of the interface for creating a Google Compute Engine \(GCE\) instance group, used during deployment of NGINX Plus as the load balancer for Google Cloud.](https://www.nginx.com/wp-content/uploads/2016/11/gce-create-instance-group.png)

#### Creating the Second Application Instance Group

1. On the Instance groups summary page, click CREATE INSTANCE GROUP.
2. Repeat the steps in [Creating the First Application Instance Group](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#groups-app-1) to create a second application instance group, specifying the same values as for the first instance template, except for these fields:
   * **Name** – nginx-plus-app-2-instance-group
   * **Description** – Instance group to host NGINX Plus app-2 instances
   * Instance template – nginx-plus-app-2-instance-template \(select from the drop-down menu\)

#### Creating the Load-Balancing Instance Group

1. On the Instance groups summary page, click CREATE INSTANCE GROUP.
2. Repeat the steps in [Creating the First Application Instance Group](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#groups-app-1) to create the load‑balancing instance group, specifying the same values as for the first instance template, except for these fields:
   * **Name** – nginx-plus-lb-instance-group
   * **Description** – Instance group to host NGINX Plus load balancing instances
   * Instance template – nginx-plus-lb-instance-template \(select from the drop-down menu\)

#### Updating and Testing the NGINX Plus Configuration

Update the NGINX Plus configuration on the two LB instances \(nginx-plus-lb-instance-group-\[a...z\]\) to list the internal IP addresses of the four application servers \(two instances each of nginx-plus-app-1-instance-group-\[a...z\] and nginx-plus-app-2-instance-group-\[a...z\]\).

Repeat these instructions for both LB instances.

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Connect to the LB instance over SSH using the method of your choice. GCE provides a built-in mechanism:
   * Navigate to the Compute Engine &gt; VM instances tab.
   * In the row for the instance in the table, click the triangle icon in the Connect column at the far right and select a method \(for example, Open in browser window\).
2. Working in the SSH terminal, use your preferred text editor to modify gce-all-active-lb.conf, changing the `server` directives in the `upstream` block to reference the internal IP addresses of the two nginx-plus-app-1-instance-group-\[a...z\] instances and the two nginx-plus-app-2-instance-group-\[a...z\] instances. You can look up the addresses in the Internal IP column of the table on the Compute Engine &gt; VM instances summary page. For example:

   ```text
   upstream upstream_app_pool {
       zone upstream-apps 64k;
       
       server 10.10.10.1;
       server 10.10.10.2;
       server 10.10.10.3;
       server 10.10.10.4;

       sticky cookie GCPPersist expires=300;
   }
   ```

   Directive documentation: [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`sticky cookie`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream), [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

3. Validate the NGINX Plus configuration and restart NGINX Plus:

   ```text
   $ nginx -t
   $ nginx -s reload
   ```

4. Verify that the four application instances are receiving traffic and responding, by accessing the NGINX Plus live activity monitoring dashboard on the load‑balancing instance \(nginx-plus-lb-instance-group-\[a...z\]\). You can look up the instance’s external IP address on the Compute Engine &gt; VM instances summary page, in the External IP column of the table.

   https//:_LB-external-IP-address_:8080/status.html

5. Verify that NGINX Plus is load balancing traffic among the four application instance groups, by running this command on a separate client machine:

   ```text
   $ while true; do curl -s <LB-external-IP-address> | grep Server: ;done
   ```

   If load balancing is working properly, the unique **Server** field from the index page for each application instance appears in turn.

### Task 7: Configuring GCE Network Load Balancer

Set up GCE network load balancer to distribute incoming client traffic to the NGINX Plus LB instances. The first step is to reserve the static IP address that GCE network load balancer advertises to clients.

1. Verify that the NGINX Plus All-Active-LB project is still selected in the Google Cloud Platform header bar.
2. Navigate to the Networking &gt; External IP addresses tab.
3. Click the  Reserve static address  button.
4. On the Reserve a static address page that opens, modify or verify the fields as indicated:
   * **Name** – nginx-plus-network-lb-static-ip
   * **Description** – Static IP address for Network LB frontend to NGINX Plus LB instances
   * **Type** – Click the Regional radio button \(the default\)
   * **Region** – The GCP zone you specified when you created source instances \(Step 1 of [Creating the First Application Instance from a VM Image](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-vm-app-1) or Step 5 of [Creating the First Application Instance from a Prebuilt Image](https://docs.nginx.com/nginx/deployment-guides/google-cloud-platform/high-availability-all-active/#source-prebuilt)\). We’re using us-west1.
   * Attached to – None \(the default\)
5. Click the  Reserve  button.

   ![Screenshot of the interface for reserving a static IP address for Google Compute Engine network load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-reserve-static-address.png)

6. Navigate to the Networking &gt; Load balancing tab.
7. Click the  Create load balancer  button.
8. On the Load balancing page that opens, click Start configuration in the TCP Load Balancing box.
9. On the page that opens, click the From Internet to my VMs and No \(TCP\) radio buttons \(the defaults\).
10. Click the  Continue  button. The New TCP load balancer page opens.
11. In the **Name** field, type nginx-plus-network-lb-frontend.
12. Click Backend configuration in the left column to open the Backend configuration interface in the right column. Fill in the fields as indicated:

    * **Region** – The GCP region you specified in Step 4. We’re using us-west1.
    * **Backends** – With Select existing instance groups selected, select nginx-plus-lb-instance-group from the drop-down menu
    * Backup pool – None \(the default\)
    * Failover ratio – 10 \(the default\)
    * Health check – nginx-plus-http-health-check
    * Session affinity – Client IP

    ![Screenshot of the interface for backend configuration of GCE network load balancer, used during deployment of NGINX Plus as the Google Cloud Platform load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-backend-configuration.png)

13. Click Frontend configuration in the left column to open the Frontend configuration interface in the right column.
14. Create three Protocol-IP-Port tuples, each with:
    * **Protocol** – TCP
    * **IP** – The address you reserved in Step 5, selected from the drop-down menu \(if there is more than one address, select the one labeled in parentheses with the name you specified in Step 5\)
    * **Port** – 80, 8080, and 443 in the three tuples respectively
15. Click the  Create  button.

    ![Screenshot of the interface for frontend configuration of GCE network load balancer, used during deployment of NGINX Plus as the Google Cloud load balancer.](https://www.nginx.com/wp-content/uploads/2016/11/gce-frontend-configuration.png)

### Task 8: Testing the All-Active Load Balancing Deployment

Verify that GCE network load balancer is properly routing traffic to both NGINX Plus LB instances.

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

Working on a separate client machine, run this command, using the static IP address you set in the previous section for GCE network load balancer:

```text
$ while true; do curl -s <GCE-Network-LB-external-static-IP-address> | grep Server: ;done
```

Alternatively, you can use a web browser to access this URL:

**http://**_GCE-Network-LB-external-static-IP-address_

If load balancing is working properly, the unique **Server** field from the index page for each application instance appears in turn.

To verify that high availability is working:

1. Connect to one of the instances in the nginx-plus-lb-instance-group over SSH and run this command to force it offline:

   ```text
   $ iptables -A INPUT -p tcp --destination-port 80 -j DROP
   ```

2. Verify that with one LB instance offline, the other LB instance still forwards traffic to the application instances \(there might be a delay before GCE network load balancer detects that the first instance is offline\). Continue monitoring and verify that GCE network load balancer then re-creates the first LB instance and brings it online.
3. When the LB instance is back online, run this command to return it to its working state:

   ```text
   $ iptables -F
   ```

#### Revision History

* Version 3 \(July 2018\) – Updates for Google Cloud Platform Marketplace
* Version 2 \(April 2018\) – Standardized information about root privilege and links to directive documentation
* Version 1 \(November 2016\) – Initial version \(NGINX Plus R11\)

