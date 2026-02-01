<!-----

You have some errors, warnings, or alerts. If you are using reckless mode, turn it off to see useful information and inline alerts.
* ERRORs: 0
* WARNINGs: 0
* ALERTS: 2

Conversion time: 5.565 seconds.


Using this Markdown file:

1. Paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs™ to Markdown version 2.0β2
* Sun Feb 01 2026 03:53:22 GMT-0800 (Pacific Standard Time)
* Source doc: Understanding of k8s Network Security
* Tables are currently converted to HTML tables.
* This document has images: check for >>>>>  gd2md-html alert:  inline image link in generated source and store images to your server. NOTE: Images in exported zip file from Google Docs may not appear in  the same order as they do in your doc. Please check the images!

----->


<p style="color: red; font-weight: bold">>>>>>  gd2md-html alert:  ERRORs: 0; WARNINGs: 0; ALERTS: 2.</p>
<ul style="color: red; font-weight: bold"><li>See top comment block for details on ERRORs and WARNINGs. <li>In the converted Markdown or HTML, search for inline alerts that start with >>>>>  gd2md-html alert:  for specific instances that need correction.</ul>

<p style="color: red; font-weight: bold">Links to alert messages:</p><a href="#gdcalert1">alert1</a>
<a href="#gdcalert2">alert2</a>

<p style="color: red; font-weight: bold">>>>>> PLEASE check and correct alert issues and delete this message and the inline alerts.<hr></p>


**Understanding of K8s Network Security**

By : Simran Raitani

Table of Contents:



* Abstract 
* Introduction : The State of Network Security 

              The “Flat Network” Problem 


              The Risk of Lateral Movement 


              Zero Trust Architecture

* Overview of K8s Network Security 

              C’s of Cloud Native Security 


               K8s Network Policies 


               Default Deny Strategy 


                Limitations 

* Issue with Scalability of Standard Networking

                Performance Bottleneck


                Visibility Gaps


                IP-based Security Vs Identity-based Security

* Cilium : Next-generation  Networking with eBPF

                Introduction to Cilium


                eBPF Evolution 


                Cilium’s Identity Based Security


                Advanced Filtering

* Managing Security at Scale with Cilium

                Cilium Network Policy


                Global Network Policies


                Transparent Encryption


                 Mutual TLS (mLTS)

* Observability 

              Cilium Hubble


              Service Maps


              Network Forensics

* Summary

              Future of eBPF


              Conclusion

1. **ABSTRACT **

In the ever-evolving world of technology , security is one of the major concerns for big as well as small organisations. As businesses heavily rely on Kubernetes to manage their applications , security of containerized environments has become paramount. In this guide we will explore K8s Network Security. We will explore the shift from IP-based filtering to Zero-trust , Identity-based filtering. 

We will learn about Computer Network Interface (CNI) that act as the enforcement layer of our cluster and how cilium manages eBPF technology to provide high performance security as well as observability to our containerized environments.



2. **Introduction : The State of Network Security**

Kubernetes is the heart of the cloud-native applications. Multiple features and configurations , increases the attack surface of these environments. Misconfigured clusters , vulnerable images and exposed APIs creates a high risk environment that can easily become an entry point for the attackers. 

According to Gartner, by 2026, 90% of organizations running containers will experience a security incident due to misconfigurations.  Major kubernetes threats happen because of Misconfigured K8s Clusters , Exposed API Server , Vulnerable Container Images , Supply Chain Attach , Insecure Secret Management  and Runtime Attack. Hence , security of these clusters and workloads become paramount.

   

2.1 The “Flat Network” Problem 

By default , Kubernetes Network policy follows a flat network policy meaning all devices are connected to each other via a single network , without any hierarchy or control. It is assumed that everything inside the network is safe and can communicate to each other. A frontend pod which is exposed to the internet can directly communicate with the database pod without any rules or policies. A flat network is suitable for smaller applications but for large scale applications , it increases the attack surface for the attackers. If an attacker exploits a vulnerability in an unintended web service inside a cluster , the flat network allows them to scan the entire cluster.



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


2.2 The Risk of Lateral Movement

Kubernetes is “allow-all” by default . There are no firewalls between development namespace and production namespace unless we manually add them. Multiple Pods often share the same underlying Node. If an attacker can "break out" of a container and reach the Node's kernel, they can effectively see all traffic for every other Pod on that specific server. Several containers are created and destroyed every other instance , hence it’s difficult for security tools to keep track of the IP-address of each pod inside the cluster. Since k8s assign IP addresses individually to each pod. 

2.3 Zero Trust Architecture

Zero trust architecture in Kubernetes refers to a security model that assumes every user , service or pod as untrustable. Everything inside or outside the cluster can’t be implicitly trusted. This model helps to reduce the risk of lateral movement due to the flat network model. Every request inside or outside the cluster is strictly authenticated , authorised and inspected. 

This shift in mindset is important because traditional perimeter-based security models no longer hold up in cloud-native environments. Kubernetes clusters are dynamic, distributed, and often exposed to multiple attack surfaces — from misconfigured Role-Based Access Control (RBAC) to insecure pod-to-pod communication.

At its core, Zero Trust in Kubernetes revolves around four key principles: verify explicitly, enforce least privilege, assume breach, and continuously monitor. The attack surface of Kubernetes is more than any monolithic system. Hence , implementing measures which are safe and scalable is very important.

3. Overview of K8s Network Security

3.1 4 C’s of Cloud Native Security 

The 4C’s of cloud native security are cloud , cluster , container and code. Each layer of the security model is built upon the outermost layer. Security of the code layer depends upon the strong foundation base of the (cloud ,cluster and container) layer. Ensuring security at the code layer doesn’t guarantee security at the base layer. Securing each layer is paramount.

**Cloud **

This is the outermost  layer and also the foundation of the kubernetes cluster. If the cloud layer is vulnerable or misconfigured , then there is no guarantee of security at the upper layers. Each cloud provider gives security recommendations for securing the workloads within the environment.


<table>
  <tr>
   <td><strong>Area of concern for k8s infrastructure</strong>
   </td>
   <td><strong>Recommendation</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Network access to API Server</strong>
   </td>
   <td>All access to kubernetes control plane is not allowed publicly on the internet  and is controlled by the network access lists restricted to the list of IP addresses needed to administer the cluster.
   </td>
  </tr>
  <tr>
   <td><strong>Network access to nodes</strong>
   </td>
   <td>Nodes should be configured to accept connections (via network access control list) from the control plane on the specified port. These nodes should not be publicly exposed on the internet.
   </td>
  </tr>
  <tr>
   <td><strong>Kubernetes Access to Cloud Provider API </strong>
   </td>
   <td>Each cloud provider needs to grant a set of permissions to the kubernetes control plane and nodes. It is very important that it follows the principle of least privilege.
   </td>
  </tr>
  <tr>
   <td><strong>Access to etcd</strong>
   </td>
   <td>Access to etcd (datastore of kubernetes) should only be limited to the control plane.Access to etcd should be made using TLS.
   </td>
  </tr>
  <tr>
   <td><strong>Etcd Encryption</strong>
   </td>
   <td>Since etcd is the database store of the cluster. It is very important to protect the data both at transit and at  rest.
   </td>
  </tr>
</table>


** Cluster**

Depending on the attack surface of your application, you may want to focus on specific aspects of security. For example: If you are running a service (Service A) that is critical in a chain of other resources and a separate workload (Service B) which is vulnerable to a resource exhaustion attack, then the risk of compromising Service A is high if you do not limit the resources of Service B. 

**Container**

The following is the list  of measures to secure the container :


<table>
  <tr>
   <td><strong>Area of concerns for kubernetes container</strong>
   </td>
   <td><strong>Recommendation</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Container vulnerability scanning and os dependency security </strong>
   </td>
   <td>As part of the image build step , container vulnerability scanning is important.
   </td>
  </tr>
  <tr>
   <td><strong>Image signing and enforcement</strong>
   </td>
   <td>Sign container images to maintain trust for the content of the containers.
   </td>
  </tr>
  <tr>
   <td><strong>Disallow privileged user</strong>
   </td>
   <td>Monitor user with high privilege to the operating system.
   </td>
  </tr>
  <tr>
   <td><strong>Use container runtime with strong isolation</strong>
   </td>
   <td>Select container runtime classes for strong isolation.
   </td>
  </tr>
</table>


**Code **

Application code is one of the most common attack surfaces. It is very important to use the practice of secure-coding. Following is the list of security practices to follow for a secure-code 


<table>
  <tr>
   <td><strong>Area of concern for code</strong>
   </td>
   <td><strong>Recommendation</strong>
   </td>
  </tr>
  <tr>
   <td><strong>Access over TLS only</strong>
   </td>
   <td>Communication should happen over TLS , encryption of data both at rest and transit must be ensured.
   </td>
  </tr>
  <tr>
   <td><strong>Limiting number of open ports</strong>
   </td>
   <td>Reducing the attack surface by limiting the number of open ports . Use “Default deny” policy and allow only those ports that are needed.
   </td>
  </tr>
  <tr>
   <td><strong>3rd party dependency security</strong>
   </td>
   <td>It is a good practice to regularly scan the application’s 3rd party dependency libraries to identify possible known vulnerability.
   </td>
  </tr>
  <tr>
   <td><strong>Static code analysis</strong>
   </td>
   <td>Scanning of codebases for possible vulnerabilities ensures secure-coding practices.
   </td>
  </tr>
  <tr>
   <td><strong>Dynamic probing attacks</strong>
   </td>
   <td>Run tools that can help identify the known threats to the service.
   </td>
  </tr>
</table>


3.2 Kubernetes Network Policies

Kubernetes Network Policies are the primary mechanism for controlling traffic flow at the IP address or port level (Layer 3 and 4). By default, Pods are "non-isolated"; they accept traffic from any source. Network Policies allow administrators to specify how a "selected" group of Pods is allowed to communicate with each other and other network endpoints.

Network Policies use Labels and Selectors to determine which workloads the rules apply to. There are two main types of traffic regulated:



* Ingress: Rules defining what incoming traffic is allowed to reach the Pod.
* Egress: Rules defining what outgoing traffic the Pod is allowed to send.

These policies are additive. If multiple policies apply to a Pod, the "allow" rules from all policies are combined (a logical OR operation).

3.3 Default Deny strategy

The most critical best practice in Kubernetes security is the Default Deny Strategy. This aligns with the principle of "Least Privilege." Instead of trying to block specific "bad" traffic, you start by blocking all traffic and then selectively "whitelisting" only the communication paths that are strictly necessary for the application to function.

To implement this, a security team applies a "Deny-All" policy to a namespace. This policy selects all Pods in the namespace but contains an empty set of allowed ingress/egress rules.

The benefits of this approach include:



* Eliminating Blind Spots: Any new Pod deployed in the namespace is automatically isolated by default.
* Hardened Segmentation: It prevents unintended communication between microservices that have no business talking to one another.
* Operational Clarity: Since all traffic is blocked, every "Allow" rule serves as documentation for the application's intended network dependencies.

3.4 Limitations

While native Network Policies are a great starting point, they have significant limitations that make them difficult to manage in large-scale, 2026-standard enterprise environments:


### 1. The "L4 Wall" (Lack of Protocol Awareness)

Standard policies operate at the Layer 3/4 level. They can see that Pod A is talking to Pod B on Port 80, but they cannot see what they are saying. They cannot distinguish between a safe GET request and a destructive DELETE request. This makes them blind to API-level attacks.


### 2. Performance Degradation (The iptables Bottleneck)

Most standard Kubernetes networking implementations use iptables to enforce these policies. iptables checks every packet against a sequential list of rules. As the number of services and rules grows into the thousands, the time it takes to process a single packet increases linearly, leading to high latency and CPU overhead.


### 3. The "Silent Drop" Problem (Lack of Observability)

Standard Network Policies provide no built-in logging. When a packet is blocked, it is "silently dropped." For a DevOps engineer, this makes troubleshooting a "connectivity issue" a nightmare, as there is no record of which policy blocked the traffic or why.


### 4. Fragility of IP-Based Logic

Traditional policies often rely on IP blocks (CIDRs) for external traffic. In a dynamic cloud environment, IP addresses are constantly changing, making these hardcoded rules difficult to maintain and prone to "security drift."



3. Issue with Scalability of Standard Networking

    As Kubernetes clusters grow from dozens to thousands of nodes, the underlying networking infrastructure often becomes the "silent killer" of performance. Standard networking was designed for connectivity, not for the high-density, high-churn environments common in 2026. When organizations scale, they hit three primary walls: performance bottlenecks, visibility gaps, and the fragility of IP-based security.



### 4.1 Performance Bottleneck: The iptables Limit

Standard Kubernetes networking relies on `kube-proxy` in `iptables` mode. In this model, every Service and every Endpoint is translated into a sequential list of rules in the Linux kernel.



* **The Sequential Search Problem:** `iptables` is non-indexed. Every time a packet arrives, the kernel must check it against every rule from top to bottom. In a cluster with 5,000 services, a packet might have to pass through 10,000+ checks before it is routed.
* **The Update Latency:** Whenever a single Pod is added or deleted, the entire `iptables` ruleset must be re-calculated and re-written. In large clusters, this "sync period" can take several seconds, leading to "connection refused" errors during scaling events.


### 4.2 Visibility Gaps: Managing the "Black Box"

Standard networking operates at **Layer 4 (L4)**, meaning it only understands IP addresses and ports. For a modern enterprise, this creates a significant "Black Box" effect:



* **No Protocol Insight:** You can see that Pod A is talking to Pod B on port 443, but you cannot see if it is a legitimate API call or a data exfiltration attempt.
* **Silent Packet Drops:** When a standard Network Policy blocks a packet, it is a "silent drop." There is no native Kubernetes log that tells an engineer *why* a connection failed, leading to hours of wasted troubleshooting time.
* **The Metadata Disconnect:** Standard network logs show IPs, but in Kubernetes, IPs are ephemeral. By the time a security team investigates an alert, the Pod with that IP may already be gone, leaving no trail to the actual workload.

**4.3 IP-based Security vs. Identity-based Security**

Traditional IP-based security relies on the network location—the IP address—as the primary identifier for trust, assuming that any traffic originating from a "known" IP is safe. This model is notoriously fragile in Kubernetes environments because Pods are ephemeral; they are frequently destroyed, rescheduled, and assigned new IP addresses, which forces the system to constantly recalculate and update thousands of firewall rules across the cluster, leading to significant CPU overhead and high latency. In contrast, identity-based security (the foundation of Zero Trust and tools like Cilium) decouples security from the network topology by assigning each workload a unique, persistent Security Identity based on its metadata and labels (e.g., app: payment-gateway). This identity follows the workload regardless of its IP address or which node it runs on, allowing security policies to remain static and performant even as the underlying infrastructure churns. For an organization, this shift means moving from a reactive, brittle defense that checks "where" a packet is from to a proactive, scalable defense that verifies "who" the workload is before permitting a single byte of data to transfer.


## **5. Cilium: Next-Generation Networking with eBPF**


### **5.1 Introduction to Cilium**

Cilium is an open-source, cloud-native project designed to provide high-performance networking, security, and observability for Kubernetes workloads. Cilium was developed from the ground up to manage the scale and dynamic nature of contemporary microservices, in contrast to traditional CNIs that rely on outdated Linux networking tools. It replaces kube-proxy as a unified data plane, providing deep visibility into cluster communications, enforce security, and route traffic more effectively.


### **5.2 eBPF Evolution**

eBPF (Extended Berkeley Packet Filter) is the innovation that powers Cilium's performance. The Linux kernel used to contain networking logic, but it was inflexible and took a long time to evolve. By enabling developers to run sandboxed programs inside the kernel without altering the kernel source code, eBPF advanced this. By avoiding the overhead of "context switching" between user-space and kernel-space, which slows down conventional firewalls, Cilium is able to intercept and process network packets at nearly native hardware speeds.


### **5.3 Cilium’s Identity-Based Security**

Cilium revolutionizes security by moving away from IP addresses. Instead, it assigns a **Security Identity** to every Pod based on its Kubernetes labels. When a packet is sent, Cilium "stamps" it with this identity. The receiving node checks this identity against its policy—if the "Frontend" identity isn't authorized to talk to the "Database" identity, the packet is dropped immediately in the kernel. This ensures that security remains intact even if Pods are constantly restarting and changing IPs.


### **5.4 Advanced Filtering (Layer 7)**

While standard Kubernetes policies only see IP ports (Layer 4), Cilium’s advanced filtering extends to the application layer (**Layer 7**). This allows security teams to inspect protocol-specific traffic such as HTTP, gRPC, and Kafka. For example, a policy can be set to allow a service to "Read" from a specific API endpoint but "Deny" any "Write" or "Delete" commands, providing a much more granular level of protection against sophisticated attacks.



<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")



---


## **6. Managing Security at Scale with Cilium**


### **6.1 Cilium Network Policy (CNP)**

The native Kubernetes policy is superseded by the Cilium Network Policy. Advanced features like DNS-aware rules (like "Allow traffic only to api.google.com"), CIDR filtering for external services, and the previously mentioned Layer 7 API constraints are all made possible by it. This makes it possible to write intricate security logic in a human-readable YAML format that is simple to audit and maintain version control.


### **6.2 Global Network Policies**

Through its "ClusterMesh" feature, Cilium offers Global Network Policies for enterprises operating across several clusters or cloud regions. This makes it possible to implement a single security policy throughout the whole global infrastructure. Cilium maintains consistent identity-aware security regardless of location by treating "Web" services in the US and "Database" in Europe as if they were in the same cluster.

**6.3 Transparent Encryption**

Cilium provides Transparent Encryption with IPsec or WireGuard. Without requiring any modifications to the application code or the use of bulky "sidecar" proxies, this automatically encrypts all traffic between Pods in the cluster. Cilium guarantees data-in-transit security with minimal latency by managing encryption at the kernel level via eBPF.

**6.4  Mutual TLS (mTLS)**

A "Sidecar-less" mutual TLS (mTLS) is implemented by Cilium. Traditionally, mTLS necessitates injecting a proxy (such as Envoy) into each Pod, which uses a lot of RAM and CPU. Cilium uses eBPF to manage the encryption and authentication handshake in the data plane. For a fraction of the resource cost, this offers the same high level of encryption and service-to-service authentication.


---


## **7. Observability**


### **7.1 Cilium Hubble**

The distributed observability layer on top of Cilium is called Hubble. It provides deep network visibility without adding "agent" overhead by using eBPF. By removing the "black box" aspect of conventional Kubernetes networking, Hubble enables teams to see exactly what is happening in their network in real-time.

**7.2 Maps of Services**

The Service Map is Hubble's most potent feature. It offers a graphical depiction of the communication between services in real time. Engineers can quickly determine which services are in good condition, which have high latency, and which have security policies blocking their attempts to connect.

**7.3 Forensic Network Analysis**

Hubble offers Network Forensics tools in case of a production outage or security breach. Every flow is meticulously recorded, along with the source and destination identities and the particular policy that permitted or prohibited the traffic. This facilitates quick root-cause analysis and streamlines the compliance auditing process.


---


## **8. Summary**


### **8.1 Future of eBPF**

The future applications of eBPF extend well beyond networking. eBPF is increasingly utilized for runtime security, such as monitoring system calls and file access, as well as for deep profiling of application performance. As the eBPF ecosystem expands, it is expected to become a universal interface for interacting with the Linux kernel, thereby enhancing infrastructure safety, performance, and observability.


### **8.2 Conclusion**

Kubernetes network security now requires the establishment of a high-performance, Zero Trust environment that can scale with organizational growth. Cilium, which leverages eBPF, addresses previous performance bottlenecks while delivering identity-aware security and comprehensive observability necessary for future requirements. Adopting Cilium represents a significant advancement for organizations seeking to secure their cloud-native infrastructure.
