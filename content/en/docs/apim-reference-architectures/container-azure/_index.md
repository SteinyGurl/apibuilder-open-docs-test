{
"title": "API Management Container Reference Architecture on Azure",
"linkTitle": "API Management Container Reference Architecture on Azure",
"weight": 30,
"date": "2020-04-29",
"description": "Axway AMPLIFY™ API Management Container Reference Architecture on **Azure**"
}
-- -- ----------------------------------------------------------------------------- --


Architecture
Axway AMPLIFY™ API Management Container Reference Architecture on **Azure**
Version 1.0 April 29, 2020



1.

Document History

Version   Date         Update origin
--------- ------------ ---------------
1.0       2020-04-29   First version

Table of Contents {#table-of-contents .TOC-Heading}
=================

[1. Overview 6](#overview)

[1.1. References 6](#references)

[1.1.1. List of tables 6](#list-of-tables)

[1.1.2. List of pictures 6](#list-of-pictures)

[2. General architecture 7](#general-architecture)

[2.1. Principles 7](#principles)

[2.2. Target use case 8](#target-use-case)

[2.3. Additional components and considerations
9](#additional-components-and-considerations)

[2.3.1. Container registry 9](#container-registry)

[2.3.2. Bastion host 10](#bastion-host)

[2.3.3. DevOps Pipeline 10](#devops-pipeline)

[2.3.4. SMTP Relay 10](#smtp-relay)

[2.4. Performance goals 11](#performance-goals)

[3. Implementation details 12](#implementation-details)

[3.1. Diagram 12](#diagram)

[3.1.1. Technical view 12](#technical-view)

[3.1.2. Application view 13](#application-view)

[3.2. Choice of runtime infrastructure components
13](#choice-of-runtime-infrastructure-components)

[3.2.1. Network specification 14](#network-specification)

[3.2.2. Azure Kubernetes Services (AKS) sizes
17](#azure-kubernetes-services-aks-sizes)

[3.2.3. Cassandra cluster size 17](#cassandra-cluster-size)

[3.2.4. Azure Database for Mysql 18](#azure-database-for-mysql)

[3.2.5. Azure Files Premium (AFP) for shared storage.
18](#azure-files-premium-afp-for-shared-storage.)

[3.2.6. Azure Blob Storage for logs 19](#azure-blob-storage-for-logs)

[3.2.7. Azure App Gateway or Azure Load Balancer
19](#azure-app-gateway-or-azure-load-balancer)

[3.2.8. Azure Container Registry 20](#azure-container-registry)

[3.3. Kubernetes considerations 21](#kubernetes-considerations)

[3.3.1. Deployment options 21](#deployment-options)

[3.3.2. Namespaces 22](#namespaces)

[3.3.3. Pod resource limits 22](#pod-resource-limits)

[3.3.4. Components healthcheck 23](#components-healthcheck)

[3.3.5. Affinity and anti-affinity mode
23](#affinity-and-anti-affinity-mode)

[3.3.6. Autoscaling 24](#autoscaling)

[3.3.7. External traffic 25](#external-traffic)

[3.3.8. Secrets 26](#secrets)

[3.4. API Management implementation details
27](#api-management-implementation-details)

[3.4.1. Admin Node Manager 27](#admin-node-manager)

[3.4.2. API Manager UI 28](#api-manager-ui)

[3.4.3. API Gateway/Manager (traffic) 29](#api-gatewaymanager-traffic)

[3.5. Cassandra considerations 30](#cassandra-considerations)

[3.6. Security considerations 31](#security-considerations)

[3.7. SQL database considerations 31](#sql-database-considerations)

[3.8. Logging/tracing 31](#loggingtracing)

[3.9. Monitoring 32](#monitoring)

[3.10. Environmentalization and Promotion
32](#environmentalization-and-promotion)

[3.11. Performance testing 34](#performance-testing)

[3.11.1. Load test with 60 threads 34](#load-test-with-60-threads)

[3.11.2. Load test with 200 threads 35](#load-test-with-200-threads)

[4. Maintenance 35](#maintenance)

[4.1. New configurations 35](#new-configurations)

[4.2. Product updates 36](#product-updates)

[4.2.1. Installing a patch 36](#installing-a-patch)

[4.2.2. Installing a service pack 38](#installing-a-service-pack)

[4.2.3. Upgrading the product 38](#upgrading-the-product)

[4.2.4. Adding customization 38](#adding-customization)

[4.3. Pushing a new Docker image to your Kubernetes cluster
38](#pushing-a-new-docker-image-to-your-kubernetes-cluster)

[5. Configuration and data backups 39](#configuration-and-data-backups)

[6. Disaster recovery 40](#disaster-recovery)

[7. Known constraints and roadmap 40](#known-constraints-and-roadmap)

[8. Appendix A -- Glossary of Terms 41](#appendix-a-glossary-of-terms)

Summary

This document provides a reference architecture guide for deploying
AMPLIFY API Management (APIM) using Externally Managed Topology ([EMT
mode](https://docs.axway.com/bundle/axway-open-docs/page/docs/apim_installation/apigw_containers/container_getstarted/index.html)).
Deploying APIM using Docker containers orchestrated by Kubernetes brings
tremendous benefits in installing, developing, and operating an API
management solution.

This document describes all major areas in deploying and maintaining
Axway APIM EMT on Microsoft Azure cloud, including:

-   Physical and deployment architectures

-   Explanation and consideration for selecting underlying
infrastructure components

-   Kubernetes considerations

-   Performance, logging and monitoring aspects

-   Backup and recovery, including disaster recovery

-   Constraints and roadmap

Overview
========

Axway AMPLIFY™ API Management is a leading API management solution on
the market. It supports container-based deployment under an option
called Externally Managed Topology (EMT). The purpose of this document
is to share Axway reference architecture for the container-based
deployment of an API management solution on Kubernetes. It will address
many architectural, development, and operational aspects of the proposed
architecture.

Since the technology choices, Docker and Kubernetes, are portable, most
of the information in this guide should apply across an on-premise
environment and many cloud providers. But we include specific
recommendations for Azure as one of the most common deployment targets.

The target audience for the document is architects, developers, and
operations personnel. To get the most value from this document, a reader
should have a good knowledge of Docker, Kubernetes, and API management.

References
----------

### List of tables

[Table 1: Deployment architecture - global recommendations
8](#_Toc39071612)

[Table 2: Sensitive parameters 10](#_Toc39071613)

[Table 3: List of assets 14](#_Toc39071614)

[Table 4: Bastion NSG rules 15](#_Toc39071615)

[Table 5: AKS NSG rules 15](#_Toc39071616)

[Table 6: Data NSG rules 16](#_Toc39071617)

[Table 7: PaaS rules configuration 16](#_Toc39071618)

[Table 8: Ingress controller options 26](#_Toc39071619)

[Table 9: Kubernetes secrets list 26](#_Toc39071620)

[Table 10: Performance validation threshold with 60 threads
34](#_Toc39071621)

[Table 11: Performance validation threshold with 200 threads
35](#_Toc39071622)

### List of pictures

[Figure 1: Layered deployment
8](#_Toc39071623)

[Figure 2: Components overview
9](#_Toc39071624)

[Figure 3: Technical diagram for HA deployment
12](#_Toc39071625)

[Figure 4: Kubernetes objects deployment
13](#_Toc39071626)

[Figure 5: Network flow diagram
14](#_Toc39071627)

[Figure 6: Nodes pool configuration
17](#_Toc39071628)

[Figure 7: Disk space setting in Policy Studio
18](#_Toc39071629)

[Figure 8: Azure Application Gateway with ingress controller
20](#_Toc39071630)

[Figure 9: Azure Load balancer with ingress controller
20](#_Toc39071631)

[Figure 10: API and policy promotion
33](#_Toc39071632)

[Figure 11: Performance diagram
34](#_Toc39071633)

[Figure 12: Creating docker images
36](#_Toc39071634)

General architecture
====================

This chapter is focused on general architecture in support of an API
management deployment on a dedicated Kubernetes cluster. The chapter
discusses architectural principles, as well as required and optional
components. There are many ways to deploy software on a Kubernetes
cluster, but this document shares Axway's experience acquired from
deploying AMPLIFY API Management in an actual production environment.
Most of the implementation details will be outlined in the following
chapters.

Make sure the constraints listed in the chapters are respected in case
of deployment on an existing Kubernetes cluster.

Principles
----------

The name of the new deployment option --- EMT --- gives a good clue that
with this option, many operational aspects of the architecture are
externalized to an orchestration component. Existing users of AMPLIFY
API Management should be aware that with EMT deployment, the role of
Admin Node Manager becomes more of a monitoring tool. And **Node
Manager** is completely removed from the EMT architecture.

Official testing is taking place in Kubernetes as the orchestration
component. However, the Docker images are agnostic, so they can be
deployed in other orchestration platforms, like Swarm. Kubernetes
manages many important aspects of runtime, security, and operations:

-   Autoscaling API Gateway with CPU or memory consumption threshold

-   Self-healing

-   Rolling updates with zero downtime

-   And many more

In generic terms, reference architecture can be built by stacking four
layers of different capabilities:

![](/Images/apim-reference-architectures/container-azure/image1.png){width="6.552083333333333in"
height="3.029861111111111in"}Notice that the packaging of API Management
for deployment has changed in the EMT mode. Customers need to build a
new Docker image for any new FED or POL file that they want to deploy
(see
[documentation](https://docs.axway.com/bundle/APIGateway_77_ContainerGuide_allOS_en_HTML5/page/Content/ContainerTopics/containers_docker_setup.htm)).
We recommend creating a DevOps pipeline that can be triggered anytime a
new configuration is ready for deployment.

To manage Docker images, customers need to set up a Docker registry as a
repository for created images. To help customers with setting up a
required environment, the following table describes the required and
recommended options.

Description                                                                                          Required?
---------------------------------------------------------------------------------------------------- -------------
A DevOps pipeline is strongly recommended for building Docker images                                 Recommended
A storage system with enough capacity to store dedicated data and to share data between components   Required
A bastion for administration tasks on API management and Kubernetes                                  Required
A container registry to store Docker images                                                          Required

[]{#_Toc39071612 .anchor}Table 1: Deployment architecture - global
recommendations

Besides [Docker](https://www.docker.com/resources/what-container) and
[Kubernetes](https://kubernetes.io), we use [Helm](https://helm.sh)
charts to describe the entire deployment configuration. Using Helm
provides an efficient way to package all configuration parameters for
Kubernetes and API Management containers to be deployed with a single
command.

Target use case
---------------

We review a single deployment option where all API Management components
are running inside a single Kubernetes cluster. This pattern allows one
to virtualize back-end apps or APIs hosted inside or outside the
cluster. It is possible to deploy multiple API gateway groups in the
same cluster. But keep in mind that every API gateway group
configuration should be deployed as a separate Docker image.

In our scenario, we expose several entry points for the external clients
inside the cluster (see implementation details in the next chapter). All
API gateways write their transaction events to a shared volume. Admin
Node Manager streams those events to a relational database management
system. API Gateway Manager UI can be used to view the traffic monitor
and events data.

A dedicated deployment environment requires:

-   Kubernetes cluster

-   Docker registry

-   Cassandra cluster

-   RDBMS

-   Storage system for shared volume

-   Monitoring system

-   Bastion system for cluster access

-   DevOps pipeline(s) to control Docker image build and deployment to a
target environment

![](/Images/apim-reference-architectures/container-azure/image2.jpg){width="6.983333333333333in" height="3.35in"}The
following diagram shows a general architecture of a single cluster
configuration:

Additional components and considerations
----------------------------------------

In this section, we present considerations for using additional
components in the single cluster architecture.

### Container registry

This registry is used to store Docker images and Helm packages. You may
consider maintaining at least two pipelines: one for building images and
the other for deploying them to a target environment. The image building
pipeline would build and push images to the registry. The deployment
pipeline would deploy a specific Helm package. In this scenario, Docker
images are pulled by Kubernetes and deployed to a cluster. One of the
requirements in this environment is to securely pass container registry
credentials during Helm package deployment. We suggest using Kubernetes
secrets. Besides registry access credentials, you will need to maintain
additional sensitive data that is described in the following table.

Description                                                                                                              Required?
------------------------------------------------------------------------------------------------------------------------ -------------
Docker images contain such sensitive data as license key, certificate, and configuration. This data must be protected.   Required
The password is sensitive and must be encrypted in the system.                                                           Required
Operations should define a clear tag strategy for Docker images tagging.                                                 Recommended

[]{#_Toc39071613 .anchor}Table 2: Sensitive parameters

### Bastion host

Administrative tasks should be executed safely. We use a bastion host to
bridge to the following instances via the internet:

-   Kubernetes master nodes for managing a cluster

-   Kubernetes Dashboard

-   RBDMS and Cassandra

-   Debugging any issue with a Kubernetes cluster

The bastion must have high traceability with specific RBAC permissions
to allow a few selected users to access infrastructure components.

### DevOps Pipeline

A DevOps pipeline is based on a suite of tools to build and push gateway
configuration and APIs to a target deployment environment and run a set
of verification tests. Since a Docker container is immutable, any change
in API gateway configuration requires building a new Docker image.

Axway provides several CLI tools that fit nicely into a DevOps pipeline.
Later in the document, we will describe them:

-   Docker build scripts

-   API promotion tool (*apimanager-promote*)

-   GitHub projects

### SMTP Relay

The API Manager component can send alerts and messages. By default, the
solution only supports SMPT(S) to send an email. But customization can
be done in alerts policies to send an email by API.

Performance goals
-----------------

An important factor for achieving your goals with the EMT deployment is
to define a set of performance goals. These will be unique for a
specific set of APIs, deployment platform, and clients' expectations.
Later in the document, we show an example of the performance metrics
that have been achieved in testing a reference architecture by the Axway
team.

Implementation details[^1]
==========================

This chapter details the configuration for each component.

Diagram
-------

In this section, we show the main diagrams that represent Axway API
Management architecture deployed on Azure.

### Technical view

The architecture is designed with **High Availability** (HA) in mind. An
HA deployment requires redundancy and high throughput for all
infrastructure components and networks. To reach this target, the
architecture uses most of **Azure PaaS with an appropriate plan** and
**multiple availability zones for VMs**. The components must be deployed
in multiple zones. In our configuration, we use three availability
zones. This configuration is compliant with a minimal technical SLA of
99.99 percent. Each component will be described in the next chapters.

![](/Images/apim-reference-architectures/container-azure/image3.jpg){width="7.0in" height="4.05625in"}

**Note**: For a standard architecture, it's necessary to replace
Availability Zone by **availability Set** configuration. It's a
configuration to isolating VM resources from each other. Axway
recommends setting 3 update domains and 3 fault domains. The bigger
impact is on the Helm chart with affinity node configuration.

### Application view

![](/Images/apim-reference-architectures/container-azure/image4.png){width="7.874305555555556in"
height="4.105555555555555in"}This diagram represents the main objects
inside Kubernetes with main configurations. High availability is also
present in the pod specification.

Choice of runtime infrastructure components
-------------------------------------------

This section provides recommendations for a typical implementation of
the runtime infrastructure components. In this configuration, all assets
of the Kubernetes cluster are deployed in the same data center or a
region (in case of a cloud deployment), although components are spread
out in various racks, rooms, or availability zones.

The following table lists the number of runtime components in this
configuration.

Assets                                             Spec
-------------------------------------------------- -----------------
Worker nodes^\*^                                   3 to 6
Cassandra nodes                                    3
Azure Database for Mysql                           [1]{.underline}
Blob storage^\*^                                   70GB
Azure file storage^\*^                             20GB
External IP (public or private) ^\*^               1 min
Azure Load Balancer or Azure Application Gateway   1
Azure AD                                           1
Azure container registry                           1
Bastion                                            1
Worker pipeline                                    1

[]{#_Toc39071614 .anchor}Table 3: List of assets

^\*\ These\ values\ are\ the\ minimum\ recommended\ starting\ point.\ Your\ actual\ values\ will\ depend\ on\ many\ factors,\ like\ the\ number\ of\ APIs,\ payload\ size,\ etc.^

### Network specification

![](/Images/apim-reference-architectures/container-azure/image5.JPG){width="3.999002624671916in"
height="3.178015091863517in"}This typical network deployment is based on
a minimal number of segregated zones. It consists of 3 subnets in one
Virtual Network for a dedicated VNET:

-   **Bastion subnet** where administration tasks will be done. The
recommended network mask is **/28**.

-   **AKS Subnet** for Kubernetes worker nodes. The recommended network
mask is **/23**.

-   **Data subnet** to host Cassandra Databases. The recommended network
mask is **/27**.

Each subnet must be protected by a firewall lvl4 called Network Security
Group (NSG) with appropriate inbound and outbound rules (see tables
below).

Restrict access from the subnet to only required Azure service, like
Azure Container Registry (need a premium SKU) and Azure Database for
Mysql.

Note: the diagram in Figure 5 shows just one availability zone.

#### Bastion NSG

+----+------------+-----------+------------+----------+-------+
| ID | D          | Direction | To         | Protocol | Ports |
|    | escription |           |            |          |       |
+====+============+===========+============+==========+=======+
| 4  | Usage of   | Outbound  | K8S master | TCP      | 8001  |
|    | Kube-proxy |           | subnet     |          |       |
|    | to access  |           |            |          |       |
|    | the        |           |            |          |       |
|    | Kubernetes |           |            |          |       |
|    | Dashboard  |           |            |          |       |
+----+------------+-----------+------------+----------+-------+
| 5  | Azure      | Outbound  | Azure      | TCP      | 3306  |
|    | Database   |           |            |          |       |
|    | for Mysql  |           |            |          |       |
+----+------------+-----------+------------+----------+-------+
| 6  | Access for | Outbound  | Data       | TCP      | 9042  |
|    | admi       |           | subnet     |          |       |
|    | nistration |           |            |          |       |
|    | database   |           |            |          |       |
|    | tasks      |           |            |          |       |
+----+------------+-----------+------------+----------+-------+
| 8  | Pull (&    | Outbound  | Azure      | TCP      | 443   |
|    | push) Helm |           | Container  |          |       |
|    | package on |           | registry   |          |       |
|    | Docker     |           |            |          |       |
|    | registry   |           |            |          |       |
+----+------------+-----------+------------+----------+-------+
| 12 | Access to  | Inbound   | Bastion    | TCP      | 3389  |
|    | bastion    |           | subnet     |          |       |
|    | hosts      |           |            |          | 22    |
+----+------------+-----------+------------+----------+-------+

[]{#_Toc39071615 .anchor}Table 4: Bastion NSG rules

#### AKS NSG

+----+-----------+-----------+-----------+----------+---------+
| ID | De        | Direction | To        | Protocol | Ports   |
|    | scription |           |           |          |         |
+====+===========+===========+===========+==========+=========+
| 2  | Flow      | Inbound   | AKS       | TCP      | 443     |
|    | between   |           | Subnet    |          |         |
|    | Azure     |           |           |          |         |
|    | Load      |           |           |          |         |
|    | Balancer  |           |           |          |         |
|    | and       |           |           |          |         |
|    | ingress   |           |           |          |         |
|    | c         |           |           |          |         |
|    | ontroller |           |           |          |         |
|    | (see      |           |           |          |         |
|    | section   |           |           |          |         |
|    | 3.2.7     |           |           |          |         |
|    | Azure App |           |           |          |         |
|    | Gateway   |           |           |          |         |
|    | or Azure  |           |           |          |         |
|    | Load      |           |           |          |         |
|    | Balancer) |           |           |          |         |
+----+-----------+-----------+-----------+----------+---------+
| 3  | Co        | Outbound  | Data      | TCP      | 9042    |
|    | nnections |           | subnet    |          |         |
|    | from      |           |           |          | 7070    |
|    | Axway     |           |           |          |         |
|    | c         |           |           |          |         |
|    | omponents |           |           |          |         |
|    | to        |           |           |          |         |
|    | C         |           |           |          |         |
|    | assandra. |           |           |          |         |
|    |           |           |           |          |         |
|    | Also used |           |           |          |         |
|    | for       |           |           |          |         |
|    | P         |           |           |          |         |
|    | rometheus |           |           |          |         |
|    | c         |           |           |          |         |
|    | onnection |           |           |          |         |
|    | to JMX    |           |           |          |         |
|    | exporter  |           |           |          |         |
|    | port      |           |           |          |         |
+----+-----------+-----------+-----------+----------+---------+
| 7  | Egress    | Outbound  | Public    | TCP      | 443     |
|    | flow to   |           | network\  |          |         |
|    | pull      |           | intranet  |          |         |
|    | Docker    |           |           |          |         |
|    | images    |           |           |          |         |
+----+-----------+-----------+-----------+----------+---------+
|    | Egress    | Outbound  | Public    | TCP      | 465-587 |
|    | flow to   |           | network\  |          |         |
|    | send      |           | intranet  |          |         |
|    | email to  |           |           |          |         |
|    | SMTP      |           |           |          |         |
|    | relay     |           |           |          |         |
|    | (re       |           |           |          |         |
|    | presented |           |           |          |         |
|    | by        |           |           |          |         |
|    | SendGrid  |           |           |          |         |
|    | in the    |           |           |          |         |
|    | diagram)  |           |           |          |         |
+----+-----------+-----------+-----------+----------+---------+
|    | C         | Outbound  | Public    | TCP      | 443     |
|    | onnection |           | network\  |          |         |
|    | to        |           | intranet  |          |         |
|    | external  |           |           |          |         |
|    | identity  |           |           |          |         |
|    | access    |           |           |          |         |
|    | m         |           |           |          |         |
|    | anagement |           |           |          |         |
+----+-----------+-----------+-----------+----------+---------+
| 13 | Co        | Outbound  | Azure     | TCP      | 3306    |
|    | nnections |           | Database  |          |         |
|    | from      |           | for Mysql |          |         |
|    | Axway     |           |           |          |         |
|    | c         |           |           |          |         |
|    | omponents |           |           |          |         |
|    | to API    |           |           |          |         |
|    | Analytics |           |           |          |         |
+----+-----------+-----------+-----------+----------+---------+

[]{#_Toc39071616 .anchor}Table 5: AKS NSG rules

#### Data NSG

+----+------------+------------+------------+----------+-------+
| ID | D          | From       | To         | Protocol | Ports |
|    | escription |            |            |          |       |
+====+============+============+============+==========+=======+
| 3  | C          | K8S worker | Data       | TCP      | 9042  |
|    | onnections | subnet     | subnet     |          |       |
|    | from Axway |            |            |          | 3306  |
|    | components |            |            |          |       |
|    | to         |            |            |          |       |
|    | Cassandra  |            |            |          |       |
|    | and API    |            |            |          |       |
|    | Analytics  |            |            |          |       |
+----+------------+------------+------------+----------+-------+
| 6  | Admi       | Bastion    | Data       | TCP      | 9042  |
|    | nistration | subnet     | subnet     |          |       |
|    | tasks from |            |            |          | 3306  |
|    | Bastion to |            |            |          |       |
|    | Databases  |            |            |          | 22    |
+----+------------+------------+------------+----------+-------+
| 11 | Access to  | Outbound   | Frontal    | TCP      | 443   |
|    | the        |            | subnet     |          |       |
|    | admi       |            |            |          |       |
|    | nistration |            |            |          |       |
|    | web        |            |            |          |       |
|    | interface  |            |            |          |       |
|    | (ANM)      |            |            |          |       |
+----+------------+------------+------------+----------+-------+

[]{#_Toc39071617 .anchor}Table 6: Data NSG rules

#### PaaS firewalling configuration

+----+--------------------+--------------------+--------------------+
| ID | Components         | Kind               | Rules              |
+====+====================+====================+====================+
| 1  | Azure App Gateway  | Public network\    | No rules.          |
|    | inbound            | intranet           |                    |
+----+--------------------+--------------------+--------------------+
| 2  | Azure App Gateway  | Backends           | Select the AKS     |
|    | Outbound           | specification      | node pool only.    |
+----+--------------------+--------------------+--------------------+
| 1a | Azure Database for | Private Endpoint   | Allow Mysql        |
|    | Mysql              |                    | connections from   |
|    |                    |                    | **AKS Subnet** and |
|    |                    |                    | **Bastion          |
|    |                    |                    | Subnet.**          |
+----+--------------------+--------------------+--------------------+
| 9  | Shared Files       | Firewalls and      | Allow flow from    |
|    | Premium            | Virtual networks   | the **AKS          |
|    |                    |                    | subnet**.          |
+----+--------------------+--------------------+--------------------+
| 10 | Storage blob       | Firewalls and      | Allow flow from    |
|    |                    | Virtual networks   | the **AKS          |
| 11 |                    |                    | subnet**.          |
+----+--------------------+--------------------+--------------------+
| 7\ | Azure Container    | Firewalls and      | Allow flow from    |
| 8  | Registry           | Virtual networks   | **AKS Subnet** and |
|    |                    |                    | **Bastion          |
|    |                    |                    | Subnet**.          |
|    |                    |                    |                    |
|    |                    |                    | Also, add your     |
|    |                    |                    | **DevOps pipeline  |
|    |                    |                    | worker** for       |
|    |                    |                    | automatic          |
|    |                    |                    | containers         |
|    |                    |                    | creation.          |
+----+--------------------+--------------------+--------------------+

[]{#_Toc39071618 .anchor}Table 7: PaaS rules configuration

### Azure Kubernetes Services (AKS) sizes

![](/Images/apim-reference-architectures/container-azure/image6.png){width="1.9618055555555556in"
height="2.8465277777777778in"}AKS is a platform as a Service managed by
Microsoft. Customers cannot access the Kubernetes master component. They
must use a client like Azure CLI or Azure Portal to configure the
control plane. Using AKS, customers have the advantage of creating a
**secured and scalable cluster** in **less than 5 minutes,** and
**Master nodes are free** and only worker node costs money.

Although it is not possible to provide multiple subnets inside AKS,
customers can define different **node pools** to set a different kind of
VM. For Axway API Management, we define 2 node pools:

-   *apimpool* for all **AMPLIFY Components (API Gateway, API Manager,
ANM)**. This pool must be in autoscaling mode. Node autoscaler
creates a new VM when the control plane receives an alert that pod
cannot be scheduled on any existing node. Once the workload goes
down, and CPU utilization drops below 50% for more than 10 min, this
new node will be deleted. It is possible to change the default
values. Here is the documentation on
[GitHub](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca).
Axway recommends using a VM compliant with **premium storage**. By
default, Axway recommends a **Standard\_DS3\_v2** to support API
gateway upgrade with 3 replicas (default configuration in Helm
chart).

-   *infrapool* for other **auxiliary components** like monitoring
tools, backup, and other tools (maintenance). Axway recommends a
Standard\_D2\_v3 but makes sure that it is enough to support
additional tools.

**Note**: Node pool names are important because they will be used to
make affinity and anti-affinity configuration between both pods and
nodes. They are defined in Helm chart value.

Description                                                                                                                                                   Type
------------------------------------------------------------------------------------------------------------------------------------------------------------- -------------
Node Autoscaler is triggered when a pod fails to be scheduled. Add node scheduling logic to create a new worker node before a cluster exhausts its capacity   Recommended

### Cassandra cluster size

Cassandra is a distributed database, so each node requires its own
storage. For production, Axway recommends a standard Cassandra cluster
on 3 nodes. Axway also recommends a VM with large bandwidth and a
capacity to use premium SSD storage, like Standard\_D4s\_v3. See
[here](https://docs.microsoft.com/en-us/azure/virtual-machines/dv3-dsv3-series?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json)
for more descriptions. Three Cassandra VMs must be spread on 3
availability zones.

A volume of **50GB** must be allocated on each Cassandra node. Axway
recommends using **P6 disks** according to disk capacity. But in some
cases, Axway and Microsoft recommend using **P30 disks** for a higher
flow rate. See
[here](https://azure.microsoft.com/en-us/pricing/details/managed-disks/)
for more information about managed disks.

Description                                      Type
------------------------------------------------ -----------------
Use of Cosmos DB                                 Not recommended
Cassandra nodes spread on 3 availability zones   Recommended

### Azure Database for Mysql

Azure Database for Mysql is a DBaaS managed by Microsoft. ANM uses an
RDBMS to store data. RDBMS uses less storage than Cassandra, so **20GB**
for a simple instance is acceptable.

Axway recommends using Azure Database for Mysql with **4 vCores in
general Purpose mode** and the last generation of CPU. Minimal storage
in this plan is 50 GB with 150 IOPS. See
[here](https://docs.microsoft.com/en-us/azure/mysql/concepts-pricing-tiers)
for a complete description of the vCore and pricing tier.

Description                                     Type
----------------------------------------------- -------------
Use a standard plan with 50GB (for bandwidth)   Recommended
Activate TLS configuration                      Recommended
Use Endpoint termination                        Recommended

### Azure Files Premium (AFP) for shared storage.

#### Events logs

API gateways generate events. A shared volume is required. The volume is
limited by a setting inside the deployment package (FED file). By
default, this value is set to **1GB**. Three gateways are deployed in
the minimal configuration. But autoscaling can increase this number to
12 replicas (or more in your custom configuration), as an example. So,
storage of **13 GB** must be configured.

Please, follow this method to estimate disk space:

> ![](/Images/apim-reference-architectures/container-azure/image8.png){width="3.4730741469816273in"
> height="2.0541666666666667in"}*Max\_disk\_space x
> API\_gateway\_max\_replica + 1GB = recommended\_disk\_space*

It is important to select a proper disk option for logs in your target
environment (cloud or on-premises). In the Kubernetes environment, there
may be many pods **simultaneously writing** to shared storage. Selected
disks must support this target workload.

**Azure Files** is the perfect answer to shared files between pods. It
requires a Storage account. The minimal size is **100Gib**. See the
Security considerations section on p.31 for access restriction.

### Azure Blob Storage for logs

A volume is required to store all logs streamed out from a Kubernetes
cluster. Axway recommends using a premium SSD storage except if you want
geo-replication for data persistence. In this case, Axway recommends a
Local Replication Storage (LRS) or a Zone Redundant Storage (ZRS). The
data will contain:

-   Kubernetes logs

-   Containers logs

-   Applications logs

*FluentD* has been used in our environment to stream logs. A plugin is
available to stream logs to blob storage. In this case, Blob storage has
a dynamic sizing with high limitation. See the Logging/tracing section
on p.31 for more details.

### Azure App Gateway or Azure Load Balancer

A load balancer is required in front of the cluster. We use the
Kubernetes object called **ingress controller** that is responsible for
**fulfilling the ingress rules**. A layer 7 load balancer is configured
in this architecture. It performs important tasks of **terminating TLS
connection** and **request routing**. Two solutions are possible with
this architecture:

+----------------------------------+----------------------------------+
| Azure Application Gateway        | Azure Load Balancer              |
+==================================+==================================+
| Azure Application Gateway has a  | Azure Load Balancer is a         |
| mode for the Ingress controller  | traditional load balancer. An    |
| (AGIC). It is fully managed by   | ingress component is based on    |
| Azure and configuration is the   | Nginx. In this case, the         |
| easiest for the solution. You do | configuration is a little more   |
| not have to manage AGIC pods,    | complex and centralized inside   |
| just configuration inside the    | the cluster. It has a more       |
| cluster. For example, http2 is   | flexible configuration. For      |
| deactivated on the Azure         | example, you can configure the   |
| component and not directly in    | name of a cookie for persistent  |
| the pod. Axway recommends using  | sessions.                        |
| a Standard V2 Tier without       |                                  |
| autoscaling. HTTP2 must be       | This solution does not carry any |
| disabled.                        | additional cost.                 |
|                                  |                                  |
| But AGIC usage is more expensive | ![](/Images/apim-reference-architectures/container-azure/image11.J            |
| because outbound flows are       | PG){width="3.3020833333333335in" |
| billed at \~\$500 for 9TB.       | height="1.9631944444444445in"}   |
|                                  |                                  |
| ![](/Images/apim-reference-architectures/container-azure/image10.J            |                                  |
| PG){width="3.3229166666666665in" |                                  |
| height="1.6076388888888888in"}   |                                  |
+----------------------------------+----------------------------------+

Azure Application Gateway uses Kubernetes services only for endpoint
discovery. Then it routes requests directly to the IP's endpoint. So,
latency is smaller. But Azure Application Gateway has some limitation to
manage certificates. First, it does not allow certificates signed by
private authority. Secondly, a certificate must be composed by appending
a top-level and second-level domains: *domainname.tld*.

API Gateway Manager uses a certificate for internal communication
between components (API Gateway, API Manager) and also for its User
Interface by default. This certificate cannot use a certificate like
*domainname.tld*. For this reason, this certificate cannot be used with
Azure Application Gateway. The simplest way is to provide the second
listener (see section Admin Node Manager on p.27 for detailed
description).

### Azure Container Registry

Azure Container Registry is deployed to store both Helm chart and docker
images. Helm chart is a very small package, but 1GB is required to store
main APIM images (ANM, APIMGR, BASE). Customers need to build a new
docker image for every policy or configuration change in the FED file.
Customers should have an image governance approach to maintain the
required images in the registry. For example, if you need to roll back
the current image (configuration) to one of the previous versions, that
previous version must exist in the container registry.

A **Standard SKU is enough** for space and performance, but Axway
recommends an SKU Premium to restrict access to specific networks such
as the AKS subnet and DevOps worker if needed.

Kubernetes considerations
-------------------------

This section focuses on additional Kubernetes objects and configuration
**inside the cluster** to support Axway components. This is a required
step before deploying containers. A sample Helm chart is available
[here](https://github.com/Axway/apigw-helm-charts). Use it as a starting
point for building your Helm chart.

### Deployment options

There are parameters that you specify at the time of the creation of a
Kubernetes cluster. One of them is a network manager for communication
between pods. The second one is a set of strong permissions for
Kubernetes.

Description                                                                                                                                            Type
------------------------------------------------------------------------------------------------------------------------------------------------------ -------------
Network CNI mode with a specific plugin (CALICO or Cloud provider) to secure pod connections with other applications or resources inside the cluster   Recommended
Secure Kubernetes with RBAC capabilities                                                                                                               Recommended

#### Network plugin

By default, there is no isolation between pods inside the cluster in
Kubernetes. It is not a problem to deploy it on a dedicated cluster;
default Kubernetes networking will be enough. API management
capabilities do not require specific rules. But in some cases, API
management may be deployed with other back end or apps in the same
cluster. In this case, it is necessary to use a Container Networking
Interface (CNI) plugin. In the case of Azure, Axway recommends plugin
**Azure CNI**. This plugin avoids a NAT between network and pods for
**low latency compared to Kubelet**, but each pod will consume an IP in
AKS Subnet. This is the reason why we use a large address range on this
subnet.

Also, it is possible to apply 3 kinds of network policies with CALICO or
Azure Policy Manager (see [this
blog](https://azure.microsoft.com/en-us/blog/integrating-azure-cni-and-calico-a-technical-deep-dive/)):

-   Drop communication by default

-   Allow connection to API gateway from specific namespaces

-   Allow connection to pods from specific monitoring tools

#### RBAC Permission

RBAC permission is a secure mechanism to manage authorization inside
Kubernetes. It's recommended to set people or application permissions to
manage resources:

-   Allow Helm to manage resources

-   Allow worker nodes autoscaling

-   Allow specific users to view pods, to deploy pods, to access
Kubernetes Dashboard

-   Allow cert-manager to pull and encrypt certificate

-   Allow Kubernetes to provide cloud resources, like storage or load
balancer

This is a minimal configuration and you can define more specific
permissions with cluster roles and binding in the cluster.

Also, a Service Principal on Azure is required. Here are the main
permissions:

-   \"Azure Kubernetes Service Cluster Admin Role\",

-   \"AcrPull\" on all container registries where docker images and Helm
charts are stored

-   \"Network Contributor\" on the vnet,

-   Read/Write access to a storage account,

-   \"Managed Identity Operator\" and \"contributor\" for managed
identity \"AAD\_POD\_IDENTITY\" 

Axway recommends 2 services principals:

-   The first one is to build and configure all services

-   The second one is to run a solution in production

### Namespaces

A namespace allows splitting of a Kubernetes cluster into separated
virtual zones. It's possible to configure multiple namespaces that will
be logically isolated from each other. Pods from different namespaces
can communicate with a full DNS pattern
*(\<service-name\>.\<namespace-name\>.svc.cluster.local*). A name is
unique within a namespace, but not across namespaces.

As mentioned in [Kubernetes
documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/),
a typical usage of namespaces is **separating projects and configured
objects deployed by different teams**.

Axway recommends deploying all API management components inside the same
namespace:

-   According to Kubernetes best-practice, the deployment is the
responsibility of API team

-   It's easiest to deploy the solution inside an existing cluster

-   You don't have to specify full DNS to call other components,
therefore, preventing errors. You just use a service name
(*\<service-name\>*).

Description                                               Type
--------------------------------------------------------- -------------
Provide all API management assets in the same namespace   Recommended

### Pod resource limits

Axway API Gateway runs in a **Java VM**. Set the *Xmx* parameter for a
Java VM pool to the limited effect of a potential memory leak (see
section 3.4 on page 23).

Kubernetes permits defining a CPU and memory limits for each pod to
protect the cluster. Setting up the limits is especially important in
case a cluster is shared with other apps. When scheduling a pod
deployment, Kubernetes uses these limits to ensure that resources for a
pod are available on a target node.

Description                                                                        Type
---------------------------------------------------------------------------------- -------------
Limit memory and CPU usage to protect the cluster                                  Recommended
Change Xmx value and resources limitations according to the size of worker nodes   Recommended

These are the recommended initial limits for AMPLIFY API Management
components:

-   API Manager pod: 2cpu with 2GB memory and initial request of 0,5cpu
with 0,5GB memory

-   Admin Node Manager: memory resource limit of 2GB memory

### Components healthcheck

Kubernetes provides a very useful feature called probes. There is one
probe to check if a **pod is ready to be used at startup** and another
one to periodically check if a **container is still operational**. These
probes are respectively called "readiness probe" and "liveness probe."

These probes are configured on all ports (traffic and UI) and use the
HTTP/HTTPS protocol.

Description                                                           Type
--------------------------------------------------------------------- ----------
Implement Kubernetes probes to manage container status in real-time   Required

### Affinity and anti-affinity mode

#### Affinity node

As explained in the last chapter, 2 node pools are created in AKS: one
for APIM and another for tools. It's necessary to make an affinity mode
to avoid a pod to be created in the wrong pool. By default, in AKS each
VM has annotation with the name of node pool where it is configured. All
node pools in AKS have the *agentpool* annotation by default.

*      affinity:*

*        nodeAffinity:*

*          requiredDuringSchedulingIgnoredDuringExecution:*

*            nodeSelectorTerms:*

*            - matchExpressions:*

*              - key: agentpool*

*                operator: In*

*                values:*

*                - apimpool*

#### Anti-affinity pods

Kubernetes pod allocation strategy is based on the nodes' resources
availability. Potentially, more than one pod replica can be deployed on
the same node. For an HA deployment, you want to spread your runtime
components across multiple nodes and availability zones. For this
reason, we recommend using a Kubernetes option called
***podAntiAffinity***. You should instruct Kubernetes [not]{.underline}
to schedule the same replicas on the same node if it is possible based
on the resource availability.

Description                                                                                  Type
-------------------------------------------------------------------------------------------- ----------
Dispatch APIM pods across available nodes (monitoring node can be excluded from this rule)   Required

*        podAntiAffinity:*

*          preferredDuringSchedulingIgnoredDuringExecution:*

*          - weight: 100*

*            podAffinityTerm:*

*              labelSelector:*

*                matchExpressions:*

*                  - key: app*

*                    operator: In*

*                    values:*

*                    - traffic*

### Autoscaling

To properly increase or decrease the number of runtime components to
accommodate a workload, there are two scaling techniques used in the
reference architecture:

-   Nodes/VMs autoscaling

-   Kubernetes pod autoscaling

#### Node scaling

AKS can create new nodes (in different pools) when a pod can't be
scheduled because of insufficient resources (CPU or RAM). For example,
when you perform a rolling update of an API gateway, Kubernetes create
pods from the new docker image before deleting the old ones. For
example, if you run 6 replicas with a maximum of 2CPU per pod,
Kubernetes will need 6 more CPUs to create a new deployment during a
short time. This means that Kubernetes will create 2 new nodes (if using
the recommended VM size) in *akspool* during the deployment. Kubernetes
will delete them after 5 minutes of low activity (under 50% average
usage).

If the allocated number of nodes/VMs is not enough for increasing
traffic, and your API solution may experience an abrupt increase of API
calls, Axway recommends using a platform-provided mechanism to scale
nodes. For Azure, it should be an Azure Automation script based on a
target resource consumption, for example, CPU.

#### Horizontal Pod Autoscaler

Using Kubernetes Horizontal Pod Autoscaler (HPA) you can automatically
scale the number of API Gateway components. HPA uses a control loop that
checks selected utilization metrics every 15s (default value). There are
several options for triggering autoscaling. The average CPU utilization
can be used. It is set to a high enough value for optimal resource
usages. When CPU utilization exceeds this threshold, Kubernetes adds
more pods. You need to test what should be a good CPU utilization based
on your pod start-up time, traffic pattern, and potential impact on the
overall performance. An average CPU utilization of 75 percent is a good
starting point. Keep in mind that to get HPA working, you need to define
resource limits (notice, that our CPU limit is 2cpu, see section 3.3.3
on page 20). This is an example of this setting in Helm:

*metrics:*

*- type: Resource*

*resource:*

*name: cpu*

*target:*

*type: Utilization*

*averageUtilization: 75*

### External traffic

We use the ingress controller to expose API management apps and services
to external clients. It dynamically creates externally reachable URLs
for desirable endpoints, load balance traffic between pods, and
terminate TLS connections. This mode is available only for HTTP and
HTTPS.

Description                                                                               Type
----------------------------------------------------------------------------------------- ----------
Terminate TLS before a request reaches pods                                               Required
Need specific DNS entry to route requests (solution with rewrite path isn't functional)   Required

A specific DNS entry is required to route requests to a service inside a
Kubernetes cluster. These are the exposed interfaces in our
configuration:

-   *anm.FQDN* for API Gateway Manager UI

-   *api-manager.FQDN* for API Manager UI

-   *api.FQDN* for API traffic

DNS records must match the certificate and ingress host configuration.
These records must target the external IP used for the Kubernetes entry
point.

It's necessary to configure the following annotations for ingress
configuration:

-   Disable HTTP/2 if your ingress chooses it by default

-   Authorized entry port (recommended to allow only HTTPS)

-   An HTTPS redirection if HTTPS is authorized with the previous rules

-   Force usage of TLS protocol v1.2

-   Specify HTTPS protocol for a back end

```{=html}
<!-- -->
```
-   Localization of each certificate

-   A range IP authorization (optional, but recommended only for private
access)

Kubernetes lists available ingress controllers
[here](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)
and you can pick any Ingress controller option that fits your
requirements.

The table below describes two options:

+---------------------------+-----------------------------------------+
| Component                 | Specifications                          |
+===========================+=========================================+
| Azure Application Gateway | **kubernetes.io/ing                     |
|                           | ress.class: azure/application-gateway** |
|                           |                                         |
|                           | **appgw.ingress.                        |
|                           | kubernetes.io/backend-protocol: https** |
|                           |                                         |
|                           | **cert-manage                           |
|                           | r.io/cluster-issuer: letsencrypt-prod** |
|                           |                                         |
|                           | **cert-m                                |
|                           | anager.io/acme-challenge-type: http01** |
|                           |                                         |
|                           | **appgw.ingress                         |
|                           | .kubernetes.io/ssl-redirect: \"true\"** |
|                           |                                         |
|                           | **appgw.ingress.kubern                  |
|                           | etes.io/connection-draining: \"true\"** |
|                           |                                         |
|                           | **appgw.ingress.kubernetes.i            |
|                           | o/connection-draining-timeout: \"30\"** |
+---------------------------+-----------------------------------------+
| Nginx                     | kubernetes.io/ingress.class: nginx      |
|                           |                                         |
|                           | cert-mana                               |
|                           | ger.io/cluster-issuer: letsencrypt-prod |
|                           |                                         |
|                           | cert                                    |
|                           | -manager.io/acme-challenge-type: http01 |
|                           |                                         |
|                           | nginx.ingres                            |
|                           | s.kubernetes.io/backend-protocol: HTTPS |
|                           |                                         |
|                           | nginx                                   |
|                           | .ingress.kubernetes.io/affinity: cookie |
|                           |                                         |
|                           | nginx.ingress.                          |
|                           | kubernetes.io/affinity-mode: persistent |
|                           |                                         |
|                           | nginx.ingress.kubernetes.io/session-    |
|                           | cookie-name: \"API-Gateway-Manager-UI\" |
|                           |                                         |
|                           | nginx.ingress.kubernete                 |
|                           | s.io/session-cookie-expires: \"172800\" |
|                           |                                         |
|                           | nginx.ingress.kubernete                 |
|                           | s.io/session-cookie-max-age: \"172800\" |
|                           |                                         |
|                           | nginx.ingress.kubernetes.io/ses         |
|                           | sion-cookie-change-on-failure: \"true\" |
+---------------------------+-----------------------------------------+

[]{#_Toc39071619 .anchor}Table 8: Ingress controller options

Session cookie must be configured only for User Interface (API Gateway
Manager, API Manager).

### Secrets

Without secrets, all passwords are set in clear in manifests. Kubernetes
define "secret" objects to encode in base64 all sensitive information.
Using *secrets* is very useful for variables in containers, Docker
registry login, and technical token for shared storage.

Here is a table to list all the secrets used by pods:

Admin Node Manager   API Gateway Manager   API Gateway Traffic   Ingress controller
----------------------- -------------------- --------------------- --------------------- --------------------------
Public certificate                                                                       X (one for each ingress)
Docker registry login   X                    X                     X
Cassandra user ID                            X                     X
Shared storage ID       X                    X                     X
SGBDR user ID           X

[]{#_Toc39071620 .anchor}Table 9: Kubernetes secrets list

Another solution to store sensitives information is to replace
Kubernetes\' secret by Azure Vault.

API Management implementation details
-------------------------------------

This chapter describes API management components in Kubernetes (see
Figure 4: Kubernetes object deployment).

### Admin Node Manager

Admin Node Manager provides the API Gateway Manager web interface with
monitoring capabilities. This component requires an RDBMS to store
analytical data. It is accessed through an ingress controller. We
recommend that ANM is available from an internal and secure location
(i.e., through bastion node).

To build an ANM container, you need to provide the following:

-   HTTPS certificate. This certificate will be used inside the cluster,
so it's possible to use a self-signed certificate. However, we
recommend reviewing the usage of self-signed certs with your
security team.

-   A license file with the EMT mode. Notice that it is not required to
start Admin Node Manager but may be required for optional features
such as FIPS mode.

-   JDBC library for the selected RDBMS.

-   RDBMS connection parameters.

A FED for Admin Node Manager must be provided with the following
configuration:

-   Second Listener on 8091

-   A certificate signed with public authority and the secret key on
this new listener.

-   Environment variables in Default Database Connection :

-   *\${environment.METRICS\_DB\_URL}*

-   *\${environment.METRICS\_DB\_USERNAME}*

-   *\${environment.METRICS\_DB\_PASS}*

To deploy you need to provide:

-   Heap size memory as mentioned before (set the value at 1024)

-   Persistent storage with read/write multiple pods capabilities to
store events

-   RDBMS parameters

-   Log format in JSON

-   Trace level setting

The Helm chart contains a precheck mechanism (*initContainers*) to check
for required preconditions before this pod can be started. Those checked
preconditions are:

-   Up and running RDBMS server

Pod characteristics:

-   Healthcheck: LivenessProbe and ReadinessProbe

-   Kubernetes object: deployment

-   Resources limit: 1cpu & 2GB memory

-   Java heap size: 1024 MB

-   Automatic scalability: no

-   Replicas: 1

### API Manager UI

This pod supports an API Manager web interface (port 8075).

To build an API Manager container, you need to provide:

-   HTTPS certificate. This certificate will be used inside the cluster.
It's possible to use a self-signed certificate. However, we
recommend reviewing the usage of self-signed certs with your
security team.

-   A license file with the EMT mode set.

-   A group name. It's mandatory because Admin Node Manager needs it to
manage gateway.

-   JDBC library for the selected RDBMS.

-   A FED file with server settings and policies. It is also possible to
use policy and environment package files. So, you can have one
common policy package and several environment packages: one for each
deployment environment.

To deploy you need to provide:

-   Cassandra hosts and Keyspace name

-   Heap size memory as mentioned before (set the value at 1024)

-   ANM hostname

-   RDBMS parameters

-   Log format in JSON

A FED for API Gateway must be provided with following configuration:

-   Certificate signed with a public authority and a secret key on API
Portal's port

-   Add path /healthcheck on API Portal's ports

-   Create new Database connection environment variables in Default
Database Connection:

-   *\${environment.METRICS\_DB\_URL}*

-   *\${environment.METRICS\_DB\_USERNAME}*

-   *\${environment.METRICS\_DB\_PASS}*

-   Environment variables for Cassandra settings:

-   *\${environment.CASS\_HOST}*

-   *\${environment.CASS\_PORT}*

-   *\${environment.CASS\_USERNAME}*

-   *\${environment.CASS\_PASS}*

-   *\${environment.CASS\_KEYSPACE}*

-   *\${environment.CASS\_TKEYSPACE}*

-   Enable SSL and add a client certificate in Cassandra Security.

The Helm chart contains a precheck mechanism (*initContainers*) to check
for required preconditions before an API Gateway pod can be started.
Those checked preconditions are:

-   Up and running three Cassandra hosts

-   Up and running RDBMS server

-   Up and running ANM pod

Pod characteristics:

-   Healthcheck: LivenessProbe and ReadinessProbe

-   Kind: deployment

-   Exposed port UI 8075 (customers may configure a different port if
needed)

-   Resources limits: 2cpu & 2GB memory

-   Java heap size: 1024 MB

-   Session affinity: by cookie

-   Replicas: 1

API Manager UI pod can send an email via SMTP relay. It's configured
inside a FED file.

### API Gateway/Manager (traffic)

API Gateway/Manager pod handles API calls. At the least, a traffic port
(port 8065) should be exposed to this container. But if needed,
additional ports can be exposed (secure and nonsecure ports 8443, 8080
and 8081). This component uses a volume mount point with read/write
multiple pods capabilities to store events. Other data to persist is
streamed out by *FluentD*. With this approach, you can reduce the size
of persistence data required for log/even data.

The API Gateway/Manager docker image is the same as the API Manager UI
image (one image per API Gateway group configuration).

API Gateway configuration can be split into multiple groups. Each group
may support a different set of configuration parameters and policies
(for example, per a business unit):

-   A group is specified as a parameter during a docker image
generation.

-   Each group needs a separate Keyspace.

-   A group API Gateway and API Manager images must have the same FED
configuration.

As you deploy an API Gateway group configuration, the API
Gateway/Manager and API Manager UI containers are created from the same
group image. The deployment parameters are the same. The only
differences are in the Helm chart configuration. These are primarily the
number of replicas and exposed ports.

You have two patterns in terms of a group deployment:

1)  Configure all APIs in one group

2)  Split APIs into multiple groups. With this pattern you will need to:

a.  Generate multiple docker images: one per group

b.  For each group, create required Cassandra Keyspaces (general,
throttling)

> API Gateway Manager UI shows the complete topology: multiple groups,
> each with corresponding gateways.

The Helm chart contains a precheck mechanism (*initContainers*) to check
for required preconditions before this pod can be started. Those checked
preconditions are:

-   Up and running three Cassandra hosts

-   Up and running RDBMS server

-   Up and running ANM pod

-   UP and running API Manager UI pod

A FED for API Gateway must be provided with following configuration:

-   Certificate signed with a public authority and a secret key on API
Manager's port

-   Add path /healthcheck on API Manager's port

-   Create new Database connection environment variables in Default
Database Connection:

-   *\${environment.METRICS\_DB\_URL}*

-   *\${environment.METRICS\_DB\_USERNAME}*

-   *\${environment.METRICS\_DB\_PASS}*

-   Environment variables for Cassandra settings:

-   *\${environment.CASS\_HOST}*

-   *\${environment.CASS\_PORT}*

-   *\${environment.CASS\_USERNAME}*

-   *\${environment.CASS\_PASS}*

-   *\${environment.CASS\_KEYSPACE}*

-   *\${environment.CASS\_TKEYSPACE}*

-   Enable SSL and add the client certificate in Cassandra Security.

Pod characteristics:

-   Healthcheck: LivenessProbe and ReadinessProbe

-   Kind: deployment

-   Exposed traffic port 8065 (customers may configure a different port
if needed)

-   Resources limits: 2cpu & 2GB memory

-   Java heap size: 1512 MB

-   Automatic scalability: yes

-   Replica number: minimum/initial value is 3

Cassandra considerations
------------------------

Cassandra must be hosted outside of the cluster. Minimum three nodes are
required: one node in each availability zone. Please see [the official
documentation](https://docs.axway.com/bundle/APIGateway_77_CassandraGuide_allOS_en_HTML5/page/Content/CassandraTopics/CassandraStartPage.htm)
on the Axway support website.

Cassandra must be in version 2.2.12.

Communication and access to the database must be protected :

-   Use login and password.

-   Configure Mutual authentication with a valid certificate in
Cassandra Security options.

-   Deactivate SSLv3 and TLSv1 protocols.

For a single region configuration, initial replication must be set as
follows:

-   **Keyspace** must have an Initial replication to 3 with a **simple
strategy.**

-   **Throttling** configuration must be configured with an initial
replication to **3** with a **simple strategy**. Read and write
consistency level at **QUORUM**.

A JMX exporter must be configured on each Cassandra node to monitor
Cassandra. The listener is configured with the default port of 7070.

Security considerations
-----------------------

The following is a summary of security considerations discussed within
this document:

-   In layered infrastructure, the network is protected by firewall LVL4
that filters only specifics ports. Any administrative access to
infrastructure components is enabled through a subnet called a
bastion. PaaS services have selected access from AKS or data subnet.

-   All components are deployed in a cluster mode. Kubernetes uses RBAC
permission to set roles per user or group. CNI plugin protects
communications inside the cluster. All external connections are
protected by an ingress controller with a public certificate.

-   In the application layer, sensitive data is protected inside a
secret. All communications are encrypted in a cluster. API
Management uses only TLS 1.2.

-   Do not run containers in a privileged mode.

-   Only allow communication on required ports.

Here additional information on security in AKS:

-   It is possible to limit actions performed by a container or process
with *AppArmor* or *seccomp*, but it is not enabled by default.

-   Microsoft doesn't reboot nodes after patching them. It recommends
*Kured* to reboot them one by one.

-   See Microsoft security recommendations
[here](https://docs.microsoft.com/en-us/azure/aks/operator-best-practices-cluster-security).

-   Axway did not configure *AppArmor* or *seccomp* filters by default,
but it strongly recommends deploying *Kured*.

SQL database considerations
---------------------------

AMPLIFY API Management supports several RDBMS:

-   MySQL or MariaDB

-   Microsoft SQL Server

-   Oracle database

For this guide, Axway didn't test Azure SQL Database because the
supported MySQL version isn't available on Azure.

Logging/tracing
---------------

The following logs should be persisted:

-   Trace files

-   Admin Node Manager: *INSTALL\_DIR/trace*

-   API Gateway instance:
*INSTALL\_DIR/groups/\<group-id\>/\<instance-id\>/trace*

-   Transaction audit log -- see [this
documentation](https://docs.axway.com/bundle/APIGateway_77_AdministratorGuide_allOS_en_HTML5/page/Content/AdminGuideTopics/log_global_settings.htm).

-   Transaction access log -- see [this
documentation](https://docs.axway.com/bundle/APIGateway_77_AdministratorGuide_allOS_en_HTML5/page/Content/AdminGuideTopics/log_access_settings.htm).

-   Transaction event log -- see [this
documentation](https://docs.axway.com/bundle/APIGateway_77_AdministratorGuide_allOS_en_HTML5/page/Content/AdminGuideTopics/log_event_settings.htm)

-   Open traffic event log -- see [this
documentation](https://docs.axway.com/bundle/APIGateway_77_AdministratorGuide_allOS_en_HTML5/page/Content/AdminGuideTopics/log_open_traffic_event_settings.htm)

> **Fluentd** is deployed on *infrapool* nodes in AKS to stream logs.
> Fluentd is deployed on each node using a *Daemonset*. Although Fluentd
> doesn't have an Azure blob connector, a plugin is available on
> [GitHub](https://github.com/Microsoft/fluent-plugin-azure-storage-append-blob).
> It's also possible to use Fluentd with Azure Log Analytics as
> described
> [here](https://github.com/yokawasa/fluent-plugin-azure-loganalytics).

Monitoring
----------

A system is required to monitor the platform and containers inside it.

Axway recommends using the Prometheus server with Grafana Web interface.
These 2 components are deployed in the *infrapool* nodes in AKS.
Prometheus is composed of 3 services:

-   *Server* with a Web interface. This component needs a PVC to store
data.

-   *PushGateway* monitors Kubernetes\' ephemeral objects as Jobs.
Kubernetes jobs are used to deploy APIM.

-   *AlertManager* notifies your central monitoring system or sends
Alert email. In this reference architecture, an only email alert is
possible.

For more details and specific configuration, please read Prometheus
[documentation](https://prometheus.io/docs/introduction/overview/).

Microsoft Azure offers a complete system monitoring for all managed
services (Azure Container Registry, Azure App Gateway, Azure Kubernetes
Services, Virtual Machines). It's based on **Azure monitor coupled with
log analytics** that collects all metrics and stores them in blob
storage.

Also, for Azure Kubernetes Services, Microsoft has developed an extended
feature of Azure monitor to collects containers *stdout*, *stderr,* and
environmental variables.

Please read this official
[document](https://docs.microsoft.com/en-us/azure/azure-monitor/insights/container-insights-agent-config#overview-of-configurable-prometheus-scraping-settings)
for more information.

+---------------------------------------------------------------+-------------+
| Description                                                   | Type        |
+===============================================================+=============+
| Use one of the following solutions:                           | Recommended |
|                                                               |             |
| -   Azure monitor with container analytics and logs analytics |             |
|                                                               |             |
| -   Prometheus with Grafana                                   |             |
+---------------------------------------------------------------+-------------+

Environmentalization and Promotion
----------------------------------

Environmentalization and promotion go hand in hand. AMPLIFY API
Management uses two different artifacts:

1.  Polices -- used by API Gateway

2.  API data files -- used by API Manager

![](/Images/apim-reference-architectures/container-azure/image13.png){width="3.5805555555555557in"
height="1.6208333333333333in"}The section below covers how the polices
are promoted from the development environment to the testing
environment.

Policy promotion steps:

1.  Policy developer edits configuration and deploys it in the
development environment via Policy Studio.

2.  Test the policies in the development environment.

3.  Enable the display of configuration settings that are assigned for
environmentalization in Policy Studio.

4.  Open *Window \> Preferences \> Environmentalization* in the main
menu, and select *Allow environmentalization* *of fields*

5.  Policy developers environmentalize environment-specific settings.
For example:

-   *URL*, *User Name*, and *Password* fields in a Default Database
Connection

-   The *URL* field in a Connect to URL filter

-   *X.509 Certificate* field in an HTTPS interface

-   *URL*, *User Name*, *Password*, and *Signing Key* fields for an LDAP
configuration

6.  Check-in the code to git.

7.  CI pipeline checks out the code from git and uses
[projpack](https://docs.axway.com/bundle/APIGateway_77_PromotionGuide_allOS_en_HTML5/page/Content/DeployPromoGuideTopics/deploy_package_tools.htm)
CLI to create a pol and env package. This can be done in the Policy
Studio. When the active configuration is loaded, select *File \>
Save \> Policy Package* to save the pol file and *Select File \>
Save \> Environment Package* to save the env file.

8.  Policy Developer uses env package to create an environment-specific
artifact (like *test.env* and *prod.env*).

9.  Create a Docker container using pol and env package. See [this
documentation](https://docs.axway.com/bundle/APIGateway_77_ContainerGuide_allOS_en_HTML5/page/Content/ContainerTopics/docker_script_gwimage.htm)
for more info.

Example

*./build\_gw\_image.py \--license=license.lic \--default-cert
\--pol=banking.fed \\\
\--env=test.env \--merge-dir /home/axway/apigateway*

10. Push Docker image to your Docker registry.

11. Deploy to test environment by pulling an environment-related tag
from the Docker registry.

API Manager promotion:

-   API as code. The GitHub project can be used to create and promote an
API across environments.

> <https://github.com/Axway-API-Management-Plus/apimanager-swagger-promote>.

-   Promote API using export and import mechanism - [refer to this
document](https://docs.axway.com/bundle/APIManager_77_APIMgmtGuide_allOS_en_HTML5/page/Content/APIManagementGuideTopics/api_mgmt_promote.htm).

-   Promote API using REST API. Sample implementation -
<https://github.com/Axway-API-Management-Plus/apim-deployment>.

Performance testing
-------------------

![](/Images/apim-reference-architectures/container-azure/image14.png){width="7.745138888888889in"
height="2.9618055555555554in"}The Axway team ran a variety of
performance tests on the reference architecture. These tests were
executed with a testing tool JMeter. Axway recommends deploying the
performance stack in the same VNET and the appropriate location. In our
case, it has a technical separation between 2 platforms without
additional costs of the outbound data transfer.

All connections are encrypted in TLS. Backend is hosted on 2 nodes on
the AKS cluster. The API has multiple methods that correspond to the
payload. It is protected by basic authentication.

### Load test with 60 threads

Message size   Configuration   \# of Nodes   \# of gateways   Minimal Threshold (TPS)   Result (TPS)      Duration (min)   \# of requests   90% Req duration
-------------- --------------- ------------- ---------------- ------------------------- ----------------- ---------------- ---------------- ------------------
1Kb            Passthrough                                    \>1270                    To be completed
API key                                        \>800                     To be completed
OAuth                                          \>745                     To be completed
10kb           Passthrough     3             3                \>1200                    3100              60               11211912         44.57
API key         3             3                \>750                     3370              60               12189645         39.70
OAuth                                          \>670                     To be completed
50kb           API key                                        \>300                     To be completed
100kb          Passthrough                                    \>100                     To be completed

[]{#_Toc39071621 .anchor}Table 10: Performance validation threshold with
60 threads

### Load test with 200 threads

+-------+-------+-------+-------+-------+-------+-------+-------+-------+
| Pa    | Con   | \# of | \# of | Mi    | R     | Dur   | \# of | 90%   |
| yload | figur | Nodes | gat   | nimal | esult | ation | req   | Req   |
|       | ation |       | eways | Thre  | (TPS) | (min) | uests | dur   |
|       |       |       |       | shold |       |       |       | ation |
|       |       |       |       | (TPS) |       |       |       |       |
+=======+=======+=======+=======+=======+=======+=======+=======+=======+
| 1Kb   | P     | 3     |       | \     | To be |       |       |       |
|       | assth |       |       | >1270 | comp  |       |       |       |
|       | rough |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
|       | API   | 3     |       | \>800 | To be |       |       |       |
|       | key   |       |       |       | comp  |       |       |       |
|       |       |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
|       | OAuth | -   3 |       | \>745 | To be |       |       |       |
|       |       |       |       |       | comp  |       |       |       |
|       |       |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
| 10kb  | P     | 3     | 6     | \     | 5760  | 10    |       | 90.48 |
|       | assth |       |       | >1200 |       |       |       |       |
|       | rough |       |       |       |       |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
|       | API   | 3     |       | \>750 | To be |       |       |       |
|       | key   |       |       |       | comp  |       |       |       |
|       |       |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
|       | OAuth | 3     |       | \>670 | To be |       |       |       |
|       |       |       |       |       | comp  |       |       |       |
|       |       |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
| 50kb  | API   | 3     |       | \>300 | To be |       |       |       |
|       | key   |       |       |       | comp  |       |       |       |
|       |       |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+
| 100kb | P     | 3     |       | \>100 | To be |       |       |       |
|       | assth |       |       |       | comp  |       |       |       |
|       | rough |       |       |       | leted |       |       |       |
+-------+-------+-------+-------+-------+-------+-------+-------+-------+

[]{#_Toc39071622 .anchor}Table 11: Performance validation threshold with
200 threads

Maintenance
===========

After you deploy AMPLIFY API Management in production, you will need to
update product configurations (policies and settings) and install fixes
and service packs. This section outlines best practices in maintaining
your installation.

New configurations
------------------

With a shift to a container-based deployment, the notion of pushing API
Gateway/Manager configuration updates directly to the running instances
has changed. Now you need to create a new Docker image that contains the
latest Gateway/Manager configuration. Using [Kubernetes rolling
upgrades](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/),
you deploy a new Docker image to a cluster without interrupting your
request processing. The following guide explains how to create and
deploy a new Docker image:

<https://docs.axway.com/bundle/APIGateway_77_ContainerGuide_allOS_en_HTML5/page/Content/ContainerTopics/container_development.htm>

In general, the process of building AMPLIFY API Management Docker images
for installation can be depicted as in the picture below.

![](/Images/apim-reference-architectures/container-azure/image15.tmp){width="3.3333333333333335in"
height="2.441666666666667in"}Description of the images:

-   A base image includes a base product installation and doesn't change
frequently. You update a base image only when you need to install an
update for the underlying operating system, or when you need to
install a service pack for AMPLIFY API Management or upgrade the
product.

-   An ANM image represents Admin Node Manager, but in the
container-based deployment, it is not used for pushing new
configurations to the gateways. It is primarily a monitoring tool.

-   Gateway or API Manager images contain configuration and settings
(*.pol + .evn* , or *.fed*) that are updated frequently. These are
the type of images that you would rebuild frequently.

Axway provides sample build scripts as a working example to be modified
and used by customers. The sample scripts are provided from the Axway
support site. For example, a download file for AMPLIFY API Management
v7.7 is:

*APIGateway\_7.7-1\_DockerScripts.tar.gz*

To streamline this process for building Docker images, Axway recommends
creating a CI/CD pipeline that should, at least, include these tasks:

-   Creating a policy package (.pol)

-   Creating an environment package (.env) for every target deployment
environment

-   Creating an environment-specific Docker image; for example, for a
test, QA, or production

-   Deploying a new Docker image to a target environment and smoke
testing

Customers should use a source control management (SCM) system for
maintaining/versioning policy and environment packages and a Docker
registry for Docker images.

Product updates
---------------

There are variations in how to install a new patch, service pack, or
upgrade for a new AMPLIFY API Management version. But the overall
approach is that:

-   For an upgrade or SP installation, you rebuild your base, ANM and
Gateway/Manager images

-   For a patch installation, you rebuild your ANM and Gateway/Manager
images

### Installing a patch

Patch installation requires updating one or more files in the target
product installation directory. To apply a patch, you need to overwrite
one or more files when you build an ANM or Gateway/Manager images. The
build scripts provide the *--merge-dir* option that allows you to
overwrite any file(s) in a Docker image. The example and additional
explanation of the procedure can be found in [this
section](https://docs.axway.com/bundle/APIGateway_77_ContainerGuide_allOS_en_HTML5/page/Content/ContainerTopics/container_patch_sp.htm).
The following is an example of a build command for a Gateway/Manager
image:

*\
./build\_gw\_image.py*

*\--license=/tmp/api\_gw.lic*

*\--domain-cert=certs/mydomain/mydomain-cert.pem*

*\--domain-key=certs/mydomain/mydomain-key.pem*

*\--domain-key-pass-file=/tmp/pass.txt*

*\--parent-image=my-base:latest*

*\--fed=my-group-fed.fed \--fed-pass-file=/tmp/my-group-fedpass.txt*

*\--group-id=my-group*

***\--merge-dir=/tmp/apigateway***

![](/Images/apim-reference-architectures/container-azure/image16.png){width="6.138888888888889in"
height="1.475in"}The *--merge-dir* points to a directory that contains
the file(s) that will replace specific installation files on a target
Docker image. A merge directory must be named ***apigateway***. For
example, if you need to update the *envSettings.props* file in a Gateway
image, this is what you should have in your merge directory:

The */tmp/apigateway* directory should mirror the file path in your
target Docker image.

#### Step-by-step example

Let's look at applying one of the patches for APIM v7.7 -- *APIGateway
7.7-SP1 Patch17276*:

1.  Download a patch from the support website

2.  Create a merge directory:

*mkdir /tmp/apigateway*

3.  Extract downloaded file into the merge directory:

> *tar -xvzf
> APIGateway\_7.7-SP1\_Patch17276\_d16e79fb\_allOS\_BN20191024.tgz -C
> /tmp/apigateway/*

4.  ![](/Images/apim-reference-architectures/container-azure/image17.png){width="5.743055555555555in"
height="1.1111111111111112in"}If you look at the merge directory,
you will see that two files will be written to a target Docker image
during build:

5.  Now you can run your build script specifying the merge directory
that you've created:

*./build\_gw\_image.py*

*\--license=/tmp/api\_gw.lic*

*\--domain-cert=certs/mydomain/mydomain-cert.pem*

*\--domain-key=certs/mydomain/mydomain-key.pem*

*\--domain-key-pass-file=/tmp/pass.txt*

*\--parent-image=my-base:latest*

*\--fed=my-group-fed.fed \--fed-pass-file=/tmp/my-group-fedpass.txt*

*\--group-id=my-group*

***\--merge-dir=/tmp/apigateway***

*\--out-image=my-gtw:7.7-SP1-p17276*

6.  Two files from the patch will overwrite (or create) corresponding
files in the new Docker image.

### Installing a service pack

Installing a service pack is identical to creating your first API
Gateway/Manager Docker image. The only difference is that you just need
to download and use a combined installation file that includes base
product plus a corresponding service pack. As an example, we look at
AMPLIFY API Management v7.7 and API Management v7.7 with SP1install
files:

-   API Management v7.7 install is titled: *API Gateway and API Manager
7.7 Install (linux-x86-64)* with the following file -
*APIGateway\_7.7\_Install\_linux-x86-64\_BN4.run*

-   API Management v7.7 with SP1install is titled: *API Gateway 7.7
Install Service Pack 1 (linux-x86-64)* with the following file -
*APIGateway\_7.7\_SP1\_linux-x86-64\_BN201908271.run*

The build process will be identical to the one described in section 4.1
New configurations.

### Upgrading the product

Upgrading your existing deployment to a new version of AMPLIFY API
Management will be similar to a process described in section *4.1* *New
configurations*. But there are some additional steps to migrate your
existing Gateway/Manager configuration using Policy Studio. The
[following
section](https://docs.axway.com/bundle/APIGateway_77_ContainerGuide_allOS_en_HTML5/page/Content/ContainerTopics/container_upgrade.htm)
in the documentation describes this process for API Management v7.7. The
extra steps are needed to import your existing .fed file in Policy
Studio. This will trigger an automatic update to your FED file. When
done, export the updated FED (or *.pol* and *.env*) file and use it for
building a new Docker image(s).

Notice, that there may be additional upgrade steps related to other
parts of your deployment (e.g. Cassandra or RDBMS).

### Adding customization

You may want to add or customize some files in the product installation
directory. To do this with a container-based deployment, follow steps
outlined in the section 4.2.1 Installing a patch. The only difference is
the need to create manually a proper directory structure of your merge
directory.

Pushing a new Docker image to your Kubernetes cluster
-----------------------------------------------------

When there is time to push a new image to a Kubernetes cluster (in any
environment), we rely on Kubernetes rolling updates that are supported
under the **Deployment** object. This object has a section that governs
the process of updating running containers with a new Docker image. This
is an example of an update strategy specification:

*strategy:*

*type: RollingUpdate*

*rollingUpdate:*

*maxSurge: 1*

*maxUnavailable: 0*

With the rolling update, Kubernetes will take down a designated number
of pods, update them with a new image, and start. Then continue this
process with other pods. The optional *maxUnavailable* parameter
specifies that only one pod at a time can be updated. Instead of an
absolute value for *maxUnavailable,* you can provide a percentage value
that will represent a portion of your Deployment pods that can be
updated at once. With four pods in Deployment set and *maxUnavailable:
25%* (the default value for *maxUnavailable* if it is omitted),
Kubernetes will update one pod at a time.

There is another optional parameter -- *maxSurge* (default value is 25
percent). For example, if there are four pods in your Deployment, then
during a rolling update, the maximum number of pods with old and new
configurations combined can't be more than 5:

> *4 pods (100%) + 1 pod (25% - maxSurge) = 5 pods (125%)*

You can read the [following
articles](https://kubernetes.io/blog/2018/04/30/zero-downtime-deployment-kubernetes-jenkins/)
to better understand rolling updates and other techniques for a zero
downtime deployment.

Configuration and data backups
==============================

It is essential for the smooth operation of an API management solution
to perform regular backups of data and configurations. At a minimum, the
following data, source code and artifacts should be included:

-   Policies and server configuration -- there are a couple of ways to
maintain your deployment artifacts. You can maintain an API Gateway
configuration as a Policy Studio folder in a source code management
system (SCM). GitHub and GitLab are two popular choices. As an
alternative, you can version control FED or POL/ENV files.

In addition to the source code, you need to maintain:

-   Deployment artifacts in the form of a Docker image, since this is
what you deploy in the EMT mode. Thus, a Docker registry is
required.

-   RDBMS backup - use the appropriate backup procedure for the selected
RDBMS. Azure Database for Mysql has a native feature to save data
and the transaction logs. By default, Azure schedules one full
backup a week and 2 incremental backups per day. Axway recommends
selecting geo-redundant backup storage.

-   Cassandra -- follow recommendations in the official documentation
for the Cassandra backup (see
[this](https://docs.axway.com/bundle/APIGateway_77_CassandraGuide_allOS_en_HTML5/page/Content/CassandraTopics/cassandra_BUR.html)).
You will need to decide how frequently you should back up Cassandra.
This decision will be impacted by what data you store in Cassandra.
Is it only API Manager data? Or does it include OAuth tokens and
custom KPS? The main discussion point is to decide how much data you
could potentially lose without seriously affecting your business. A
daily backup may be a good starting point. There are a couple of
good blogs on backing up and restoring Cassandra:

```{=html}
<!-- -->
```
-   [Azure recovery service to backup VM with policy rules. Data
retention is 2 weeks by
default.](https://docs.microsoft.com/fr-fr/azure/backup/tutorial-backup-vm-at-scale)

-   [Medusa - Spotify's Apache Cassandra backup tool is now open
source](https://thelastpickle.com/blog/2019/11/05/cassandra-medusa-backup-tool-is-open-source.html)

```{=html}
<!-- -->
```
-   Log/trace/even files -- Azure file premium is an Azure backup
solution. Axway recommends a geo-redundant backup with a default
history of 14 days.

-   Helm charts and other environment-specific files should be part of
an SCM with its own backup mechanism. You can use the Azure
Container registry (geo-replication is enabled by default) to store
Helm chart packages.

Disaster recovery
=================

For a disaster recovery procedure, you should have access to cloud
resources in another region. Using backed-up configurations/data and
Docker registry, you should be able to run a CI/CD pipeline for creating
a new Kubernetes cluster in another region.

A complete deployment/restoration takes 1hours.

Known constraints and roadmap
=============================

As of AMPLIFY API Management v7.7, there are some differences or
constraints compared to the classic mode deployment:

-   API Portal and Embedded Analytics are not yet supported in the EMT
mode. They should be deployed outside of a Kubernetes cluster

-   Distributed Ehcache is not supported. However, you can use Apache
Cassandra as a distributed data store where CRUD operations are
supported to directly interact with KPS, using scripts.

-   Running with Embedded ActiveMQ configured in an instance is not
advised. The recommendation is to use a JMS provider deployed
outside of the API Management cluster.

The current version does not support all the configurations that
currently exist for the classic deployment. Axway plans to address these
limitations by delivering the following capabilities in the future:

-   Broader support of current APIM configurations

-   Running API Portal in a Docker container

-   Native support of Embedded Analytics

-   Advancements in the current EMT mode

-   Mutable policy configuration - that enables a policy to be
modified without the need to bake in a new Docker image

-   Reference architecture document for other cloud providers

You can view all roadmaps on Axway Community Portal
[here](https://community.axway.com/s/product-roadmaps).

Appendix A -- Glossary of Terms
===============================

Reference   Description
----------- ----------------------------------------------------------------------------------------------
SAML        Security Assertion Markup Language -- based on XML, this protocol permits SSO authentication
SSO         Single Sign-On -- unique authentication for a user
K8S         Short name of Kubernetes product
FED         The file extension of the federated deployment package (policy + environment packages)
POL         The file extension of a policy package file
ENV         A file extension of an environment package file
VM          Virtual Machine
PaaS        Platform as a Service
HA          High Availability
RDBMS       Relational Database Management System
DBaaS       Database as a Service
IOPS
NSG         Network Security Group

[^1]: Axway publishes some development assets on GitHub. You can find
them
[here](https://github.com/Axway/Cloud-Automation/tree/master/APIM).