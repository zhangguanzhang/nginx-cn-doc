# Global Server Load Balancing

##  <a id="page_title"></a>

* [Amazon Route 53](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/)
  * [About NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#about-nginx-plus)
  * [Topology for Global Load Balancing with Amazon Route 53 and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#topology-for-global-load-balancing-with-amazon-route-53-and-nginx-plus)
  * [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#prerequisites)
  * [Configuring Global Server Load Balancing](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-global-server-load-balancing)
    * [Creating a Hosted Zone](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#creating-a-hosted-zone)
    * [Linking the Domain to EC2 Instances](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#linking-the-domain-to-ec2-instances)
    * [Configuring Health Checks for Route 53 Failover](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-health-checks-for-route-53-failover)
      * [Configuring Route 53 Health Checks for Individual Load Balancers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-route-53-health-checks-for-individual-load-balancers)
      * [Configuring Route 53 Health Checks for the Paired Load Balancers in Each Region](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-route-53-health-checks-for-the-paired-load-balancers-in-each-region)
      * [Modifying Record Sets to Associate Them with the Newly Defined Health Checks](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#modifying-record-sets-to-associate-them-with-the-newly-defined-health-checks)
    * [Configuring NGINX Plus Application Health Checks](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-nginx-plus-application-health-checks)
  * [Appendix](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#appendix)
    * [Creating EC2 Instances and Installing the NGINX Software](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#creating-ec2-instances-and-installing-the-nginx-software)
    * [Configuring Elastic IP Addresses](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-elastic-ip-addresses)
    * [Configuring NGINX Open Source on the Backend Servers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-nginx-open-source-on-the-backend-servers)
    * [Configuring NGINX Plus on the Load Balancers](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#configuring-nginx-plus-on-the-load-balancers)
    * [Revision History](https://docs.nginx.com/nginx/deployment-guides/amazon-web-services/route-53-global-server-load-balancing/#revision-history)
* [NS1](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/)
  * [About NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#about-nginx-plus)
  * [Prerequisites](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#prerequisites)
  * [Setting Up NS1](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#setting-up-ns1)
  * [Installing the NS1 Agent](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#installing-the-ns1-agent)
  * [Verifying that NS1 Redistributes Traffic](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#verifying-that-ns1-redistributes-traffic)
    * [Verifying Traffic Redistribution when an NGINX Plus Instance Is Marked Down](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#verifying-traffic-redistribution-when-an-nginx%C2%A0plus-instance-is-marked-down)
    * [Verifying Traffic Redistribution When an Upstream Group Is Down](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#verifying-traffic-redistribution-when-an-upstream-group-is-down)
    * [Verifying Traffic Redistribution When a Threshold Is Exceeded](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#verifying-traffic-redistribution-when-a-threshold-is-exceeded)
      * [Creating the Shed Filter](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#creating-the-shed-filter)
      * [Testing the Threshold](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#testing-the-threshold)
    * [Revision History](https://docs.nginx.com/nginx/deployment-guides/global-server-load-balancing/ns1-global-server-load-balancing/#revision-history)

