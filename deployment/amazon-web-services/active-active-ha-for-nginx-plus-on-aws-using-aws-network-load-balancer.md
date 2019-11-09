# Active-Active HA for NGINX Plus on AWS Using AWS Network Load Balancer



This guide explains how to create our recommended solution for a highly available, active‑active deployment of NGINX Plus in the Amazon Web Services \(AWS\) cloud. The solution combines the AWS Network Load Balancer \(NLB\) for fast and efficient handling of Layer 4 traffic with NGINX Plus for advanced, Layer 7 features such as load balancing, caching, and content‑based routing. The combined solution is fast, powerful, reliable, and likely to be relatively low‑cost.

This guide explains how to set up an AWS NLB in front of one pair of NGINX Plus load balancers. \(You can increase resiliency as needed by following the same steps for additional NGINX Plus instances.\)

* [About AWS NLB](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#about-nlb)
* [About NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#about-nginx)
* [Solution Overview](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#overview)
* [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#prereqs)
* [Configuring an AWS Network Load Balancer](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-configure)
  * [Allocating an Elastic IP Address](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-eip)
  * [Creating an AWS NLB](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-create)
  * [Configuring the AWS NLB Routing Options](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-routing-options)
  * [Registering Instances in the Target Group](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-register-instances)
  * [Launching the AWS NLB](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-launch)

The [Appendix](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#appendix) provides instructions for creating EC2 instances with the names used in this guide, and installing and configuring the NGINX software on them.

### About AWS NLB

AWS NLB is optimized for fast, efficient load balancing at the connection level \(Layer 4\). AWS NLB uses a [flow hash routing algorithm](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/how-elastic-load-balancing-works.html#routing-algorithm).

AWS NLB is ideal for fast load balancing of TCP traffic, as it’s able to handle millions of requests per second while maintaining ultra‑low latencies. This enables AWS NLB to more easily handle volatile traffic patterns – patterns with sudden and dramatic changes in the amount of traffic.

Unlike previous AWS solutions, AWS NLB supports both static IP addresses and [Elastic IP addresses](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html).

### About NGINX Plus

NGINX Plus is complementary to NLB. Operating at Layer 7 \(the application layer\), it uses more advanced load‑balancing criteria, including schemes that rely on the content of requests and the results of NGINX Plus’ [active health checks](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/nginx/deployment-guides/admin-guide/load-balancer/http-health-check.md).

[NGINX Plus](https://www.nginx.com/products/nginx) is the commercially supported version of the [NGINX Open Source](https://nginx.org/en) software. NGINX Plus is a complete application delivery platform, extending the power of NGINX with a host of enterprise‑ready capabilities that enhance an AWS web application deployment and are instrumental to building web applications at scale.

NGINX Plus provides both reverse‑proxy features and load‑balancing features, including:

* [Full‑featured HTTP, TCP, and UDP load balancing](https://www.nginx.com/products/nginx/load-balancing/)
* [Intelligent session persistence](https://www.nginx.com/products/nginx/load-balancing/#session-persistence)
* [High‑performance reverse proxy](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/nginx/deployment-guides/admin-guide/web-server/reverse-proxy.md)
* [Caching and offload of dynamic and static content](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/nginx/deployment-guides/admin-guide/content-cache/content-caching.md)
* [Adaptive streaming to deliver audio and video to any device](https://www.nginx.com/products/nginx/streaming-media/)
* [Application-aware health checks](https://www.nginx.com/products/nginx/load-balancing/#health-checks) and [high availability](https://www.nginx.com/products/nginx/high-availability/)
* [Advanced activity monitoring available via a dashboard or API](https://www.nginx.com/products/nginx/live-activity-monitoring/)
* [Management and real‑time configuration changes with DevOps‑friendly tools](https://www.nginx.com/products/nginx/load-balancing/#load-balancing-api)

### Solution Overview

The setup in this guide combines AWS NLB, AWS target groups, Amazon Elastic Compute Cloud \(EC2\) instances running NGINX Plus, and EC2 instances running NGINX Open Source, which together provide a highly available, all‑active NGINX and NGINX Plus solution.

![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-topology.png)

AWS NLB handles Layer 4 TCP connections and balances traffic using a flow hash routing algorithm. By default, an AWS NLB has a DNS name to which an IP address is assigned dynamically, but you can optionally attach an Elastic IP address to the AWS NLB to ensure that it will always be reachable at the same IP address.

The AWS NLB listens for incoming connections as defined by its listeners. Each listener forwards a new connection to one of the available instances in a target group, chosen using the flow hash routing algorithm.

In this guide, the target group consists of two NGINX Plus load balancer instances. However, you can register an unlimited number of instances in the target group, or use an [AWS Auto Scaling group](https://aws.amazon.com/autoscaling/) to dynamically adjust the number of NGINX Plus instances.

### Prerequisites

These instructions assume you have the following:

* [An AWS account](https://docs.aws.amazon.com/AmazonSimpleDB/latest/DeveloperGuide/AboutAWSAccounts.html).
* Six EC2 instances, four running NGINX Open Source and two running NGINX Plus. You need a subscription for each NGINX Plus instance, either paid or a [30‑day free trial](https://www.nginx.com/free-trial-request).

  The [Appendix](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#appendix) provides instructions for creating instances with the expected names, and installing and configuring the NGINX software.

* Familiarity with NGINX and NGINX Plus configuration syntax. Complete configuration snippets are provided, but not analyzed in detail.

### Configuring an AWS Network Load Balancer

With NGINX Open Source and NGINX Plus installed and configured on the EC2 instances \(see the [Appendix](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#appendix)\), we’re ready to configure an AWS NLB for a highly available, all‑active NGINX Plus setup.

* [Allocating an Elastic IP Address](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-eip)
* [Creating an AWS NLB](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-create)
* [Configuring the AWS NLB Routing Options](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-routing-options)
* [Registering Instances in the Target Group](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-register-instances)
* [Launching the AWS NLB](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-launch)

#### Allocating an Elastic IP Address

The first step is to allocate an Elastic IP address, which becomes the fixed IP address for your AWS NLB. \(While using an Elastic IP address is optional, we strongly recommend that you do so. With a dynamic IP address, the AWS NLB might not remain reachable if you reconfigure or restart it.\)

1. Log in to the [AWS Management Console](https://console.aws.amazon.com/ec2/) for EC2 \(**https://console.aws.amazon.com/ec2/**\).
2. In the left navigation bar, select **Elastic IPs**, then click either of the  Allocate new address  buttons.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-open.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-open.png)

3. In the **Allocate new address** window that opens, click the  Allocate  button.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-allocate-new.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-allocate-new.png)

4. When the message appears indicating that the request for an Elastic IP address succeeded, click the  Close  button.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-new-success.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-new-success.png)

The new Elastic IP address appears on the **Elastic IPs** dashboard.

[![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-new-display.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-eip-new-display.png)

#### Creating the AWS NLB

1. In the left navigation bar, select **Load Balancers**, then click the  Create Load Balancer  button.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-open.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-open.png)

2. In the **Select load balancer type** window that opens, click the  Create  button in the  Network Load Balancer  panel \(the center one\).

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-types.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-types.png)

3. In the **Step 1: Configure Load Balancer** window that opens, enter the following values:

   * In the **Basic Configuration** section:
     * **Name** – Name of your AWS NLB \(aws-nlb-lb in this guide\).
     * **Scheme** – internet-facing.
   * In the **Listeners** section:
     * **Load Balancer Protocol** – TCP \(the only available option\).
     * **Load Balancer Port** – Port on which your AWS NLB listens for incoming connections. In this guide, and for most web applications, it is port 80.
   * In the **Availability Zones** section, the zones that host the EC2 instances to which your AWS NLB routes traffic. Click the radio button in the leftmost column of the appropriate row:
     * If you set up your instances with the instructions in [Creating Amazon EC2 Instances to Run NGINX Open Source and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/nginx/deployment-guides/amazon-web-services/amazon-ec2-instances-for-nginx.md), select the default subnet within the default [Amazon Virtual Private Cloud](https://aws.amazon.com/vpc/) \(VPC\) to target a single availability zone.
     * If you set up your instances using our scripts for [Packer and Terraform](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#create-instances-automated), use the aws-nlb-subnet within the aws-nlb-vpc VPC to target a single availability zone.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-configure.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-configure.png)

4. When you select an availability zone in the table, a drop‑down menu appears in the **Elastic IP** column. Select the address you allocated in [Allocating an Elastic IP Address](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-eip).
5. Click the  Next: Configure Routing  button in the lower‑right corner of the window.

#### Configuring the AWS NLB Routing Options

In the **Step 2: Configure Routing** window that opens, you create a _target group_, which contains the set of EC2 instances across which your AWS NLB load balances traffic \(you’ll specify those instances in the [next section](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-register-instances)\).

1. In the **Target group** section, select or enter the following values:
   * **Target group** – New target group
   * **Name** – Name of the target group \(for this guide, aws-nlb-tg\)
   * **Protocol** – TCP \(the only available option\)
   * **Port** – The port you specified for the **Load Balancer Port** field in Step 3 of the [previous section](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-create) \(80 in this guide\)
   * **Target type** – instance
2. In the **Health checks** section, open the **Advanced health check settings** subsection and enter the following values:

   * **Protocol** – Protocol the AWS NLB uses when sending health checks. This guide uses TCP, which means the AWS NLB makes a health check by attempting to open a TCP connection on the port specified in the next field.
   * **Port** – Port on the target instances to which the AWS NLB sends health checks. In this guide, we’re selecting traffic port to send health checks to the same port as regular traffic.
   * **Healthy threshold** – Number of consecutive health checks an unhealthy instance must pass to be considered healthy.
   * **Unhealthy threshold** – Number of consecutive health checks a healthy instance must fail to be considered unhealthy.
   * **Timeout** – Number of seconds the AWS NLB waits for a response to the health check before considering the instance unhealthy.
   * **Interval** – Number of seconds between health checks.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-routing.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-routing.png)

   If you want to use HTTP‑based health checks, select HTTP or HTTPS in the **Protocol** field instead of TCP. Two additional fields open \(not shown in the screenshot\):

   * **Path** – The path to which the AWS NLB sends a `GET` request as the health check.
   * **Success codes** – Range of HTTP response codes the AWS NLB accepts as indicating a successful health check.

3. Click the  Next: Register Targets  button in the lower‑right corner of the window.

#### Registering Instances in the Target Group

In the **Step 3: Register Targets** window that opens, you add instances to the empty target group you created in the previous section. For this guide, we add both of our NGINX Plus load balancer instances.

1. In the **Instances** table, click the radio button in the left‑most column for the two NGINX Plus load balancer instances, ngx-plus-1 and nginx-plus-2.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-register-targets.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-register-targets.png)

2. Click the  Add to registered  button above the table. The instances are added to the **Registered targets** table.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-targets-display.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-targets-display.png)

3. Click the  Next: Review  button in the lower‑right corner of the window.

#### Launching the AWS NLB

In the **Step 4: Review** window that opens:

1. Verify that the settings are correct. If so, click the  Create  button in the lower‑right corner of the window. To change settings, click the  Previous  button to go back to previous screens.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-review.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-review.png)

2. The AWS NLB is provisioned. When the success message appears, click the  Close  button to return to the **Load Balancers** dashboard.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-success.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-load-balancer-success.png)

3. The **Load Balancers** dashboard opens. As noted in the previous **Load Balancer Creation Status** window, it can take a few minutes to provision the AWS NLB. When the value in the **State** column of the table changes to active, click the radio button in the left‑most column to display details about the AWS NLB.

   [![](https://www.nginx.com/wp-content/uploads/2018/03/aws-nlb-load-balancer-active-state.png)](https://www.nginx.com/wp-content/uploads/2018/03/aws-nlb-load-balancer-active-state.png)

4. To verify that the AWS NLB is working correctly, open a new browser window and navigate to the AWS NLB’s public DNS name, which appears in the **DNS name** field in the **Basic Configuration** section of the dashboard. \[If you copy and paste the DNS name, be sure not to include the parenthesized words at the end, \(A Record\).\]

   The default **Welcome to nginx!** page indicates that the AWS NLB has successfully forwarded a request to one of the two NGINX Plus instances.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-welcome-to-nginx.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-welcome-to-nginx.png)

5. To verify that the NGINX Plus load balancer is working correctly, add /backend-one and then /backend-two to the public DNS name. The pages indicate that you have reached NGINX instances serving the two backend applications, App 1 and App 2.

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-app1.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-app1.png)

   [![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-app2-v2.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-app2-v2.png)

### Appendix

This Appendix provides links to instructions for creating EC2 instances with the names used in this guide, and then installing and configuring NGINX Open Source and NGINX Plus on them:

* [Creating EC2 Instances and Installing the NGINX Software](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#create-instance-install-nginx)
* [Configuring NGINX Open Source on the Web Servers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#configure-web-servers)
* [Configuring NGINX Plus on the Load Balancers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#configure-load-balancers)

As an alternative to creating and configuring instances individually, you can use our Packer and Terraform scripts to completely automate the process:

* [Automating Instance Setup with Packer and Terraform](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#create-instances-automated)

After completing the instructions, you have completed the prerequisites for this guide and can continue to [Configuring an AWS Network Load Balancer](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-configure).

#### Creating EC2 Instances and Installing the NGINX Software

The deployment in this guide uses six EC2 instances: two instances running NGINX Plus that load balance traffic to four instances running NGINX Open Source as a web server. The four NGINX Open Source instances are deployed in two pairs, each pair running a different app.

Step‑by‑step instructions for creating EC2 instances and installing NGINX Open Source and NGINX Plus are provided in our deployment guide, [Creating and Configuring Amazon EC2 Instances for NGINX Open Source and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/nginx/deployment-guides/amazon-web-services/amazon-ec2-instances-for-nginx.md).

**Note:** When installing NGINX Open Source or NGINX Plus, you connect to each instance over SSH. To save time, leave the SSH connection to each instance open after installing the software, for reuse when you configure it with the instructions in the sections below.

Assign the following names to the instances, and then install the indicated NGINX software. The screenshot below shows the resulting **Instances** table.

* Four NGINX Open Source instances:
  * App 1:
    * ngx-oss-app1-1
    * ngx-oss-app1-2
  * App 2:
    * ngx-oss-app2-1
    * ngx-oss-app2-2
* Two NGINX Plus instances:
  * ngx-plus-1
  * ngx-plus-2

[![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-instances-summary.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-instances-summary.png)

#### Configuring NGINX Open Source on the Web Servers

For the purposes of this guide, you configure the NGINX Open Source instances as web servers that return a page specifying the server name, address, and other information. As an example, here’s the page returned by App 1:

[![](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-app1.png)](https://www.nginx.com/wp-content/uploads/2017/11/aws-nlb-app1.png)

Step‑by‑step instructions are provided in our deployment guide, [Setting Up an NGINX Demo Environment](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/setting-up-nginx-demo-environment#nginx-oss).

Repeat the instructions on all four web servers:

* Running App 1:
  * ngx-oss-app1-1
  * ngx-oss-app1-2
* Running App 2:
  * ngx-oss-app2-1
  * ngx-oss-app2-2

#### Configuring NGINX Plus on the Load Balancers

For the purposes of this guide, you configure the NGINX Plus instances as load balancers to distribute requests to the NGINX Open Source web servers configured in the [next section](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#configure-web-servers).

Step‑by‑step instructions are provided in our deployment guide, [Setting Up an NGINX Demo Environment](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/setting-up-nginx-demo-environment#nginx-plus).

Repeat the instructions on both ngx-plus-1 and ngx-plus-2.

### Automating Instance Setup with Packer and Terraform

As an alternative to individual creation and configuration of the six instances used in this guide, you can use the Packer and Terraform scripts from our [GitHub repository](https://github.com/nginxinc/NGINX-Demos/tree/master/aws-nlb). They generate the complete setup for this guide, with two load balancer instances running NGINX Plus and four web server instances running NGINX Open Source.

After executing the scripts, you can jump directly into the [instructions for creating an AWS NLB](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/#nlb-configure) without any further setup. Additionally, the scripts create a new set of networking rules and security group settings to avoid conflicts with any pre‑existing network settings.

**Note:** Instead of using the default VPC like the [instructions in our Deployment Guide](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/high-availability-network-load-balancer/nginx/deployment-guides/amazon-web-services/amazon-ec2-instances-for-nginx.md), the scripts create a new VPC.

To run the scripts, follow these instructions:

1. Install [Packer](https://www.packer.io/intro/getting-started/install.html) and [Terraform](https://www.terraform.io/intro/getting-started/install.html).
2. Clone or download the scripts from our [GitHub repository](https://github.com/nginxinc/NGINX-Demos/tree/master/aws-nlb):
   * The scripts in packer/ngx-oss are for creating an Ubuntu AMI running open source NGINX.
   * The scripts in packer/ngx-plus are for creating an AWS Linux AMI running NGINX Plus.
   * The scripts in **terraform** are for launching and configuring the two NGINX Plus load balancer instances and the four NGINX Open Source web server instances.
3. Set your AWS credentials in the Packer and Terraform scripts:
   * For Packer, set your credentials in the `variables` block in both packer/ngx-oss/packer.json and packer/ngx-plus/packer.json:

     ```text
     "variables": {
         "home": "{{env `HOME`}}",
         "aws_access_key": "",
         "aws_secret_key": ""
     }
     ```

   * For Terraform, set your credentials in **terraform/provider.tf**:

     ```text
     provider "aws" {
       region = "us-west-1"
       access_key = ""
       secret_key = ""
     }
     ```
4. Copy your NGINX Plus certificate and key to **~/.ssh/certs**.
5. Run the `setup.sh` script:

   ```text
   $ chmod +x setup.sh
   $ ./setup.sh
   ```

The script launches two NGINX Plus load balancer instances and four NGINX web server instances and configures the appropriate settings on each instance to run the guide.

If you want to delete the infrastructure created by Terraform, run the `cleanup.sh` script.

```text
$ chmod +x cleanup.sh
$ ./cleanup.sh
```

#### Revision History

* Version 3 \(April 2019\) – Modularization of Appendix
* Version 2 \(April 2018\) – Revisions to Appendix
* Version 1 \(November 2017\) – Initial version \(NGINX Plus Release 13\)

