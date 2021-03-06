# Table of contents

* [nginx docs](README.md)

## Admin Guide

* [Installing NGINX and NGINX Plus](admin-guide/installing-nginx/README.md)
  * [Installing NGINX Plus](admin-guide/installing-nginx/installing-nginx-plus.md)
  * [Installing NGINX Open Source](admin-guide/installing-nginx/installing-nginx-open-source.md)
  * [Installing NGINX Plus AMIs on Amazon EC2](admin-guide/installing-nginx/installing-nginx-plus-amis-on-amazon-ec2.md)
  * [Installing NGINX Plus on the Google Cloud Platform](admin-guide/installing-nginx/installing-nginx-plus-on-the-google-cloud-platform.md)
  * [Installing NGINX Plus on Microsoft Azure](admin-guide/installing-nginx/installing-nginx-plus-on-microsoft-azure.md)
  * [Deploying NGINX and NGINX Plus on Docker](admin-guide/installing-nginx/deploying-nginx-and-nginx-plus-on-docker.md)
* Basic Functionality
  * [Controlling NGINX Processes at Runtime](admin-guide/basic-functionality/runtime-control.md)
  * [Creating NGINX Plus and NGINX Configuration Files](admin-guide/basic-functionality/managing-configuration-files.md)
* [Load Balancer](admin-guide/load-balancer/README.md)
  * [HTTP Load Balancing](admin-guide/load-balancer/http-load-balancing.md)
  * [TCP and UDP Load Balancing](admin-guide/load-balancer/tcp-and-udp-load-balancing.md)
  * [HTTP Health Checks](admin-guide/load-balancer/http-health-checks.md)
  * [TCP Health Checks](admin-guide/load-balancer/tcp-health-checks.md)
  * [UDP Health Checks](admin-guide/load-balancer/udp-health-checks.md)
  * [Configuring Dynamic Load Balancing with the NGINX Plus API](admin-guide/load-balancer/configuring-dynamic-load-balancing-with-the-nginx-plus-api.md)
  * [Accepting the PROXY Protocol](admin-guide/load-balancer/accepting-the-proxy-protocol.md)
* [Content Cache](admin-guide/content-cache/README.md)
  * [NGINX Content Caching](admin-guide/content-cache/nginx-content-caching.md)
* [Web Server](admin-guide/web-server/README.md)
  * [Configuring NGINX Plus as a Web Server](admin-guide/web-server/configuring-nginx-plus-as-a-web-server.md)
  * [Serving Static Content](admin-guide/web-server/serving-static-content.md)
  * [NGINX Reverse Proxy](admin-guide/web-server/nginx-reverse-proxy.md)
  * [Compression and Decompression](admin-guide/web-server/compression-and-decompression.md)
  * [Using NGINX and NGINX Plus as an Application Gateway with uWSGI and Django](admin-guide/web-server/using-nginx-and-nginx-plus-as-an-application-gateway-with-uwsgi-and-django.md)
* [Security Controls](admin-guide/security-controls/README.md)
  * [NGINX SSL Termination](admin-guide/security-controls/nginx-ssl-termination.md)
  * [SSL Termination for TCP Upstream Servers](admin-guide/security-controls/ssl-termination-for-tcp-upstream-servers.md)
  * [Restricting Access with HTTP Basic Authentication](admin-guide/security-controls/restricting-access-with-http-basic-authentication.md)
  * [Authentication Based on Subrequest Result](admin-guide/security-controls/authentication-based-on-subrequest-result.md)
  * [Setting up JWT Authentication](admin-guide/security-controls/setting-up-jwt-authentication.md)
  * [Limiting Access to Proxied HTTP Resources](admin-guide/security-controls/limiting-access-to-proxied-http-resources.md)
  * [Restricting Access to Proxied TCP Resources](admin-guide/security-controls/restricting-access-to-proxied-tcp-resources.md)
  * [Restricting Access by Geographical Location](admin-guide/security-controls/restricting-access-by-geographical-location.md)
  * [Securing HTTP Traffic to Upstream Servers](admin-guide/security-controls/securing-http-traffic-to-upstream-servers.md)
  * [Securing TCP Traffic to Upstream Servers](admin-guide/security-controls/securing-tcp-traffic-to-upstream-servers.md)
  * [Dynamic Blacklisting of IP Addresses](admin-guide/security-controls/dynamic-blacklisting-of-ip-addresses.md)
* [Monitoring](admin-guide/monitoring/README.md)
  * [Live Activity Monitoring](admin-guide/monitoring/live-activity-monitoring.md)
  * [Configuring Logging](admin-guide/monitoring/configuring-logging.md)
  * [Debugging NGINX](admin-guide/monitoring/debugging-nginx.md)
  * [Monitoring NGINX and NGINX Plus with the New Relic Plug-In](admin-guide/monitoring/monitoring-nginx-and-nginx-plus-with-the-new-relic-plug-in.md)
* [High Availability](admin-guide/high-availability/README.md)
  * [High Availability Support for NGINX Plus in On-Premises Deployments](admin-guide/high-availability/high-availability-support-for-nginx-plus-in-on-premises-deployments.md)
  * [Configuring Active-Active High Availability and Additional Passive Nodes with keepalived](admin-guide/high-availability/configuring-active-active-high-availability-and-additional-passive-nodes-with-keepalived.md)
  * [Synchronizing NGINX Configuration in a Cluster](admin-guide/high-availability/synchronizing-nginx-configuration-in-a-cluster.md)
  * [Runtime State Sharing in a Cluster](admin-guide/high-availability/runtime-state-sharing-in-a-cluster.md)
  * [How NGINX Plus Performs Zone Synchronization](admin-guide/high-availability/how-nginx-plus-performs-zone-synchronization.md)
* [Dynamic Modules](admin-guide/dynamic-modules/README.md)
  * [Dynamic Modules](admin-guide/dynamic-modules/dynamic-modules.md)
  * [Brotli](admin-guide/dynamic-modules/brotli.md)
  * [Cookie-Flag](admin-guide/dynamic-modules/cookie-flag.md)
  * [Encrypted-Session](admin-guide/dynamic-modules/encrypted-session.md)
  * [GeoIP](admin-guide/dynamic-modules/geoip.md)
  * [GeoIP2](admin-guide/dynamic-modules/geoip2.md)
  * [Headers-More](admin-guide/dynamic-modules/headers-more.md)
  * [HTTP Substitutions Filter](admin-guide/dynamic-modules/http-substitutions-filter.md)
  * [Image-Filter](admin-guide/dynamic-modules/image-filter.md)
  * [Lua](admin-guide/dynamic-modules/lua.md)
  * [njs Scripting Language](admin-guide/dynamic-modules/njs-scripting-language.md)
  * [NGINX WAF](admin-guide/dynamic-modules/nginx-waf.md)
  * [OpenTracing](admin-guide/dynamic-modules/opentracing.md)
  * [Phusion Passenger Open Source](admin-guide/dynamic-modules/phusion-passenger-open-source.md)
  * [Perl](admin-guide/dynamic-modules/perl.md)
  * [Prometheus-njs](admin-guide/dynamic-modules/prometheus-njs.md)
  * [RTMP](admin-guide/dynamic-modules/rtmp.md)
  * [Set-Misc](admin-guide/dynamic-modules/set-misc.md)
  * [XSLT](admin-guide/dynamic-modules/xslt.md)
* [Mail Proxy](admin-guide/mail-proxy/README.md)
  * [Configuring NGINX as a Mail Proxy Server](admin-guide/mail-proxy/configuring-nginx-as-a-mail-proxy-server.md)

## Deployment Guides <a id="deployment"></a>

* [Deployment Guides](deployment/deployment-guides.md)
* [Amazon Web Services](deployment/amazon-web-services/README.md)
  * [Active-Active HA for NGINX Plus on AWS Using AWS Network Load Balancer](deployment/amazon-web-services/active-active-ha-for-nginx-plus-on-aws-using-aws-network-load-balancer.md)
  * [Active-Passive HA for NGINX Plus on AWS Using Elastic IP Addresses](deployment/amazon-web-services/active-passive-ha-for-nginx-plus-on-aws-using-elastic-ip-addresses.md)
  * [Global Server Load Balancing with Amazon Route 53 and NGINX Plus](deployment/amazon-web-services/global-server-load-balancing-with-amazon-route-53-and-nginx-plus.md)
  * [Creating Amazon EC2 Instances for NGINX Open Source and NGINX Plus](deployment/amazon-web-services/creating-amazon-ec2-instances-for-nginx-open-source-and-nginx-plus.md)
  * [Setting Up an NGINX Demo Environment](deployment/amazon-web-services/setting-up-an-nginx-demo-environment.md)
* [Global Server Load Balancing](deployment/global-server-load-balancing/README.md)
  * [Global Server Load Balancing with NS1 and NGINX Plus](deployment/global-server-load-balancing/global-server-load-balancing-with-ns1-and-nginx-plus.md)
* [Google Cloud Platform](deployment/google-cloud-platform/README.md)
  * [All-Active HA for NGINX Plus on the Google Cloud Platform](deployment/google-cloud-platform/all-active-ha-for-nginx-plus-on-the-google-cloud-platform.md)
* [Load Balancing Third-Party Servers](deployment/load-balancing-third-party-servers-1/README.md)
  * [Load Balancing Apache Tomcat Servers with NGINX Open Source and NGINX Plus](deployment/load-balancing-third-party-servers-1/load-balancing-third-party-servers.md)
  * [Load Balancing Microsoft Exchange Servers with NGINX Plus](deployment/load-balancing-third-party-servers-1/load-balancing-microsoft-exchange-servers-with-nginx-plus.md)
  * [Load Balancing Node.js Application Servers with NGINX Open Source and NGINX Plus](deployment/load-balancing-third-party-servers-1/load-balancing-node.js-application-servers-with-nginx-open-source-and-nginx-plus.md)
  * [Load Balancing Oracle E-Business Suite with NGINX Plus](deployment/load-balancing-third-party-servers-1/load-balancing-oracle-e-business-suite-with-nginx-plus.md)
  * [Load Balancing Oracle WebLogic Server with NGINX Open Source and NGINX Plus](deployment/load-balancing-third-party-servers-1/load-balancing-oracle-weblogic-server-with-nginx-open-source-and-nginx-plus.md)
  * [Load Balancing Wildfly and JBoss Application Servers with NGINX Open Source and NGINX Plus](deployment/load-balancing-third-party-servers-1/load-balancing-wildfly-and-jboss-application-servers-with-nginx-open-source-and-nginx-plus.md)
* [Migrating Configuration from Hardware ADCs](deployment/migrating-configuration-from-hardware-adcs/README.md)
  * [Migrating Load Balancer Configuration from Citrix ADC to NGINX Plus](deployment/migrating-configuration-from-hardware-adcs/migrating-load-balancer-configuration-from-citrix-adc-to-nginx-plus.md)
  * [Migrating Load Balancer Configuration from F5 BIG-IP LTM to NGINX Plus](deployment/migrating-configuration-from-hardware-adcs/migrating-load-balancer-configuration-from-f5-big-ip-ltm-to-nginx-plus.md)
* [Enabling Single Sign-On for Proxied Applications](deployment/enabling-single-sign-on-for-proxied-applications/README.md)
  * [Enabling Single Sign-On for Proxied Applications with Okta, NGINX Plus, and OpenID Connect](deployment/enabling-single-sign-on-for-proxied-applications/enabling-single-sign-on-for-proxied-applications-with-okta-nginx-plus-and-openid-connect.md)
  * [Enabling Single Sign-On for Proxied Applications with OneLogin, NGINX Plus, and OpenID Connect](deployment/enabling-single-sign-on-for-proxied-applications/enabling-single-sign-on-for-proxied-applications-with-onelogin-nginx-plus-and-openid-connect.md)

## Releases

* [Releases](releases/releases.md)

## Technical Specifications

* [Technical Specifications](technical-specifications/technical-specifications.md)

## Open Source Components

* [Open Source Components](open-source-components/open-source-components.md)

## FIPS Compliance

* [FIPS Compliance](fips-compliance/fips-compliance.md)

## Directives

* [Directives](directives/directives.md)

