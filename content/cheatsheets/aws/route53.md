---
title: Route53
linktitle: 🛣️ Route53
type: book
tags:
  - AWS
date: "2019-05-05T00:00:00+01:00"
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 75
---

Scalable DNS and Domain Name Registration

<!--more-->

## Overview

In AWS, Route53 is global managed ```DNS``` (Domain Name System) & we already know DNS is a collection of rules and records which helps clients understand how to reach a server through URLs. DNS operates on port 53. Amazon decided to call it Route53 so that’s where the name comes from.

## Features

Route 53 offers the following functions:

* **Domain name registrar**:

  * A domain name **registrar** is an organization that manages the reservation of Internet domain names like:

    * GoDaddy
    * Namecheap
    * Etc…
    * And also… Route53 (i.e. AWS)!
  * You can register a domain with other registrars also and still use Route53's other functions. Say for eg. if you have bought domain from 3rd party (eg: Go Daddy), you can use it in AWS Route53 by creating a ```hosted zone``` in Route53 & update ```NameServer``` records on 3rd party website to use Route53 name servers.
  * You can transfer your domain from another registrar to Route 53
  * You can register your domain with Route 53 and not use other functionality like DNS service or health checks if you want to.

* **DNS resolution**:

  * Route53 translates domain names to **IPs**.
  * Route53 responds to DNS queries using a global network of authoritative DNS servers.
  * You can transfer DNS service (from another registrar) to Amazon Route53 with or without transferring registration for the Domain
  * If you register a domain with Route53, it will automatically be configured as the DNS service for the domain by doing the following:

    * Creates a ```hosted zone``` with the same name as your domain.
    * Assigns a set of 4 nameservers to the hosted zone. When someone uses a browser to access your website like www.example.com these nameservers tells the browser where to find your resources like Webserver or S3 bucket.
    * Gets the nameservers from the hosted zone and adds them to your domain.

* **Health checking**:

  * **Monitor** the health and performance of your application’s servers, or endpoints, from a network of health checkers in locations around the world.
  * You can specify either a domain name or an IP address and a port to create HTTP, HTTPS, and TCP health checks that check the health of the endpoint.
  * You can configure AWS ```CloudWatch Alarms``` for your health checks so that you get notified when a resource becomes unavailable.
  * Routing **policies** could be configured to route traffic depending on the DNS health checks.

## Key Terms

  ###### Top Level Domains (TLDs)

  * The TLD is the farthest position to the right (as separated by a dot).
  * Parties can distribute domain names under the TLD usually through a domain name registrar.
  * Generic TLDs: Like .com, .net, .org etc.
  * Geographic TLDs: Like .us, .fr, .in etc.  

  ###### Domain Names

  * **Human friendly** name associated with an internet resource (cloud-native.wiki is a domain name)

  ###### Subdomains

  * Every domain name except the root domain name is called a Subdomain (api.cloud-native.wiki is a subdomain)

  ###### Hosts

  * Hosts are instances or services accessible via a domain.

  ###### Name Servers

  * Name Servers are servers in the DNS that translates domain names to IP addresses.

    * **Authoritative** servers provide answers to queries about domains under their control.
    * **Non-Authoritative** servers point to other servers or serve cached copies of other Name Servers data.

  ###### Zone files

  * A zone file is a simple text file that contains mappings between domain names and IP addresses.
  * Zone files reside in the Name Servers
  * The more zone files a Name Server has, the more requests it will be able to answer authoritatively.

  ###### TTL (Time to Live)

  * TTL is length of time that a DNS records are cached on either the resolving server or user owned Laptop.
  * The Lower the TTL, the faster changes to DNS records.
  * Whenever you created record set, you need to define TTL for it.

## DNS resolution

{{< figure src="images/uploads/dns-resolution.PNG" width="350" height="350" class="alignright">}}
{{< figure src="images/uploads/cloudformation-metadata-structure.png" width="300" height="1000" class="alignright">}}


All URLs map to an IP address. When you request a URL, you are actually instructing the system to find the IP address that is associated with the URL, and then the computer connects to that IP address.

1. **Checking the local cache**:
Computer: Use its own cache — If not found, move to step 2.
2. **Checking the Name Resolving Server**:
Usually, the Name Resolving Server is your Internet Service Provider (ISP). Checks Name Resolving Server Cache — If not found, move to step 3.
3. **Check Root Server**:
Get the Top Level Domain server IP address and tell the Name Resolving Server — Got TLD Server Address.
Move to step 4.
4. **Check TLD Server**:
Got the .com Domain Server IP address and give it to Name Resolving Server. Move to Step 5.
5. **Check Domain Level Name Server**:
The Name Server looks up the zone file associated with http://example.com server IP address. Pass this to Name Resolving Server and cache it. Pass this to the computer and cache it.

## Record Types

Whenever we register a domain in Route53, it creates a hosted zone as well. A hosted zone is a container for records, and records contain information about how you want to route traffic for a specific domain, such as example.com, and its subdomains (api.example.com). A hosted zone and the corresponding domain have the same name.

When we create a hosted zone in Route53, two types of records gets automatically created

  * **SOA**: Basic SOA stores information about below things.

    * Name of Server that supplied the data for zone.
    * The administrator of that zone & current version of data file.

  * **NS**: NS records is basically your name server records which are used by top level domain servers to direct traffic to content DNS server which contains the authoritative records.

  ![Route53-RecordTypes](/images/uploads/route53-record-types-1.PNG)

You could also register your domain with another registrar and then manually create a hosted zone in Rout53.
Here are some tutorials for some of the most popular domain providers on how to change the domain’s nameservers:
  * [Godaddy](https://www.godaddy.com/help/change-nameservers-for-my-domains-664)
  * [Namecheap](https://www.namecheap.com/support/knowledgebase/article.aspx/767/10/how-to-change-dns-for-a-domain/)

  In addition the most common records are:

  * **A**: The “A” record stands for Address record. The A record is used by computer to translate the name of the domain to an IP address.
  ![Route53-A-record](/images/uploads/route53-A-record.PNG)
  ![Route53-A-record](/images/uploads/route53-A-record-nslookup.PNG)

  * **AAAA**: hostname to IPv6

  * **CNAME** (Canonical Records- URL to URL): CNAME Points a URL to any other URL. (app.mydomain.com => blabla.anything.com). We can use it only for Subdomains (Non-Root Domains). AWS Resources (Load Balancer, CloudFront…) expose an AWS hostname: lb1-1234.us-east-2.elb.amazonaws.com and you want myapp.mydomain.com
  ![Route53-CNAME-record](/images/uploads/route53-CNAME-record.PNG)

  * **Alias**: Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com). Alias record are used to map resource record sets in your hosted zone to Elastic Load Balancer, CloudFront or S3 Buckets websites.
    * Works for ROOT DOMAIN and NON ROOT DOMAIN (aka mydomain.com)
    * Free of charge
    * Native health check

    ![Route53-Alias-record](/images/uploads/route53-Alias-record.PNG)

## Routing Policies

### Simple Routing:
Redirect to single resource, can’t attach health check, If multiple records are attached, random one will be selected.

### Weighted Routing:
“N” % requests will go to specific Endpoint, It’s helpful to test 5–10% traffic on new application version, can attach health check.

### Latency Routing:
Redirect to the server that has the least latency close to us, latency is calculated in terms to AWS Region, health check attached.

### Failover Routing:
If primary resource is not working, traffic is redirect to secondary instance/resource. Health check is mandatory.

### Geo-location Routing:
Routing is based on user location. Specify that, traffic from XYZ location should go always to particular instance/resource, if it doesn’t match, should go to default policy(We define this also).

### Multi-Value:
Use when, traffic needs to go to multiple resources, health check mandatory. It’s not substitute for having an ELB.
