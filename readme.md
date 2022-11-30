# Hybrid DNS Solutions in Azure
One of the main challenges organizations are facing in moving towards the cloud is how to deal with DNS. Traditionally, on-premises DNS-servers would be responsible for name resolution. This would often involve multiple machines hosting internal zones and forwarding requests for public domains to external DNS-servers. 

Working in a hybrid environment, where resources are located not only on-premises, but also in the cloud introduces additional challenges. Common questions asked when dealing with these challenges:

- Do we lift & shift?
- What architectural changes do we require? 
- Are there native capabilities we should be utilizing? 
- How do we make sure our DNS Solution has no ambiguity when it comes to traffic flows?

Understandably, all of this can feel quite daunting. The goal of this guideline is outlining the common DNS solutions, understanding the benefits/disadvantages of each solution and to help make an informed decision. Sadly, there is no 'best' or 'one-size-fits-all' solution. This goes for pretty much everything we deploy in the cloud and DNS Solutions are no exception.

## Assessment
Before diving into the different solutions, it is important to assess the current DNS setup and objectives for the new solution. The goal is to have a good understanding of the situation and the desired outcome. 

Whether it is being new to the cloud, needing to meet deadlines or other circumstances, many organizations proceed too quickly without a plan in place. This could potentially lead to a technical debt and need of fixing retro-actively.

For example, if no IaaS platform is desired or even against internal/external compliancy rules, some of the solutions described will not be preferred. Conversely, if other requirements take precedence, you might be left with no alternative. Going back to the drawing board after deployment is not desired.

### What does our current DNS Solution look like? 
---
Start with identifying the different workloads. E.g.: offices in different regions, remote users, applicationbased DNS, use of proxies, dependencies 
and DNS-traffic flows. Make sure to document these findings as it will help to find potential blindspots before having to deal with them in retrospect.  

### Identify the technical and business requirements
---
What are the rules this new solution needs to be compliant with? Are there limitations on certain services? Are we targeting specific RTO/RPO/SLA's for this new solution? What other factors are in play? Good starting point is to refer to the Well-Architected Framework and analyze whether your new solution fit in with all of the WAF-pillars. This is a crucial step in the assessment, providing insight into the framework to which this solution needs to fit. 

### What challenges are we facing in our current DNS Solution and are there noteworthy aspects of this change to take into consideration? 
---
Identify these challenges, utilize lessons learned and set clear goals on what challenges should be fixed in the new solution. Each environment is different, you might be dealing with certain things that is very specific your organization. Document these findings and have clear objectives on how to deal with them.  

### Which stakeholders and assets will be impacted by this change? 
---
Analyze what impact a change in DNS would have and what measures to take to limit the impact. Make sure there is no ambiguity as to how DNS-traffic will flow and how this is going to impact flows such as routing and security.

 

## DNS Scenario 1
![](https://github.com/infobozk/Hybrid-DNS/blob/6201c4e0e441fafa251eb17a31831fe981ae4e31/images/HybridDNS-1.png)
![](https://github.com/infobozk/Hybrid-DNS/blob/91532108477e49b2214e594e90ced4334f0e6f06/images/DNSFlow-1.png)

Scenario 1 describes one of the more commonly used DNS Solution:

- On-premises locations with local DNS-servers
- Connectivity to Azure by ExpressRoute and/or VPN
- IaaS DNS VM's hosted in Azure 

This solution is straightforward in its design and setup, it provides the managing team with a familiar setup. In this scenario, the existing DNS-solution is extended by adding in 'another datacenter' (Azure).

Benefits of this scenario:
- DNSSec support available with Windows DNS Service and some other 3rd party solutions
- DNS-policy for applying filters & split-brain operations
- Automatic replication of all hosted zones (can be modified if desired)
- Works across cloud providers 
- Centralized servers can be configured as the main DNS-servers hosting the zones, local DNS-servers (e.g., in branch locations) could be read-only servers. This provides the capability to manage all DNS-entries from a centralized environment. 

Disadvantages of this scenario:
- Requires IaaS-components which will need to be maintained
- Limited automation capabilities for non-domain environments, AD-domain joined machines would automatically get a DNS-entry in the hosted zone. For environments where domains are not used, additional automation or software/tooling would be required to add new entries into the hosted zone.

Note: there is no conditional forwarding rule configured. For name resolution for Azure resources such as PrivateLink endpoints you would be required to host those zones on your DNS-servers. 

Additionally, some organisations opt to make the Azure hosted DNS VM's the primary DNS-servers. Local DNS-servers in branch locations and on-premises are configured as ReadOnly machines, transferring all zone information from the centralized DNS-servers. Benefit of this type of setup is a single-pane-of-glass type of DNS solution. Most administrative tasks relating to DNS would only be performed on the centralized servers.

## DNS Solution 2
![](https://github.com/infobozk/Hybrid-DNS/blob/6201c4e0e441fafa251eb17a31831fe981ae4e31/images/HybridDNS-2.png)
![](https://github.com/infobozk/Hybrid-DNS/blob/91532108477e49b2214e594e90ced4334f0e6f06/images/DNSFlow-2.png)
An alternative to solution 1 is configuring the on-premises DNS-servers with conditional forwarding. In case the on-premises client requires name resolution for certain domains (e.g., privatelink), the on-premises DNS-server will forward these requests towards Azure.

The same pros and cons apply to this solution, but there are also some notable difference:

- Conditional forwarding rules must be setup and maintained 
- A dependency is created by having this conditional forwarding. For name resolution to work, the connectivity between on-prem and Azure needs to be up and running. In most use-cases this should not be an issue, if the connection is down there is no traffic flow either way. So even if the name resolution did work, traffic would not reach the destination. However, in certain use-cases, this is not desired. Be aware of this when designing your infrastructure.

## DNS Scenario 3
![](https://github.com/infobozk/Hybrid-DNS/blob/e96a7e5d67d2869f0feef56c2752e90eef649b7e/images/HybridDNS-3.png)
![](https://github.com/infobozk/Hybrid-DNS/blob/91532108477e49b2214e594e90ced4334f0e6f06/images/DNSFlow-3.png)
A different variation to this kind of setup is to replace the IaaS VM in Azure by the Azure Private DNS Resolver.

The official documentation mentions the following benefits:

- Fully managed: Built-in high availability, zone redundancy.
- Cost reduction: Reduce operating costs and run at a fraction of the price of traditional IaaS solutions.
- Private access to your Private DNS zones: Conditionally forward to and from on-premises.
- Scalability: High performance per endpoint.
- DevOps Friendly: Build your pipelines with Terraform, ARM, or Bicep.

There are some drawbacks to this setup:

- DNSSec not supported 
- Limited policy functionality compared to traditional DNS-servers
- Subnet restrictions, might conflict with address plan
- No IPv6 support

## DNS Scenario 4
![](https://github.com/infobozk/Hybrid-DNS/blob/17e6bddf2374d965d9380d1ea92370eeecbe31a9/images/HybridDNS-4.png)
![](https://github.com/infobozk/Hybrid-DNS/blob/91532108477e49b2214e594e90ced4334f0e6f06/images/DNSFlow-4.png)

This setup also removes the need for having IaaS infrastructure in Azure and was mostly used by organisations under special circumstances (e.g. Azure Private DNS Resolver was not avail). Azure-hosted resources are able to perform DNS-queries using the DNS Private Zones and interal Azure DNS service. This also comes with the upside of having automatic registration/deregistration of resources inside of Azure (at time of writing limited to the primary IP on the NIC).

A potential disadvantage of this scenario is the name resolution between on-premise and Azure resources. Out of the box this will not be working. Two solutions for this problem:

- Manually keeping the  DNS-environments in sync with the Azure Private Zone, anytime a resource is deployed/modified on-premise that particular change will need to be done on the DNS Private Zone. Similarly, when changes are made on the Azure side, this would need to be set on the local DNS servers.

- Having an automated method (like a script) to keep the DNS Private Zone and local DNS servers in sync, eliminating the need for manual labor.

- This setup is not common and offers very limited benefits compared to the other solutions, it should be viewed as a possible alternative if the other solutions do not meet the requirements and this does. 

# Summary
The solutions described are just some of the options available to us, but at the same time also the more commonly used. It is important to note that the cloud is ever-evolving, best-practices and recommendations change over time. Keep this in mind when using this or any other guideline. 

## Resources
[Azure Hybrid DNS Infra](https://learn.microsoft.com/en-us/azure/architecture/hybrid/hybrid-dns-infra)

[Azure DNS Private Resolver](https://learn.microsoft.com/en-us/azure/architecture/example-scenario/networking/azure-dns-private-resolver)
