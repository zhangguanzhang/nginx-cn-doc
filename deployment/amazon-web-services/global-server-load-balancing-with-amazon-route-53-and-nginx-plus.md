# Global Server Load Balancing with Amazon Route 53 and NGINX Plus



This deployment guide explains how to configure global server load balancing \(GSLB\) of traffic for web domains hosted in Amazon [Elastic Compute Cloud](https://aws.amazon.com/ec2/) \(EC2\). For high availability and improved performance, you set up multiple backend servers \(web servers, application servers, or both\) for a domain in two or more AWS regions. Within each region, NGINX Plus load balances traffic across the backend servers.

The AWS Domain Name System \(DNS\) service, [Amazon Route 53](https://aws.amazon.com/route53/), performs global server load balancing by responding to a DNS query from a client with the DNS record for the region that is closest to the client and hosts the domain. For best performance and predictable failover between regions, “closeness” is measured in terms of network latency rather than the actual geographic location of the client.

* [About NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#about-nginx)
* [Topology for Global Load Balancing with Amazon Route 53 and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#topology)
* [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#prereqs)
* [Configuring Global Server Load Balancing](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#gslb)
  * [Creating a Hosted Zone](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#hosted-zone)
  * [Linking the Domain to EC2 Instances](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#link-instances)
  * [Configuring Health Checks for Route 53 Failover](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#route-53-health-checks)
  * [Configuring NGINX Plus Application Health Checks](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#nginx-plus-health-checks)

The [Appendix](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#appendix) provides instructions for creating EC2 instances with the names used in this guide, and installing and configuring the NGINX software on them.

### About NGINX Plus

[NGINX Plus](https://www.nginx.com/products/nginx) is the commercially supported version of the [NGINX Open Source](https://nginx.org/en) software. NGINX Plus is a complete application delivery platform, extending the power of NGINX with a host of enterprise‑ready capabilities that enhance an AWS application server deployment and are instrumental to building web applications at scale:

* [Full‑featured HTTP, TCP, and UDP load balancing](https://www.nginx.com/products/nginx/load-balancing/)
* [Intelligent session persistence](https://www.nginx.com/products/nginx/load-balancing/#session-persistence)
* [High‑performance reverse proxy](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/nginx/deployment-guides/admin-guide/web-server/reverse-proxy.md)
* [Caching and offload of dynamic and static content](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/nginx/deployment-guides/admin-guide/content-cache/content-caching.md)
* [Adaptive streaming to deliver audio and video to any device](https://www.nginx.com/products/nginx/streaming-media/)
* [Application-aware health checks](https://www.nginx.com/products/nginx/load-balancing/#health-checks) and [high availability](https://www.nginx.com/products/nginx/high-availability/)
* [Advanced activity monitoring available via a dashboard or API](https://www.nginx.com/products/nginx/live-activity-monitoring/)
* [Management and real‑time configuration changes with DevOps‑friendly tools](https://www.nginx.com/products/nginx/load-balancing/#load-balancing-api)

### Topology for Global Load Balancing with Amazon Route 53 and NGINX Plus

The setup for global server load balancing \(GSLB\) in this guide combines Amazon Elastic Compute Cloud \(EC2\) instances, Amazon Route 53, NGINX Plus instances, and NGINX Open Source instances.

**Note:** [Global server load balancing](https://www.nginx.com/resources/glossary/global-server-load-balancing/) is also sometimes called _global load balancing_ \(GLB\). The terms are used interchangeably in this guide.

![Diagram showing a topology for global server load balancing \(GSLB\). Eight backend servers, four in each of two regions, host the content for a domain. Two NGINX Plus load balancers in each region route traffic to the backend servers. For each client requesting DNS information for the domain, Amazon Route 53 provides the DNS record for the region closest to the client.](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-topology.png)

Route 53 is a Domain Name System \(DNS\) service that performs global server load balancing by routing each request to the AWS region closest to the requester’s location. This guide uses two regions: US West \(Oregon\) and US East \(N. Virginia\).

In each region, two or more NGINX Plus load balancers are deployed in a high‑availability \(HA\) configuration. In this guide, there are two NGINX Plus load balancer instances per region. You can also use NGINX Open Source for this purpose, but it lacks the [application health checks](https://www.nginx.com/products/application-health-checks/) that make for more precise error detection. For simplicity, we’ll refer to NGINX Plus load balancers throughout this guide, noting when features specific to NGINX Plus are used.

The NGINX Plus instances load balance traffic across web or app servers in their region. The diagram shows four backend servers, but you can deploy as many as needed. In this guide, there are two NGINX Open Source web servers in each region \(four total\); each one serves a customized static page identifying the server so you can track how load balancing is working.

Health checks are an integral feature of the configuration. Route 53 monitors the health of each NGINX Plus load balancer, marking it as down if a connection attempt times out or the HTTP response code is not `200 OK`. Similarly, the NGINX Plus load balancer monitors the health of the upstream web servers and propagates those errors to Route 53. If both web servers or both NGINX Plus instances in a region are down, Route 53 fails over to the other region.

### Prerequisites

The instructions assume you have the following:

* [An AWS account](https://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/AboutAWSAccounts.html).
* An NGINX Plus subscription, either [purchased](https://www.nginx.com/products/pricing/) or a [free 30-day trial](https://www.nginx.com/free-trial-request/).
* Familiarity with NGINX and NGINX Plus configuration syntax. Complete configuration snippets are provided, but not analyzed in detail.
* Eight EC2 instances, four in each of two regions. The [Appendix](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#appendix) provides instructions for creating instances with the expected names, and installing and configuring NGINX Plus and NGINX Open Source as appropriate.

### Configuring Global Server Load Balancing

With the [required AWS configuration](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#prereqs) in place, we’re ready to configure Route 53 for global server load balancing.

Complete step‑by‑step instructions are provided in the following sections:

* [Creating a Hosted Zone](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#hosted-zone)
* [Linking the Domain to EC2 Instances](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#link-instances)
* [Configuring Health Checks for Route 53 Failover](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#route-53-health-checks)
* [Configuring NGINX Plus Application Health Checks](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#nginx-plus-health-checks)

#### Creating a Hosted Zone

Create a _hosted zone_, which basically involves designating a domain name to be managed by Route 53. As covered in the instructions, you can either use \(transfer\) an existing domain name or purchase a new one from Amazon.

**Note**: When you transfer an existing domain, it can take up to 48 hours for the updated DNS record to propagate. The propagation time is usually much shorter for a new domain.

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/) \(**console.aws.amazon.com/**\).
2. Access the Route 53 dashboard page by clicking **Services** in the top AWS navigation bar, mousing over **Networking** in the All AWS Services column and then clicking **Route 53**.

   ![Screenshot showing how to access the Amazon Route 53 dashboard to configure global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-dashboard-open.png)

3. Depending on your history working with Route 53 instances, you might see the **Route 53 Dashboard**. Navigate to the **Registered domains** tab, shown here:

   ![Screenshot showing the Route 53 Registered domains tab during configuration of NGINX GSLB \(global server load balancing\)](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-registered-domains-tab.png)

   If you see the Route 53 home page instead, access the **Registered domains** tab by clicking the  Get started now  button under Domain registration.

   ![Screenshot showing the Amazon Route 53 homepage for a first-time Route 53 user during configuration of AWS GSLB \(global server load balancing\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-homepage.png)

4. On the **Registered domains** tab \(the first figure in the Step 3\), click either the  Register Domain  or  Transfer Domain  button as appropriate and follow the instructions.
5. When you register or transfer a domain, AWS by default creates a hosted zone for it. To verify that there is a hosted zone for your domain, navigate to the **Hosted Zones** tab on the Route 53 dashboard. This example shows the domain we registered for this guide, nginxroute53.com.

   ![Screenshot showing a newly registered hosted zone during configuration of Route 53 global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-hosted-zones-tab.png)

   \(If no domain is listed, click the  Create Hosted Zone  button. The Create Hosted Zone column opens on the right side of the tab. Type the domain name in the **Domain Name** field and click the  Create  button at the bottom of the column.\)

#### Linking the Domain to EC2 Instances

Now we link the domain to our EC2 instances so that content for the domain can be served from them. We do this by creating Route 53 _record sets_ for the domain. To implement GSLB, in the record sets we specify **Latency** as the _routing policy_. This means that in response to a client query for DNS information about a domain, Route 53 sends the DNS records for the region hosting that domain in which servers are responding most quickly to addresses in the client’s IP range.

You can also select **Geolocation** as the routing policy, but failover might not work as expected. With the **Geolocation** routing policy, only clients from a specific a continent or country can access the content on a server. If you specify the United States, you also have the option of specifying a state as the “sublocation”, limiting access to users from only that state. In this case, you can also create a generic US location to catch all requests that aren’t sent from a listed sublocation.

We recommend that you choose **Geolocation** as the routing policy only for particular use cases, for example when you want to customize content for users in different countries – written in the country’s official language with prices in its currency, say. If your goal is to deliver content as quickly as possible, **Latency** remains the best routing policy.

Create records sets for your domain:

1. If you’re not already on the **Hosted Zones** tab of the Route 53 dashboard, navigate to it.
2. Click on the domain name in the **Domain Name** row for your hosted zone.

   ![Screenshot showing how to access the Create Record Set interface for Route 53 hosted zone during configuration of AWS global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-hosted-zones-tab-click-domain-name.png)

   The tab changes to display the record sets for the domain.

3. Click the  Create Record Set  button. The Create Record Set column opens on the right side of the tab, as shown here:

   ![Screenshot showing Create Record Set interface in Route 53 during configuration of global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-create-record-set-empty.png)

4. Fill in the fields in the **Create Record Set** column:
   * **Name** – You can leave this field blank, but for this guide we are setting the name to www.nginxroute53.com.
   * **Type** – A – IPv4 address.
   * **Alias** – No.
   * **TTL \(Seconds\)** – 60.

     **Note**: Reducing TTL from the default of 300 in this way can decrease the time that it takes for Route 53 to fail over when both NGINX Plus load balancers in the region are down, but there is always a delay of about two minutes regardless of the TTL setting. This is a built‑in limitation of Route 53.

   * **Value** – [Elastic IP addresses](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#elastic-ip) of the NGINX Plus load balancers in the first region \[in this guide, US West \(Oregon\)\].
   * **Routing Policy** – Latency.
5. A new area opens when you select Latency. Fill in the fields as indicated \(see the figure below\):

   * **Region** – Region to which the load balancers belong \(in this guide, us-west-2\).
   * **Set ID** – Identifier for this group of load balancers \(in this guide, US West LBs\).
   * **Associate with Health Check** – No.

   When you complete all fields, the tab looks like this:

   ![Screenshot showing a completed Route 53 record set during configuration of global server load balancing \(GSLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-create-record-set-completed.png)

6. Click the  Create  button.
7. Repeat Steps 3 through 6 for the load balancers in the other region \[in this guide, US East \(N. Virginia\)\].

You can now test your website. Insert your domain name into a browser and see that your request is being load balanced between servers based on your location.

#### Configuring Health Checks for Route 53 Failover

To trigger failover between AWS regions, we next configure health checks in Route 53. Route 53 monitors the NGINX Plus load balancers and fails over to the next closest region if both NGINX Plus load balancers are timing out or returning HTTP status codes other than `200 OK`. \(In this guide, failover is to the other region, since there are only two.\)

**Note:** It can take up to **three minutes** for Route 53 to begin routing traffic to another region. Although the TTL you specify in the record set for a region can slightly affect the amount of time, failover is never instantaneous because of the processing Route 53 must perform.

![Diagram showing failover between AWS regions when Amazon Route 53 is configured for global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-topology-failover.png)

We create health checks both for each NGINX Plus load balancer individually and for the load balancers in each region as a pair. Then we update the record sets created in the previous section to refer to the health checks.

* [Configuring Route 53 Health Checks for Individual Load Balancers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#route-53-health-checks-individual)
* [Configuring Route 53 Health Checks for the Paired Load Balancers in Each Region](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#route-53-health-checks-pair)
* [Modifying Record Sets to Associate Them with the Newly Defined Health Checks](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#route-53-health-checks-record-sets)

**Configuring Route 53 Health Checks for Individual Load Balancers**

1. Navigate to the **Health checks** tab on the Route 53 dashboard.

   ![Screenshot of Amazon Route 53 welcome screen seen by first-time user of Route 53 during configuration of global server load balancing \(GSLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-health-checks-tab-welcome.png)

2. Click the  Create health check  button. In the Configure health check form that opens, specify the following values, then click the  Next  button.

   * **Name** – Identifier for an NGINX Plus load balancer instance, for example US West LB 1.
   * **What to monitor** – Endpoint.
   * **Specify endpoint by** – IP address.
   * **IP address** – The [elastic IP address](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#elastic-ip) of the NGINX Plus load balancer.
   * **Port** – The port advertised to clients for your domain or web service \(the default is 80\).

   ![Screenshot of Amazon Route 53 interface for configuring health checks, during configuration of AWS global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-configure-health-check.png)

3. On the Get notified when health check fails screen that opens, set the **Create alarm** radio button to **Yes** or **No** as appropriate, then click the  Create health check  button.

   ![Screenshot of Route 53 configuration screen for enabling notifications of failed health checks, during configuration of Route 53 global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-get-notified-health-check.png)

4. Repeat Steps 2 and 3 for your other NGINX Plus load balancers \(in this guide, US West LB 2, US East LB 1, and US East LB 2\).
5. Proceed to the next section to configure health checks for the load balancer pairs.

**Configuring Route 53 Health Checks for the Paired Load Balancers in Each Region**

1. Click the  Create health check  button.
2. In the Configure health check form that opens, specify the following values, then click the  Next  button.

   * **Name** – Identifier for the pair of NGINX Plus load balancers in the first region, for example US West LBs.
   * **What to monitor** – Status of other health checks .
   * **Health checks to monitor** – The health checks of the two US West load balancers \(add them one after the other by clicking in the box and choosing them from the drop‑down menu as shown\).
   * **Report healthy when** – at least 1 of 2 selected health checks are healthy \(the choices in this field are obscured in the screenshot by the drop‑down menu\).

   ![Screenshot of Amazon Route 53 interface for configuring a health check of combined other health checks, during configuration of global server load balancing \(GSLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-configure-health-check-status-others.png)

3. On the Get notified when health check fails screen that opens, set the **Create alarm** radio button as appropriate \(see Step 5 in the [previous section](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#route-53-health-checks-individual)\), then click the  Create health check  button.
4. Repeat Steps 1 through 3 for the paired load balancers in the other region \[in this guide, US East \(N. Virginia\)\].

When you have finished configuring all six health checks, the **Health checks** tab looks like this:

![Screenshot showing six successfully configured health checks in Amazon Route 53, during configuration of NGINX Plus for GSLB \(global server load balancing\)](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-health-checks-tab-completed.png)

**Modifying Record Sets to Associate Them with the Newly Defined Health Checks**

1. Navigate to the **Hosted Zones** tab.
2. Click on the domain name in the **Domain Name** row for your hosted zone.

   ![Screenshot showing how to access the Create Record Set interface for Route 53 hosted zone during configuration of AWS global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-hosted-zones-tab-click-domain-name.png)

   The tab changes to display the record sets for the domain.

3. In the list of record sets that opens, click the row for the record set belonging to your first region \[in this guide, US West \(Oregon\)\]. The Edit Record Set column opens on the right side of the tab.

   ![Screenshot of interface for editing Route 53 record sets during configuration of global server load balancing \(GSLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-route53-edit-record-set.png)

4. Change the **Associate with Health Check** radio button to Yes.
5. In the **Health Check to Associate** field, select the paired health check for your first region \(in this guide, US West LBs\).
6. Click the  Save Record Set  button.

#### Configuring NGINX Plus Application Health Checks

When you are using the NGINX Plus load balancer, we recommend that you to configure [application health checks](https://www.nginx.com/products/application-health-checks/) of your backend servers. You can configure NGINX Plus to check more than simply whether a server is responding or returning `5_xx_` – for example, checking whether the content returned by the server is correct. When a server fails a health check, NGINX Plus removes it from the load‑balancing rotation until it passes a configured number of consecutive health checks. If all backend servers are down, NGINX Plus sends a `5_xx_` error to Route 53, which triggers a failover to another region.

These instructions assume that you have configured NGINX Plus on two EC2 instances in each region, as instructed in [Configuring NGINX Plus on the Load Balancers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configure-load-balancers).

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Connect to the US West LB 1 instance. For instructions, see [Connecting to an Instance](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/amazon-ec2-instances-for-nginx/#connect-to-instance).
2. Change directory to **/etc/nginx/conf.d**.

   ```text
   $ cd /etc/nginx/conf.d
   ```

3. Edit the west-lb1.conf file and add the **@healthcheck** location to set up health checks.

   ```text
   upstream backend-servers {
       server <public DNS name of Backend 1>; # Backend 1
       server <public DNS name of Backend 2>; # Backend 2
       zone backend-servers 64k;
   }

   server {
       location / {
           proxy_pass http://backend-servers;
       }
    
       location @healthcheck {
           proxy_pass http://backend-servers;
           proxy_connect_timeout 1s;
           proxy_read_timeout 1s;
           health_check uri=/ interval=1s;
       }
   }
   ```

   Directive documentation: [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location), [`proxy_connect_timeout`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_connect_timeout), [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass), [`proxy_read_timeout`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_read_timeout), [`server` \(virtual\)](https://nginx.org/en/docs/http/ngx_http_core_module.html#server), [`server` \(upstream\)](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream), [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

4. Verify the validity of the NGINX configuration and load it.

   ```text
   $ nginx -t
   $ nginx -s reload
   ```

5. Repeat Steps 1 through 4 for the other three load balancers \(US West LB 2, US East LB 1, and US East LB2\).

   In Step 3, change the filename as appropriate \(west-lb2.conf, east-lb1.conf, and east-lb2.conf\). In the east-lb1.conf and east-lb2.conf files, the `server` directives specify the public DNS names of Backup 3 and Backup 4.

### Appendix

The instructions in this Appendix explain how to create EC2 instances with the names used in this guide, and then install and configure NGINX Open Source and NGINX Plus on them:

* [Creating EC2 Instances and Installing the NGINX Software](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#create-instance-install-nginx)
* [Configuring Elastic IP Addresses](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#elastic-ip)
* [Configuring NGINX Open Source on the Backend Servers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configure-backend-servers)
* [Configuring NGINX Plus on the Load Balancers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configure-load-balancers)

#### Creating EC2 Instances and Installing the NGINX Software

The deployment in this guide uses eight EC2 instances, four in each of two AWS regions. In each region, two instances run NGINX Plus to load balance traffic to the backend \(NGINX Open Source\) web servers running on the other two instances.

Step‑by‑step instructions for creating EC2 instances and installing NGINX software are provided in our deployment guide, [Creating and Configuring Amazon EC2 Instances for NGINX Open Source and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/nginx/deployment-guides/amazon-web-services/amazon-ec2-instances-for-nginx.md).

**Note:** When installing NGINX Open Source or NGINX Plus, you connect to each instance over SSH. To save time, leave the SSH connection to each instance open after installing the software, for reuse when you configure it with the instructions in the sections below.

Assign the following names to the instances, and then install the indicated NGINX software.

* In the first region, which is US West \(Oregon\) in this guide:
  * Two load balancer instances running NGINX Plus:
    * US West LB 1
    * US West LB 2
  * Two backend instances running NGINX Open Source:
    * Backend 1
    * Backend 2
* In the second region, which is US East \(N. Virginia\) in this guide:
  * Two load balancer instances running NGINX Plus:
    * US East LB 1
    * US East LB 2
  * Two backend instances running NGINX Open Source:
    * Backend 3
    * Backend 4

Here’s the **Instances** tab after we create the four instances in the N. Virginia region.

![Screenshot showing newly created EC2 instances in one of two regions, which is a prerequisite to configuring AWS GSLB \(global server load balancing\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-ec2-useast-instances.png)

#### Configuring Elastic IP Addresses

For some EC2 instance types \(for example, on‑demand instances\), AWS by default assigns a different IP address to an instance each time you shut it down and spin it back up. A load balancer must know the IP addresses of the servers to which it is forwarding traffic, so the default AWS behavior requires you either to set up a service‑discovery mechanism or to modify the NGINX Plus configuration every time you shut down or restart a backend instance. \(A similar requirement applies to Route 53 when we shut down or restart an NGINX Plus instance.\) To get around this inconvenience, assign an _elastic IP address_ to each of the eight instances.

**Note:** AWS does not charge for elastic IP addresses as long as the associated instance is running. But when you shut down an instance, AWS charges a small amount to maintain the association to an elastic IP address. For details, see the Amazon EC2 Pricing page for your pricing model \(for example, the **Elastic IP Addresses** section of the [On-Demand Pricing](https://aws.amazon.com/ec2/pricing/on-demand/) page\).

Perform these steps on all eight instances.

1. Navigate to the **Elastic IPs** tab on the EC2 Dashboard.

   ![Screenshot of the Elastic IPs tab used during configuration of a new AWS EC2 instance, which is a prerequisite to configuring NGINX GSLB \(global server load balancing\)](https://cdn-1.wp.nginx.com/wp-content/uploads/2018/03/aws-ec2-elastic-ips-tab.png)

2. Click the  Allocate New Address  button. In the window that pops up, click the  Yes, Allocate  button and then the  Close  button.
3. Associate the elastic IP address with an EC2 instance:

   * Right‑click in the IP address’ row and select  Associate Address  from the drop‑down menu that appears.
   * In the window that pops up, click in the **Instance** field and select an instance from the drop‑down menu.

   Confirm your selection by clicking the  Associate  button.

   ![Screenshot of the interface for associating an AWS EC2 instance with an elastic IP address, which is a prerequisite to configuring AWS global load balancing \(GLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2016/10/aws-ec2-associate-address-popup.png)

After you complete the instructions on all instances, the list for a region \(here, Oregon\) looks like this:

![Screenshot showing the elastic IP addresses assigned to four AWS EC2 instances during configuration of global server load balancing \(GSLB\) with NGINX Plus](https://cdn-1.wp.nginx.com/wp-content/uploads/2018/03/aws-ec2-elastic-ip-address-list.png)

#### Configuring NGINX Open Source on the Backend Servers

Perform these steps on all four backend servers: Backend 1, Backend 2, Backend 3, and Backend 4. In Step 3, substitute the appropriate name for `Backend X` in the **index.html** file.

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Connect over SSH to the instance \(or return to the terminal you left open after installing NGINX Open Source\) and change directory to and change directory to your root directory. For the instance in this guide, it is **/home/ubuntu**.

   ```text
   $ cd /home/ubuntu
   ```

2. Create a directory called **public\_html** and change directory to it.

   ```text
   $ mkdir public_html
   $ cd public_html
   ```

3. Using your preferred text editor, create a new file called **index.html** and add this text to it:

   ```text
   This is Backend X
   ```

4. Change directory to **/etc/nginx/conf.d**.

   ```text
   $ cd /etc/nginx/conf.d
   ```

5. Rename **default.conf** to **default.conf.bak** so that NGINX Plus does not load it.

   ```text
   $ mv default.conf default.conf.bak
   ```

6. Create a new file called **backend.conf** and add this text, which defines the docroot for this web server:

   ```text
   server {
       root /home/ubuntu/public_html;
   }
   ```

   Directive documentation: [`root`](https://nginx.org/en/docs/http/ngx_http_core_module.html#root), [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server)

7. Verify the validity of the NGINX configuration and load it.

   ```text
   $ nginx -t
   $ nginx -s reload
   ```

#### Configuring NGINX Plus on the Load Balancers

Perform these steps on all four backend servers: US West LB 1, US West LB 2, US East LB 1, and US West LB 2.

**Note:** Some commands require `root` privilege. If appropriate for your environment, prefix commands with the `sudo` command.

1. Connect over SSH to the instance \(or return to the terminal you left open after installing NGINX Plus\) and change directory to **/etc/nginx/conf.d**.

   ```text
   $ cd /etc/nginx/conf.d
   ```

2. Rename **default.conf** to **default.conf.bak** so that NGINX Plus does not load it.

   ```text
   $ mv default.conf default.conf.bak
   ```

3. Create a new file containing the following text, which configures load balancing of the two backend servers in the relevant region. The filename on each instance is:

   * For US West LB 1 – west-lb1.conf
   * For US West LB 2 – west-lb2.conf
   * For US East LB 1 – east-lb1.conf
   * For US West LB 2 – east-lb2.conf

   In the `server` directives in the `upstream` block, substitute the public DNS names of the backend instances in the region; to learn them, see the **Instances** tab in the EC2 Dashboard.

   ```text
   upstream backend-servers {
       server <public DNS name of Backend 1>; # Backend 1
       server <public DNS name of Backend 2>; # Backend 2
   }
   server {
       location / {
           proxy_pass http://backend-servers;
       }
   }
   ```

   Directive documentation: [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location), [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass), [`server` \(virtual\)](https://nginx.org/en/docs/http/ngx_http_core_module.html#server), [`server` \(upstream\)](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream)

4. Verify the validity of the NGINX configuration and load it.

   ```text
   $ nginx -t
   $ nginx -s reload
   ```

5. To test that the configuration is working correctly, for each load balancer enter its public DNS name in the address field of your web browser. As you access the load balancer repeatedly, the content of the page alternates between “This is Backend 1” and “This is Backend 2” in your first region, and “This is Backend 3” and “This is Backend 4” in the second region.

Now that all eight EC2 instances are configured and local load balancing is working correctly, we can set up global server load balancing with Route 53 to route traffic based on the IP address of the requesting client.

Return to main instructions, [Configuring Global Server Load Balancing](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#gslb)

#### Revision History

* Version 3 \(April 2018\) – Reorganization of Appendix
* Version 2 \(January 2017\) – Clarified information about root permissions; miscellaneous fixes \(NGINX Plus Release 11\)
* Version 1 \(October 2016\) – Initial version \(NGINX Plus Release 10\)

