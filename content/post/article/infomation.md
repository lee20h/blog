---
title: "devops 읽어볼 정보"
date: 2023-09-05T01:02:14+09:00
tags:
  - devops
categories:
  - article
published: true
---

The objective is to document findings and recommendations from the review of DevOps processes and artifacts.

Table of Contents
=================

* [Istio Recommendations](#istio-recommendations)
    * [Telemetry with Stackdriver](#telemetry-with-stackdriver)
    * [Telemetry with Kiali](#telemtry-with-kiali)
    * [Canary Deployment](#canary-deployment)
    * [Mesh-wide security options](#mesh-wide-security-options)
    * [Network security logging](#network-security-logging)
    * [Managed Istio CSM(Cloud Service Mesh) in the near future](#managed-istio-csmcloud-service-mesh-in-the-near-future)
    * [istio-operator for OSS Istio](#istio-operator-for-oss-istio)
* [YAML Recommendations](#yaml-recommendations)
* [Helm Recommendations](#helm-recommendations)
    * [Developing chart](#developing-chart)
        * [References](#references)
    * [Security](#security)
    * [Terraform](#terraform)
        * [Refactor](#refactor)
        * [Production grade](#production-grade)
    * [Manage GCP resources with Kubernetes CRD](#manage-gcp-resources-with-kubernetes-crd)
    * [References](#references-1)
* [Terraform](#terraform-1)
    * [Review](#review)
    * [Recommendations](#recommendations)
* [Network Security](#network-security)
    * [Review](#review-1)
    * [Recommendations](#recommendations-1)
* [GKE](#gke)
    * [Review](#review-2)
    * [Recommendations](#recommendations-2)
        * [GKE Networking](#gke-networking)
        * [GKE Security](#gke-security)
    * [References](#references-2)
## Istio Recommendations
options for Istio on GKE are: GKE with Istio addon, OSS Istio, CSM(Cloud Service Mesh) in the near future.

### Telemetry with Stackdriver
[This example](https://github.com/GoogleCloudPlatform/istio-samples/tree/master/istio-stackdriver) demonstrates the ways you can use Stackdriver to gain insights into and debug microservices deployments running on GKE with Istio's telemetry support: monitoring, tracing and logging. It is by default and configurable via *Rule* in GKE with Istio addon. It can be enabled with OSS Istio as well.

### Telemetry with Kiali
[Installing Istio on a GKE cluster](https://cloud.google.com/istio/docs/how-to/installing-oss) shows how to install a version of istio on a GKE cluster. Enable the stackdriver metrics if desired.
Furthermore, prometheus, grafana, tracing withJaeger and kiali can be installed.
Service toplogy with [kiali](https://istio.io/docs/tasks/telemetry/kiali/) provides answers to the question: What microservices are part of my Istio service mesh and how are they connected?

### Canary Deployment
[This example](https://github.com/GoogleCloudPlatform/istio-samples/tree/master/istio-canary-gke) demonstrates how to use *VirtualService* and *DestinationRule* to implement carnary deployment.
Furthermore, [Istio by example](https://istiobyexample.dev) shows lots of use cases for traffic managment & security.

### Mesh-wide security options
Strict mTLS mode is recommended for Istio on GKE. Otherwise for the Permissive mTLS(MTLS_PERMISSIVE) mode, [introduction of Istio](https://github.com/GoogleCloudPlatform/istio-samples/tree/master/security-intro) shows an example incrementally adopting Istio mutual TLS authentication across the service mesh. Configure mTLS with *VirtualService* and *Policy* for both sides of TLS.

### Network security logging
* [Palo Alto Networks Security adapter for Istio](https://github.com/PaloAltoNetworks/istio)
    * full visibility into container activity from a network perspective for container workloads running on Kubernetes environments with Istio.
    * Ability to use the logs and information to perform forensics to understand exploits and vulnerabilities.

### Managed Istio CSM(Cloud Service Mesh)
Istio plays a key role in Anthos's CSM. This is still in alpha and it would be nice to be aware of the longer term offering from GCP on service mesh, then plan accordingly.
Microservices architectures present a range of benefits, but they introduce many challenges. Google Cloud Service Mesh provides a fully managed platform that simplifies operating services across the board, from traffic management and mesh telemetry to securing communications between services, thereby taking a significant burden off your operations and development teams. For example, Traffic Director is a Google managed Pilot.

For a demo, please check out [Understanding SLOs and Error Budgets With Istio (Cloud Next '19)](https://www.youtube.com/watch?v=AKh8uuVCpFI&t=920s)

### istio-operator for OSS Istio
If you choose to use OSS Istio, [Istio operator](https://github.com/banzaicloud/istio-operator/) can be a great aid to automate and simplify these and enable popular service mesh use cases (multi cluster federation, canary releases, resource reconciliation, etc) by introducing easy higher level abstractions. It can be enabled to send metrics to the Stackdriver.

## YAML Recommendations
* Can use VSCode as text editor, plus Red Hat’s YAML Support plugin as it is very handy for validation, autocompletion, and formatting.
* An interesting idea to customize the helm chart via [Kustomize](https://github.com/kubernetes-sigs/kustomize) tool. Kustomize is a project that came out of the CLI Special interest group. Kustomize lets you customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is. This can be leveraged by solving customization of upstream Helm charts without PRs. [Customizing Upstream Helm Charts with Kustomize](https://testingclouds.wordpress.com/2018/07/20/844/) shows pro and con side on using Helm with Kustomize.


## Helm Recommendations

### Developing chart
* Please check out the following official guides
    * [Charts best practice](https://helm.sh/docs/chart_best_practices/) as it the official best practice for structing charts.
    * [Tips and Tricks](https://helm.sh/docs/developing_charts/#chart-development-tips-and-tricks) has lots of useful tips for developing charts.
    * [Chart template developer's guide](https://helm.sh/docs/chart_template_guide/)

* Always use *helm create* for a new chart because it is always up to date with the recommended practices.
* *helm lint* your chart including custom values. It is a good practice to lint your charts before trying to install them. The linting will apply the templating and verify that the output is a well formatted yaml. A good practice would be to create a ci folder on the same level as your templates one and put there the additional values files you want to verify. So if you have a file called ingress-enabled-values.yaml in your ci folder just run `helm lint --values ci/ingress-enables-values.yaml`.
* Use dry-run and debug to see what the chart will install, e.g. `helm install stable/postgresql --name standalone --dry-run --debug`
* *helm test*:Write funcitonal test according to [Chart Tests](https://github.com/helm/helm/blob/master/docs/chart_tests.md). It is implemnented via [helm lifecycle hooks](https://github.com/helm/helm/blob/master/docs/charts_hooks.md). For example we can verify the network endpoint or database credentials.

* [Customizing Upstream Helm Charts with Kustomize](https://testingclouds.wordpress.com/2018/07/20/844/) shows pro and con side on using Helm with Kustomize.


#### References
* [Helm Chart Patterns - Vic Iglesias, Google](https://www.youtube.com/watch?v=WugC_mbbiWU&t=356s) is a pretty awesome talk on Helm Charts.
* [Charts best practice](https://helm.sh/docs/chart_best_practices/)
* [Helm Documentation](https://github.com/helm/helm/tree/master/docs)

### Security
https://blog.ropnop.com/attacking-default-installs-of-helm-on-kubernetes/ walks through how an attacker who compromises a running pod could abuse the lack of security controls to completely take over the cluster and become full admin.

* [Securing your Helm Installation](https://github.com/helm/helm/blob/master/docs/securing_installation.md#best-practices-for-securing-helm-and-tiller) captures the current best practice
* [how to create strong SSL/TLS connections between Helm and Tiller.](https://github.com/helm/helm/blob/master/docs/tiller_ssl.md)
* [Install Secure Helm In Google Kubernetes Engine (GKE)](https://medium.com/google-cloud/install-secure-helm-in-gke-254d520061f7) illustrates with a real example for the best practice above.

* Another option is not to use Tiller. For example,[how to install istio without Tiller ](https://istio.io/docs/setup/kubernetes/install/helm/#option-1-install-with-helm-via-helm-template) is feasible with `helm template`.

### Terraform

#### Refactor
`
resource "null_resource" "kubernetes-ready" {
triggers {
dependency_id = "${var.cluster_api_endpoint}"
}
}`
can be added to `terraform-helm-tiller` module so that tiller deployment happens after the GKE cluster is ready. Hence there is no need for callers of the module to check dependency.

#### Production grade
[Deploying a production-grade Helm release on GKE with Terraform](https://cloud.google.com/blog/products/devops-sre/deploying-a-production-grade-helm-release-on-gke-with-terraform) from Gruntwork.

### Manage GCP resources with Kubernetes CRD
Ever wonder if we can manage GCP resources in Helm charts just like istio, knative etc?
[Config Connector](https://cloud.google.com/config-connector/docs/overview) is a Kubernetes addon that allows you to manage your Google Cloud Platform (GCP) resources through Kubernetes configuration such as CloudSQL, Cloud memory store, GCS and GKE. This enables a consistent tooling for resources. I think it is supplemental to Terraform.

### References
* [Exploring the Security of Helm](https://engineering.bitnami.com/articles/helm-security.html)
* [running-helm-in-production](https://engineering.bitnami.com/articles/running-helm-in-production.html)
* [Deploying a production-grade Helm release on GKE with Terraform](https://cloud.google.com/blog/products/devops-sre/deploying-a-production-grade-helm-release-on-gke-with-terraform)
* [Helm Chart Patterns - Vic Iglesias, Google](https://www.youtube.com/watch?v=WugC_mbbiWU&t=356s) is a pretty awesome talk on Helm Charts.
* [Charts best practice](https://helm.sh/docs/chart_best_practices/)
* [Helm Documentation](https://github.com/helm/helm/tree/master/docs)


## Terraform

### Review
Pretty solid design and follow best practices for Terraform in general
1. Clean seperation of remote state by application and environment.
2. Clean seperation of git repository and folder by application and environment.
3. Code reuse with modules on GCP services such as GCP project creation, GKE, CloudSQL, memorystore,filestore etc.
4. Don’t Repeat Yourself (DRY) by reusing `terraform-google-modules` git repo as much as possible.
5. `null_resource` technique is used for dependency management between modules.
6. Versions are pinned for various providers and modules.

### Recommendations
1. [Remote state as data](https://www.terraform.io/docs/providers/terraform/d/remote_state.html) can be used in *terraform-gcp-projects-admin/environments* to refer the *parent_folder_id* dynamically by adding output variables in *terraform-gcp-projects-admin/master-hierarchy* for GCP folder ids. [Mentioned here as well](https://github.com/ozbillwang/terraform-best-practices#retrieve-state-meta-data-from-a-remote-backend)
2. Test as code: [Kitchen-Terraform](https://github.com/newcontext-oss/kitchen-terraform) enables verification of Terraform state with InSpec.
3. Nice to have some DevSecOps? [Terraform Validator](https://github.com/GoogleCloudPlatform/terraform-validator) can be used to validate terraform plans before they are applied. Validations are ran using Forseti Config Validator. [Forseti Config Validator Efforts](https://forsetisecurity.org/news/2019/04/10/config-validator.html) describes how Terraform validator works with Forseti. [Forseti Config Validator in GCP](https://cloud.google.com/blog/products/identity-security/protecting-your-gcp-infrastructure-at-scale-with-forseti-config-validator)


## Network Security
[Defense-in-depth strategy](https://cloud.google.com/blog/products/networking/google-cloud-networking-in-depth-three-defense-in-depth-principles-for-securing-your-environment) is enabled in GCP with a comprehensive portfolio of security controls.

### Review
* Deploy your VMs with only private IPs. Even better this can be enforced with an Org policy.
* Access Google managed services and GCP managed services (CloudSQL,MemoryStore,Filestore) privately.
* Provide secure outbound internet connections with Cloud NAT.

### Recommendations
* [Define an organization policy](https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations#organization-policy) per project, per folder, or per organization.
    * You can define a constraint to restrict virtual machine instances from having an external IP address
    * You can restrict the set of identities that are allowed to be used in Cloud Identity and Access Management policies per folder or per project. For example, restrict a folder "engineer" for your domain only users while adding contractor outside under another folder.
* By deploying Google *Cloud Armor* security policies, you can block malicious or otherwise unwanted traffic at the edge of Google’s network, far upstream from your infrastructure. Use preconfigured WAF rules to protect against the most common application vulnerabilities like Cross-site Scripting (XSS) and SQL injection (SQLi).
* *Cloud Identity-Aware Proxy (IAP)*:  you can permit access for authorized users to applications over the internet based on their identity and other contexts without requiring them to connect to a VPN.
* *VPC Service Controls*: tl;dr Mitigate exfiltration risks by preventing your data from moving outside the boundaries of a trusted perimeter. VPC Service Controls allows you to build a trusted private perimeter and ensure that data access is not allowed outside the boundaries of that perimeter. Similarly, the data can't move outside of the perimeter boundaries, mitigating exfiltration risks.
* The *Web Security Scanner* identifies security vulnerabilities in your bGoogle Kubernetes Engine web applications. It crawls your application, following all links within the scope of your starting URLs, and attempts to exercise as many user inputs and event handlers as possible.
* [Forseti](https://opensource.google.com/projects/forseti-security) is used by Spotify to create a notification pipeline that proactively informs us about risky misconfigurations in GCP. [Read the story](https://cloud.google.com/blog/products/gcp/with-forseti-spotify-and-google-release-gcp-security-tools-to-open-source-community15)

## GKE
### Review
1. *Regional cluster* provides the benefits of resilience from single zone failure as well as zero downtime master upgrades, master resize, and reduced downtime from master failures.
2. Node auto repair is enabled.
3. Node auto upgrade is enabled. Keeping the version of Kubernetes up to date is one of the simplest things you can do to improve your security.
4. Use custom node pool by removing the defaut node pool and create new ones with custom machine types.
5. Cluster autoscaler and horizontal pod autoscaling are enabled.
6. Daily maintenance window is specified.
7. Stackdriver Kubernetes Engine Monitoring and Prometheus are used instead of the legacy one.
8. Istio is used.
9. kubernetes dashboard is disabled.

### Recommendations
Here are some relevant best practices on GKE. There are many more can be found in the reference section.

#### GKE Networking
There are overlapping functionalities between istio and GKE such as ingress and ingress-gateway. The purpose here is to list what GCP offers as managed servcies as an option.

1. [Google managed TLS certificates](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs) When you configure an HTTP(S) load balancer through Ingress, you can configure the load balancer to present up to ten TLS certificates to the client. It is created via a [manifest](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs#setting_up_the_managed_certificate).
2. For GKE ingress traffic via HTTP(S) Load Balancer
   GCP HTTP(S) Load Balancing is implemented at the edge of Google's network in Google's points of presence around the world.
* *Cloud Armor* enforces web application firewall (WAF) policies at the edge  [How to use a BackendConfig custom resource to configure Google Cloud Armor in Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine/docs/how-to/cloud-armor-backendconfig).
* *Cloud CDN*  similar to Cloud Armor, [How to use a backendConfig custom resource](https://cloud.google.com/kubernetes-engine/docs/how-to/cdn-backendconfig#creating_a_backendconfig) is used.
* *Container-native load balancing* optimizes the HTTP(S) load balancers performance. Container-native load balancing uses a data model called network endpoint groups (NEGs), which are collections of network endpoints represented by IP-port pairs.
    * It bypasses the *kube_proxy* and *iptables*, this solves the "double hop" problem shown in the [Pods are first-class citizens for load balancing](https://cloud.google.com/blog/products/containers-kubernetes/introducing-container-native-load-balancing-on-google-kubernetes-engine)
    * It reduces the cost for egress traffic across zones in a regional cluster, which priced at 0.01 per GB.
    * The NEG data model  is required to use Traffic Director, GCP's fully managed traffic control plane for service mesh.
* [Managed Service Mesh: Traffic Director vs Istio](https://medium.com/cloudzone/google-clouds-traffic-director-what-is-it-and-how-is-it-related-to-the-istio-service-mesh-c199acc64a6d)

#### GKE Security
1. Securing GKE cluster by either whitelisting via MAN(master authorized network) for a public cluster or using a private GKE cluster completely. In the case of private GKE cluster, private Google access is enabled by default so a VPC peering is created between your own VPC which hosts the cluster nodes and Google-owned VPC which hosts the master nodes. For setting up the network path from office and home to the private API endpoint, one option is  `kubectl exec` via VPN, kubectl can use the internal-ip based Kubernetes context pointing to the private API endpoint. The other option is to create a private compute instance in the same VPC of the GKE cluster, so the instance can SSH forwarding via Cloud IAP(Identify Aware Proxy) to the private API endpoint. When a user runs SSH from the gcloud command-line tool, SSH traffic is tunneled over a TLS connection to Cloud IAP, which applies any relevant context-aware access policies. If access is allowed, the tunneled SSH traffic is transparently forwarded to the VM instance.
2. [Use Least Privilege Service Accounts for your Nodes](https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster#use_least_privilege_sa). By default the compute engine default servic account is used, which has project editor role granted. In the terraform, [set the service_account as "create"](https://github.com/terraform-google-modules/terraform-google-kubernetes-engine/blob/v3.0.0/modules/beta-public-cluster/variables.tf#L246) can create the node service account with least privileges, granting more privileges if needed. Alternativley, [Reduce your Node Service Account Scopes](https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster#reduce_node_sa_scopes) if you don't want to use a custom service account.

3. One of the key security concerns for running Kubernetes clusters is knowing what container images are running inside each pod and being able to account for their origin. With *Binary Authorization*, you can ensure that internal processes that safeguard the quality and integrity of your software have successfully completed before an application is deployed to your production environment.[GKE Binary Authorization Demo](https://github.com/GoogleCloudPlatform/gke-binary-auth-demo) can be a starting point.
4. [Use vulnerability scanning in Container Registry](https://cloud.google.com/solutions/best-practices-for-building-containers#use-vulnerability-scanning)
5. [Restrict Pod Permissions with a Pod Security Policy](https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster#restrict_pod_permissions). By default, Pods in Kubernetes can operate with capabilities beyond what they require. You should constrain the Pod's capabilities to only those required for that workload. Kubernetes offers controls for restricting your Pods to execute with only their minimum required capabilities.
6. [Authenticating to Cloud Platform with Service Accounts](https://cloud.google.com/kubernetes-engine/docs/tutorials/authenticating-to-cloud-platform). The recommended way to authenticate to Google Cloud Platform services from applications running on GKE is to create your own service accounts. Ideally you must create a new service account for each application that makes requests to Cloud Platform APIs.



### References
* https://cloud.google.com/solutions/prep-kubernetes-engine-for-prod
* https://cloud.google.com/blog/products/networking/google-cloud-networking-in-depth-three-defense-in-depth-principles-for-securing-your-environment
* https://cloud.google.com/kubernetes-engine/docs/how-to/hardening-your-cluster
* https://cloud.google.com/solutions/best-practices-for-operating-containers
* https://cloud.google.com/solutions/best-practices-for-building-containers
* https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations
* https://cloud.google.com/solutions/best-practices-vpc-design
* Container Native Load Balancing
    * https://cloud.google.com/kubernetes-engine/docs/how-to/container-native-load-balancing
    * [Container Native Load Balancing at next 18]()
* https://cloud.google.com/blog/products/gcp/with-forseti-spotify-and-google-release-gcp-security-tools-to-open-source-community15
