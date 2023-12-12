# Day 1 - Getting Access
* Infrastructure Public Cloud - Orientation - https://docs.ipc.inday.io/onboarding/orientation/
* Root JIRA ticket: https://jira2.workday.com/browse/INF-281548
* LDAP access to groups:
	* inf.ipc (approved by Ronan French)
	* inf-gcp
	* infrastructure.engineering
	* infra-systems
	* infrastructure.servers
	* infra-systems.pci
	* splunk.web.inf.users (approved by Nitin Bhatia)
	* ipam.superadmins
	* Bitbucket / AD group: Stash-users
	* pharos.metrics (Okta - Pharos)
	* Okta - Grafana CLE Stage
	* Okta - Grafana CLE Prod
	* Okta - Big Panda
	* github-users
# KT - F5
* [KT - Sessions](https://docs.google.com/document/d/1SO-sTpR4rTKM1hRQGoYQK8myZEeNF6A6QCs2-0wGg3Q/edit)
* [KT - Recording](https://workday.zoom.us/rec/play/V8KeZtpH3sMMBCcqdGJDJUMfVE4eBbviVT9P6meudqDYDkuleRjO5J7KXnbEaki5k8b0HhwLVNsp97EP.epos6surtHLTqkfD?canPlayFromShare=true&from=share_recording_detail&continueMode=true&componentName=rec-play&originRequestUrl=https%3A%2F%2Fworkday.zoom.us%2Frec%2Fshare%2FhGhCuR2H80GlaJRrU34PFI8vi76N9AU73nxhN3nRH79_l6qTe2WwjUYjqQDIIkwJ.flGGbayQ5JqBU_Uk)
	* Passcode: b7^.G61A
* [AWS - F5 BigIP Runbook](https://confluence.workday.com/display/INFRA/AWS+F5+Bigip++-+Runbook)
* Presenter: Elliot Davie
* wd10.workday.com / wd5.workday.com
* Need to eliminate F5 in favor of istio ingress managed by Scylla on AWS
* Scylla deployments in AWS
	* SWC - Scylla Workday Clusters
		* first iteration of the Workday application being deployed within Scylla's Kubernetes clusters.
		* Thats what we within IPC operate and thats what the F5s are sitting in front of.
		* So in general, F5 -> SWC -> Istio Ingress 
	* SPC - Scylla Platform Clusters. Ultimately, SPC are the replacement of SWC
		* SPCs are Kubernetes clusters completely managed by Scylla
		* Can be bulk deployed
		* Leveraging Venice for internal service communication
		* The need for F5s are completely gone in favor of entirely using Istio Ingress
* AWS - Maintenance windows
	* Weekly patch - zero downtime
		* Integration traffic can be affected but UI traffic cannot
	* Hybrid patch
		* Allowed to interrupt everything
* We deploy our F5s using combination of Terraform and Ansible
	* Takes about 45 mins tops to configure from scratch
	* First we deploy using Terraform, we have a bootstrap config that would be applied - this lays down rough configuration - for dropping admin pw, sshd, etc
	* After the instances are stood up with that base config, we would run an Ansible playbook to apply the remaining items
	* Total 81 F5s across 31 environments
	* No DDoS or WAF activities on these F5s
	* Need to do DDoS and WAF on the Cloudflare
	* F5 Deployments - Staggered vs Predeployment/Swap method
* Pharos - is the monitoring platform
	* previously we were using a tool called wavefront
	* SNMP exporter - to gather metrics from F5s to send to the Pharos
		* it is a prometheus exporter - runs on a different AWS instance
# KT - Transit Gateway (TGW) / Lighthouse
Connectivity from AWS VPCs to the DCs
* [KT - Recording](https://workday.zoom.us/rec/share/R2wAMe1FuiJaGWAuqlc2Qce1gWE4VPenPSDmuSregpIHgdYibtMi4QxqYM-GuozF.2lQrQcbCcdRYgSZI)
	* Passcode: 0L?eeS36
* [Slide deck](https://docs.google.com/presentation/d/1l32UlPGlwohSSFD5UKK4fP81orV8Rumkf6VRO_RiD3c/edit#slide=id.g214e45718d0_0_5)
* Presenters: Shabia Yosuvaraj, Kai Berkutova

* DCs:
	* PDX-ENG - Our development environments
	* Any BT DC - enables connectivity through the Corporate network
	* Any CUST DC - carry customer traffic. In Dublin, AMS, ATL and Ash

* TGW
	* There are 5 flavors of TGW for DC connectivity
	* IPCTGW1 for AWS to PDX-ENG
	* IPCTGW2 for AWS to PDX-ENG and Any BT DC
	* IPCTGW3 for AWS to PDX-ENG, Any CUST DC
	* IPCTGW4 for AWS to Any CUST DC
	* IPCTGW5 for AWS to Any CUST DC (but exclusive for FEDRamp)

* How does TGW work?
# KT - Public Cloud Enablement (PCE)
Provision a cloud account in GCP. These are done per environment.

* [KT - Recording](https://urldefense.com/v3/__https://workday.zoom.us/rec/share/vR83Wcy5YoSBiDUxhv2djbcrbGeef95egfx--s6IMkkplkMby5Eir_w2Xf94WxPF.eSCWKAiH24o2uB08__;!!Iz9xO38YGHZK!5eq7waUbOjFjnjTQ0r03fg9jXVGpTnm8LRNSl-nuTmcshP8vX7PAdON3HQBDsoPtSxrhUJ6BOiQJWAKf3ihV$)
	* Passcode: =G0mHT*3
* Presenter: 
* Forms for GCP are done using service-now
	* Earlier - 
# KT - IPC Service Offerings

* [KT - Recording](https://urldefense.com/v3/__https://workday.zoom.us/rec/share/60FEmSJ-pMiezW-v3SiUQD8QVzRyg4kynYVj7s3Jz649FqlXi21ubhJye-yQnjMr.r7ccHrCYlDzZOe4P__;!!Iz9xO38YGHZK!9PYfaYsgMXUm7ZhM95DpaMi5V3oniFSrzkReUXyGRo3Nc0Ua70OJa2sg3VV7cC1ReWgyaUX50VzBqPLinp0Dvw$)
	* **Passcode: @1ca6LuC**
* [Slide deck](https://docs.google.com/presentation/d/1ypyCIb4P_X3p3uUMsdlD1oNs_mb3kxSKMw1RNEx64WI/edit#slide=id.g83a7b7cd0e_7_1542)
* Presenter: Raymond Rhine
* [FluentD](https://confluence.workday.com/display/INFRA/Fluentd+Description)
## Bastions
* Entrypoint to our environments
* We use a tool called aws-tunnel - to send traffic through these bastions
* Two types of bastions - RSA and SSM. These refer to the second factor of auth
## Blackbox
* It is a probing service for HTTP, HTTPS, DNS, TCP, ICMP and gRPC
* It presents us metrics that Prometheus can scrape
* We use it to monitor latency between AZs. Also latency against various AWS service APIs like EC2, S3, KMS, etc.
## Logging - Fluentd and Fluentd Forwarder
* Fluentd is an open source data collector, which lets you unify the data collection and consumption for a better use and understanding of data
* Fluentd Forwarder - Sends our logs from the spoke to the hub
## CorpLDAP and ProdLDAP
* LDAP is the authentication service
* CorpLDAP - is rarely used by us. Dev-Test is the only account this is used in
* ProdLDAP - used everywhere else
## Prometheus
* Scrapes metrics and sends them out to a push gateway to Pharos
## SMTP
* Mail servers
* Mail servers SMS
## Vault
* Identity management
* Secret management
* Certificate management
# KT - TGWMON Prometheus AL2 - Shoulder surf
* Presenter: Muhammad Ahsan, Kai Berkutova, Keith Gaughan
* Attendees: Paul Omeally, Nathalie Andre, Magesh Thulasi
* [GHE Pull Request](https://ghe.megaleo.com/INFServices/terraform-base/pull/14001)
* [JIRA ticket](https://jira2.workday.com/browse/INF-281532)
