NetDevOps Foundation-IaaC 
Network Reliability Engineering (NRE)  
IT Solution Documentation 
HCL (Vyshagh Meyena), Omar Farook (Volvo) 
04/29/2026 
Draft v1.1 (for ARB Gate‑3 / NS Solution Review) 
Confidential 
Volvo Group – Digital & IT 
 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 2 (31) 
NAME OF DOCUMENT ISSUE DATE 
Technical Solution Document  5/18/2026 
3:28 PM ISSUER (DEP NO, NAME, PHONE, LOCATION) VERSION NO STATUS 
  Draft 
SUBJECT 
IT Solution Documentation 
 
 
  
Classification: Internal 
Table of Contents 
1. INTRODUCTION .............................................................................................................. 5 
1.1. About this document................................................................................................ 5 
1.2. Intended Audience .................................................................................................. 5 
1.3. Related documents ................................................................................................. 5 
1.4. Scope of this document ........................................................................................... 5 
1.5. Business Context .................................................................................................... 6 
1.6. Definitions & Glossary ............................................................................................. 6 
2. SCOPE OF SOLUTION.................................................................................................... 7 
2.1. Requirements .......................................................................................................... 7 
2.2. Intended usage and Limitations .............................................................................. 8 
2.2.1. Intended Usage ................................................................................................ 8 
2.2.2. Limitations ........................................................................................................ 8 
2.3. As-Is known drawbacks and limitations .................................................................. 9 
3. FUNCTIONAL DESIGN.................................................................................................. 10 
3.1. General Information............................................................................................... 10 
3.2. Conceptual ............................................................................................................ 10 
3.3. Logical ................................................................................................................... 11 
3.4. Physical ................................................................................................................. 12 
4. NON-FUNCTIONAL DESIGN ........................................................................................ 14 
4.1. Security .................................................................................................................. 15 
4.1.1. Authentication ................................................................................................. 15 
4.1.2. Authorization .................................................................................................. 15 
4.1.3. Integrity and Confidentiality ............................................................................ 15 
4.1.4. Accounting, Audit and Logging ...................................................................... 16 
4.1.5. Network Security ............................................................................................ 17 
4.1.6. Processes ....................................................................................................... 17 
4.1.7. Roles............................................................................................................... 18 
4.1.8. Policies & Directives adherence .................................................................... 18 
4.1.9. Security tests .................................................................................................. 18 
4.2. Deployment Views ................................................................................................. 19 
4.2.1. Network zones ................................................................................................ 19 
4.2.2. Deployment Views.......................................................................................... 20 
4.2.3. Deployment Platform Environments .............................................................. 20 
4.2.4. Landscape overview ...................................................................................... 20 
4.3. Availability .............................................................................................................. 21 
4.4. Manageability ........................................................................................................ 21 
Volvo Group – Digital & IT 
 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 3 (31) 
NAME OF DOCUMENT ISSUE DATE 
 2024 5/18/2026 
3:28 PM  
 
  
Classification: Internal 
4.4.1. Automation ..................................................................................................... 21 
4.4.2. Configuration management. ........................................................................... 21 
4.4.3. Life cycle management .................................................................................. 21 
4.4.4. Scalability ....................................................................................................... 21 
4.5. Event, statistics, and online monitoring event collection ...................................... 22 
4.5.1. Events collection ............................................................................................ 22 
4.5.2. Statistics and accounting collection ............................................................... 22 
4.5.3. Online monitoring events collection ............................................................... 22 
4.6. Capacity management .......................................................................................... 22 
4.6.1. Capacity manager .......................................................................................... 22 
4.6.2. Platform limits ................................................................................................. 22 
4.6.3. Data and Metrics ............................................................................................ 22 
4.6.4. Trend analysis ................................................................................................ 23 
4.6.5. Peak load management ................................................................................. 23 
4.7. Monitoring .............................................................................................................. 23 
4.7.1. Infrastructure monitoring ................................................................................ 23 
4.7.2. Application integration monitoring .................................................................. 23 
4.7.3. Performance monitoring ................................................................................. 23 
4.8. Logging .................................................................................................................. 24 
4.9. Disaster Recovery ................................................................................................. 24 
5. SERVICE DESIGN ......................................................................................................... 24 
5.1. General .................................................................................................................. 24 
5.2. Service levels ........................................................................................................ 24 
5.3. Offerings ................................................................................................................ 25 
5.4. Application-level SLA ............................................................................................ 25 
5.5. Deployment options............................................................................................... 25 
5.6. Charging ................................................................................................................ 25 
5.7. Tenant agreements ............................................................................................... 25 
5.8. Tenant Onboarding ............................................................................................... 26 
6. LANDSCAPE DEPENDENCIES .................................................................................... 26 
6.1. General .................................................................................................................. 26 
6.2. Dependency 1 ....................................................................................................... 26 
7. ARCHITECTURAL DECISIONS .................................................................................... 27 
8. IMPACTING NON-ARCHITECTURAL DECISIONS ..................................................... 29 
 
 
 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 4 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Version Change Log 
Version 
Description 
v1.0 
Initial draft created by <Vivek Singh> 
Date 
2026-04-29 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 5 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
1. Introduction 
1.1. About this document 
The Solution Documentation is owned by the Service Owner and contains all architectural 
information about a released version of a service.  
• Architecture Description (Conceptual, Logical, Physical) covering: 
o Process 
o Information Flows 
o Application 
o Infrastructure 
• Architectural decisions (who – what) 
• Fulfilled use cases (features, user stories) 
• Known drawbacks and limitations. 
1.2. Intended Audience 
It serves several purposes and can be shared with stakeholders for the following purposes: 
• Auditors and other roles mandated to review the respective IT service. 
• Design team, preparing Service Development and Enhancement activities. 
• Other IT Service providers when documentation of the As-Is is needed. 
1.3. Related documents 
The following documents are related and form a part of the system architecture. These 
documents should be referred to in this document. 
Document name 
Final_Project Charter_Sub-Lot 
1.4.5_NetDevOps Foundation 
(IaaC) 
Description / Reference / Date / Version 
FENIX Transition & Transformation - Documents - 1.4.5 - 
NetDevOps Foundation (IaaC) - All Documents 
1.4.5 Net DevOps Foundation 
(Iaac)_Business Review PPT v1.1 
1.4.5 Net DevOps Foundation (Iaac)_Business Review PPT 
v1.1.pptx 
NetDevOps(IaaC)- Integration 
Request V1.0 
NetDevOps(IaaC)- Integration Request V1.0.docx 
1.4. Scope of this document 
To outline the solution establishes Central and Standardized Network Infrastructure as Code 
(IaaC) and build Network Reliability Engineering (NRE) capability within the Volvo. 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 6 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
The solution standardizes network automation across managed network domains 
(LAN/WAN/WLAN/DCLAN/DDI/F5) using NSoT, GitHub Enterprise, Ansible Automation 
Platform AAP2.6,Network tools, ServiceNow Vanilla. 
1.5. Business Context 
Today, Volvo network operations are largely dependent on Manual processes, low 
automation and non-standard configurations causing slow change delivery, operational risk 
and higher MTTR.  
Under the Target Operation model, Volvo to adopt NetDevOps/NRE which introduces 
Standard automation, event driven self-healing, Auto-remediating Network incidents, Auto
provisioning to reduce manual effort, accelerate change delivery and improve network 
stability & reliability. 
1.6. Definitions & Glossary 
Item 
Definition 
NRE 
NetDevOps 
Network Reliability Engineering 
Network Development & Operations 
NSoT 
GitHub Enterprise 
Network Source of Truth 
Git‑based Operations 
SLO 
SLI 
Service Level Objective 
Service Level Indicator 
MTTR 
MTTD 
Mean Time To Repair / Restore 
Mean Time To Detect 
API 
IaaC 
Application Programming Interface 
Infrastructure as Code 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 7 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
2. Scope of Solution 
The objective of the NetDevOps IaaC solution is to establish a central, standardized and 
governed automation framework for network infrastructure across Volvo environments. 
The solution scope is as below: 
✓ Assessment and gap analysis of current network operations, Automation Maturity 
✓ Deploy Network Source of Truth for inventory, topology, Standard configurations 
✓ GitHub Enterprise‑based governance for change control, versioning and audits 
✓ Automated provisioning and remediation workflows trigger via “ServiceNow Vanilla
AAP” integration 
✓ Train the Volvo BAU as NRE/NetDevOps engineers to drive the Automation further. 
Integration will be enabled with Volvo‑approved enterprise systems including ServiceNow 
Vanilla, GitHub Enterprise, Ansible Automation Platform and network controllers, NSoT. 
The workflows ensure role‑based access, approval‑driven execution, audit logging, and 
policy compliance in line with Volvo security and architecture directives. 
2.1. Requirements 
Requirement 
Addressed In 
BR1.1 – Enable zero‑touch network 
provisioning and change automation 
using a centralized automation platform, 
to reduce manual effort and network 
change time to X minutes for approved 
use cases. 
The solution will use approved ServiceNow Vanilla & AAP 
integration, enabling approved catalog items to trigger AAP 
playbooks. 
Refer Section 3.2 (Functional Design), 7 (Architectural Decisions) 
BR1.2- Enable automated incident 
remediation and self‑healing workflows, 
triggered by monitoring alerts, to reduce 
MTTR for identified network incidents. 
The solution will use approved ServiceNow Vanilla & AAP 
integration, enabling approved AAP playbooks to trigger 
incidents healing. 
Refer Section 3.2 (Functional Design), 7 (Architectural Decisions) 
BR1.3 -Enable NetDevOps by integrating 
network management platforms. 
The architecture supports seamless API-based integration b/w 
AAP and Network tools (VeloCloud,Catalyst Center,Bluecat,F5, 
ACI) to deliver end-to-end automated workflows 
Refer Section 3.2 (Functional Design) 
BR1.4- Ensure centralized visibility, 
traceability of all automation executions, 
status, results, approvals. 
3 (Functional Design), 5 (Service Design), 7 (Architectural 
Decisions) 
Refer Section 3.2 (Functional Design) 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 8 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
BR 1.5 - Establish a centralized Network 
Source of Truth (NSoT) to standardize 
network data models, manage golden 
configurations, detect configuration drift, 
and drive consistent automation 
NSoT will be deployed into Dedicated Control Zone, Restricted 
Zone 
Refer Section 3.3 , 3.3 , 3.4 
BR 1.6 - Build organizational capability 
and readiness by enabling NRE, BAU 
teams with the skills required to operate 
automation platform and workflows. 
NRE Training  
Refer Section N/A  
BR 1.7- KPI Measurement  
Success Criteria and KPI measurement related to NetDevOps will 
reside in ServiceNow Vanilla. 
1.Comparing resolution time (MTTR) for automated use cases 
against historically manual resolutions time for Incidents. 
2.Time to implement automated changes will be compared 
against historical manual change durations. 
Refer Section 4.7 
2.2. Intended usage and Limitations 
2.2.1. 
Intended Usage 
This NetDevOps IaaC as a Type 1 service, owned and operated by HCL for HCL managed 
infrastructure within Volvo, supporting the following usage: 
• Network Configuration Management 
Standardized configurations. 
• Provisioning and Change Automation 
Automating network tools-controlled configuration changes. 
• Incident Remediation and Self-Healing 
Executing automated remediation workflows for monitoring alerts. 
• Compliance and Audit Enablement 
Maintaining auditable, version-controlled change histories, Vulnerability 
Management. 
*Network Automation is also agreed within AAP Project Scope. 
2.2.2. 
Limitations 
• The solution is initially scoped to pilot selected sites then to be Rollout to rest sites by 
BAU team who has enabled and trained as NRE engineer. 
• If any Legacy network devices or platforms lacking API/SSH support interfaces may 
remain out of scope 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 9 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
• Complex incidents requiring human in loop or cross‑domain coordination will 
continue to be handled manually. 
• Domains with low configuration standardization may require prior normalization 
before full automation can be enabled. 
• Device not managed via Network Tools will be in Automation scope but that needs 
direct SSH access via AAP. 
2.3. As-Is known drawbacks and limitations 
• No dedicated Network Source of Truth platform for Automation  . 
Volvo Group – Digital & IT 
 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 10 (31) 
NAME OF DOCUMENT ISSUE DATE 
 2024 5/18/2026 
3:28 PM  
 
  
Classification: Internal 
3. Functional Design 
3.1. General Information 
 The functional capabilities of the platform are mapped to the following core objectives: 
• Reducing Operational toil 
• This design bridges NetDevOps (build & change) and NRE (operate & improve) as below. 
 Functional Capability NetDevOps Value NRE Value 
GitHub-based automation Code-driven changes Reduced drift 
Event-driven automation Automation pipeline Faster remediation (INC, CR, SR) 
RBAC & approvals Controlled changes Safe operations 
Audit & logging Change traceability Incident analysis 
Reusable automation Standardization Toil reduction 
 
3.2. Conceptual  
 
 (As-Is)-There is no current Central Automation Platform for Network 
  
(To-Be) Close loop automation into incident, changes and manual tasks  
 
 
 
End-to-end NetDevOps ecosystem connecting ServiceNow Vanilla, NSoT, GitHub Enterprise, 
Ansible Platform (AAP), Network Management Tools and End Devices. 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 11 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
• ServiceNow Vanilla initiating provisioning changes and configurations tasks, 
remediation requests for Incidents  
• NSoT as the authoritative data source for device inventory golden configs and drift 
detection. 
Network Tools → NSoT (inventory sync)  
NSoT → AAP (runtime data) 
NSoT → GitHub (baseline sync) 
Servicenow Vanilla→  NSoT CI name, location, asset tag, manufacturer, hardware 
model, and serial number etc attributes will be fetched . 
• GitHub Enterprise as the configuration governance and versioning 
• Ansible Automation Platform (AAP) as the automation execution engine 
orchestrating validated playbooks. 
• Network Controllers/Tools (Catalyst Center, ACI, VeloCloud, F5, BlueCat) as the 
operational control plane endpoints for Network Devices 
• Monitoring stack Splunk producing alerts and ServiceNow Vanilla creating the 
Incident 
3.3. Logical 
(As-Is) Not applicable  
(To-Be) establishes a closed-loop automation model 
• Desired network state is defined and versioned in GitHub Enterprise 
• AAP enforces that state through controlled execution 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 12 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Production Devices (Routers, SD‑WAN, DC‑LAN, WLC/Aps etc.) receive desired 
configurations or state changes through Network Tools Or direct if not managed via Tools 
under AAP controlled playbooks. 
External data sources from controllers (Cisco/F5/BlueCat/VeloCloud) enrich Network Tools 
like information on Current Vulnerabilities, Latest Versions, EOL , EOS as Vulnerability 
Management is also in Automation Scope. 
Overall, the To-Be logical design introduces distributed execution, GitHub -driven, enhanced 
automation governance, capable of supporting large-scale network automation, self-healing 
use cases. 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 13 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
3.4. Physical 
1. Zone Placement & Execution Architecture 
✓ The Ansible Automation Platform (AAP) is inside the Volvo CSN Zone (Platform 
Gateway) 
✓ The Network Management Tools reside in Multiple SMN Zone 
✓ The NSoT is hosted in a Dedicated Control/Restricted Zone. 
*Low level details 
* AAP Placement Zone  
2. Traffic Flow & Connectivity (Inbound / Outbound) 
✓  AAP → Network Controllers uses HTTPS 443 and SSH 22 for playbooks execution. 
✓ Inbound Traffic (ServiceNow → AAP) flows primarily over HTTPS 443 for job triggers. 
Volvo Group – Digital & IT 
 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 14 (31) 
NAME OF DOCUMENT ISSUE DATE 
 2024 5/18/2026 
3:28 PM  
 
  
Classification: Internal 
✓ External Firewall boundaries enforce strict rules:  
o GitHub → AAP (HTTPS) 
o ServiceNow Vanilla → AAP (HTTPS) via IBM API Connect 
 
   3. Control & Data Exchange Paths 
✓ GitHub (Untrusted/Internet Zone)  
✓ NSOT ↔ AAP through controlled HTTPS REST calls. 
✓ ServiceNow Vanilla connects to AAP 
 
    4.Security & Segmentation Principles 
✓ The architecture isolating multiple components- internet facing, tools, devices , NSoT, 
AAP. 
✓ AAP → Production Devices connections; direct access or access via Network 
Management Tools with enforced RBAC and privileged access workflows. 
✓ External communication paths rely exclusively on TLS-encrypted channels (HTTPS 
443) ensuring compliance with Volvo Schedule‑8 & NS Guardrails. 
 
 
<<NSoT Hardware requirements (Both for PROD and Dev) Mentioned into below table>> 
Volvo NaC(NSoT) Platform VM's Requirements - On-Premises Deployment 
DEVELOPMENT         
VM Name 
CPU 
Cores 
RAM 
(GB) 
Storage 
(GB) Role/Services 
Dev-VM1 16 32 500 PostgreSQL + Redis 
Dev-VM2 16 32 256 Platform Web 
Dev-VM3 16 32 256 Celery Workers 
Dev Total: 3 VMs | 48 Cores | 96 GB RAM | 856 GB Storage 
PRODUCTION (HA)         
VM Name 
CPU 
Cores 
RAM 
(GB) 
Storage 
(GB) Role/Services 
Prod-VM1 32 64 500 PostgreSQL Primary (Patroni) 
Prod-VM2 32 64 500 PostgreSQL Standby (Patroni) 
Prod-VM3 32 64 300 K8s Worker A - Web + Celery + Beat 
Prod-VM4 16 32 300 K8s Worker B - Web + Celery + PgBouncer 
Prod-VM5 32 64 300 K8s Worker C - Celery + SSOT Jobs 
Prod-VM6 16 32 256 NaC Platform Worker 1 (Active) 
Prod-VM7 16 32 256 NaC Platform Worker 2 (HA Pair) 
Prod-VM8 16 32 100 HAProxy LB Active + Keepalived VIP 
Prod-VM9 4 16 200 K8s Master 1 + HAProxy Standby + Sentinet for Redis 
Prod-VM10 4 16 200 K8s Master 2 + Patroni DCS Arbiter 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 15 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
4 
16 
Prod-VM11 4 16 200 K8s Master 3 + etcd 
Prod-VM12 
Prod-VM13 
100 Redis HA Node 1 
4 
16 
100 Redis HA Node 2 
Prod Total: 13 VMs | 212 Cores | 464 GB RAM | 3,312 GB Storage 
4. Non-Functional Design 
3 is missing 
4.1. Security 
All network automation activities performed through the NetDevOps IaaC platform follow 
Volvo’s enterprise security standards (it leverages AAP and its agreed Security standards like 
Encryption at REST, access control, Ansible Vault etc.) 
Data exchanged between ServiceNow, GitHub Enterprise, Network Source of Truth, Ansible 
Automation Platform, and network controllers are encrypted in transit using TLS 1.2+ . 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 16 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
4.1.1. 
Authentication 
All HCL users authenticate via Volvo Microsoft Entra ID using Single Sign-On (SSO). 
NSoT to Management tools, AAP integrations authenticate using API tokens with strictly 
scoped permissions. 
4.1.2. 
Authorization 
Authorization within the NetDevOps IaaC solution is enforced using role-based access 
control (RBAC). 
Permissions will be mapped to Volvo Entra ID groups and applied consistently across GitHub, 
NSoT, Automation workflows and execution environment (Network Tools) we have in Volvo, 
ensuring least-privilege access. 
Roles include: 
Usage profile (role) 
Use case(s) 
Authorization 
Administrators 
(Automation 
Engineer) 
Designs, develops and maintains 
automation workflows, playbooks, and 
integrations 
Volvo SSO 
Supervisor 
(NRE Engineer) 
Owns standards, governance, and cross
team coordination for resilient operations.  
Volvo SSO 
Analyst 
(Platform Engineer) 
Manages day-to-day network 
infrastructure operations and supports 
lab/test environments.  
Volvo SSO 
Service Account 
API integrations 
Volvo SSO 
* mention  Citrix access, PAM  
4.1.3. 
Integrity and Confidentiality 
Integrity is ensured by enforcing GitHub based change management, mandatory code 
reviews, version control, and approved execution paths. 
Automation workflows cannot be executed directly without approval via ServiceNow Vanilla 
Approval Flow -  
A typical integrity‑controlled approval flow works as follows: 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 17 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Service Request/Requestable items – will go through the process and approval flow ( if 
needed) 
• Ground rule is – It depends on each SR/RITM process. 
• For approval RITM/SRs – 
o User submit/fill the catalog form 
o ServiceNow Vanilla triggers for approval 
o Upon approvals from approvers 
o Payload triggers to AAP 
o AAP executes 
o Upon failure condition – fall back to process flow /agreed in scope 
▪ For eg; Ctask can be created – stating this particular step failed and 
warrant for manual execution. 
▪ Notification etc…  
▪ It all depends on scope agreed during development. 
o Upon success condition – Close the ticket 
For incident automation – 
• Incident lands in ServiceNow Vanilla – Designated ServiceNow groups 
• ServiceNow Vanilla validate the condition based on ServiceNow Vanilla groups and 
pushes the right template in AAP 
• AAP execute the automation 
• Upon success – close the ticket 
• Upon failure – route the ticket to next level support team (ServiceNow Vanilla group) 
and update the ticket status with failure note. (This is part of development) 
4.1.4. 
Accounting, Audit and Logging 
All Automation job activities are traceable as per the policies defined into Ansible 
Automation platform. 
All Retention policies will same as defined into AAP. 
4.1.5. 
Network Security 
Operate strictly in customer‑approved, secure environments, define firewall port for APIs 
and connectivity. 
Port(s) 
Source 
Destination 
Protocol 
443 
ServiceNow Vanilla 
NSoT 
HTTPS 
Comments 
22,443 
API-driven request 
NSoT 
GitHub 
443 
SSH,HTTPS 
NSoT 
Network Management Tools 
HTTPS 
For API/webhook 
22 
For REST API 
NSoT 
Network Devices 
SSH 
Using TACACS 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 18 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Port(s) 
Source 
Destination 
Protocol 
443 
Comments 
AAP 
Network Management Tools 
HTTPS 
22,443 
Standard API calls 
AAP 
GitHub 
SSH,HTTPS 
22 
Need to Validate 
AAP 
Network Devices 
SSH 
443 
Direct automation 
ServiceNow Vanilla 
AAP 
HTTPS 
4.1.6. 
Processes 
Need to Validate 
This Project processes for delivering reliable network automation, ensuring operational 
readiness, and embedding reliability principles across the network and automation 
landscape. Each process activity is owned by NRE Engineer with collaboration from 
Automation Engineers, Platform Engineers, ServiceNow Vanilla support, Monitoring team 
and respective Track SPOC. 
1. Planning & Use-Case Definition Derive technical use-cases in collaboration with BAU and 
HCL Service owners. 
2.Workflow Design & Development 
• Automation Engineer develops workflows/playbooks according to standards & agreed 
architecture for Network Reliability Engineering, but BAU must maintain/improve the 
architecture & standards post-handover /Go-Live. 
• NRE create workflows for reliability, governance and Network tools integration and 
reviews with HCL BAU, Service owners. 
• Platform Engineer provides infrastructure readiness and input on operational 
dependencies.  
3. UAT & Pre-Prod Validation 
• Conduct lab-based UAT and failure/resilience testing.  
• Measure playbook success rate, runtime delays, and platform health.  
• NRE & BAU/Service owners approve or suggest enhancements based on UAT outcomes 
before release based on operational readiness. 
4. Release & Deployment 
• Define release schedule aligned with BAU maintenance windows.  
• Conduct production deployment, rollback planning, and hyper care verification.   
5.Operation Readiness & Handover  
• Maintain SOPs, runbooks and documentation.  
• Formal handover to operations team with KT sessions.  
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 19 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
• Maintain versioned playbooks and operational procedures.  
4.1.7. 
Roles 
This section defines the roles and responsibilities involved in the NetDevOps operating 
model to ensure consistent delivery, automation, governance and support of network 
services throughout their lifecycle.  
The NetDevOps roles support: 
• Automation Engineer: Designs, develops, and maintains automation workflows, 
playbooks, and integrations.  
• Network Reliability Engineer: Owns reliability standards, governance, and cross
team coordination for resilient operations.  
• Platform Engineer (BAU): Manages day-to-day network infrastructure operations 
and supports lab/test environments.  
• ServiceNow Vanilla Support Team: Manages ITSM workflows, SRDs, and automation 
integration within ServiceNow Vanilla.  
• Monitoring / Splunk Team: Provides monitoring, dashboards, and telemetry for 
automation and network reliability.  
• HCL BAU/Service owners define business requirements, provides lab access and 
Service owners (LAN/WAN/ WLAN etc..) approves workflows and releases. 
4.1.8. 
Policies & Directives adherence 
The NetDevOps is designed, implemented, and operated in full alignment with Volvo’s 
enterprise security policies and regulatory requirements. The service relies entirely on 
Volvo-approved infrastructure, identity services, and governance processes, ensuring that all 
network automation and change activities comply with internal and external mandates. 
4.1.9. 
Security tests 
The Framework is aligned to Volvo’s North Start principles.  
Applicable security tests for NSoT deployed as required by security team will be carried out. 
• Automation Artifact Security Testing 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 20 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Network automation artifacts, including configuration templates, playbooks, and 
workflow definitions, are subjected to review compliance with approved network 
standards, secure configuration baselines, and access control requirements. 
• Infrastructure and Integration Security Testing 
The NetDevOps service undergoes security testing related to: 
✓ Integration points  
✓ Secure communication between Ansible platforms & managed network Tools  
✓ Servicenow Vanilla to NSoT CI level data retrieval . 
4.2. Deployment Views  
4.2.1. 
Network zones 
The NSoT is hosted in a Dedicated Control/Restricted Zone. This zone provides a controlled, 
enterprise‑approved environment that enforces Volvo’s access restrictions. All 
communication between components, target systems, ServiceNow through this zone, 
ensuring secure connectivity, compliance with Volvo directives, and isolation from untrusted 
networks. 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 21 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
4.2.2. 
4.2.3. 
Deployment Views  
Deployment Platform Environments 
Hosted on Volvo Dedicated Control Zone. 
4.2.4. 
Landscape overview 
The NetDevOps landscape provides a centrally governed, automated, and secure 
framework for designing, validating, and deploying network configurations across the 
enterprise. It integrates version-controlled, Network Source of Truth, Ansible automation 
platforms, ServiceNow Vanilla and network infrastructure .The NetDevOps landscape is 
logically structured into the following layers 
1. Servicenow Vanilla as ITSM Layer 
2. NSoT as Source of Truth & GitHub as Version Control Layer 
3. AAP Automation & Orchestration Layer- 
4. Network Management Tools & Integration Layer 
5. Network Infrastructure (Router, Switch, F5 etc..) 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 22 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
4.3. Availability 
The NetDevOps service considerations throughout its architecture is dependent on AAP. 
NSoT is new platform introduced with Redundant platform components  
4.4. Manageability 
4.4.1. 
Automation 
The Central NetDevOps service supports end-to-end automation of network operational 
workflows, including configuration changes, provisioning, validation, and lifecycle 
operations across LAN/WAN/WLAN/DCLAN,DDI,F5 . The service enables centralized 
execution of network automation through approved Ansible platforms and can be 
incrementally expanded as new network automation use cases and requirements emerge 
4.4.2. Configuration management. 
Configuration management within the NetDevOps is achieved through standardized, 
version‑controlled playbooks. Also ensures configuration consistency, repeatability, and 
security across development, production environments. 
For NSoT solution there will scheduled sync with ServiceNow Vanilla for CI level data. 
4.4.3. Life cycle management 
The NetDevOps service supports the full automation lifecycle, including: 
• Version control playbooks of network configurations and automation  
• Time to time changes as required into Playbook 
4.4.4. Scalability 
The architecture allows NetDevOps to: 
• Scale horizontally as network automation demand increases (Depends on AAP 
Solution) 
• Support concurrent automation executions without performance degradation 
4.5. Event, statistics, and online monitoring event collection 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 23 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
4.5.1. 
Events collection 
✓ Used existing from Splunk alerts to ServiceNow Vanilla Incidents. 
✓ Manual Changes, Tasks for Network taken for assessment. 
✓ Some Events may be enabled as a part of NetDevOps Assessment and Gap analysis (Once 
approved by relevant Service Owner from HCL, Volvo) 
4.5.2. 
Statistics and accounting collection 
✓ Historical Tickets data from the past six months has been reviewed as part of the 
analysis. 
✓ Identified recurring issues, Toils from ticket dump 
4.5.3. 
Online monitoring events collection 
✓ No new monitoring tools or event sources are introduced  
4.6. Capacity management 
4.6.1. 
Capacity manager 
The NetDevOps solution is designed to handle capacity requirements based on current usage 
patterns and future expansion scenarios, without requiring architectural changes. 
4.6.2. 
Platform limits 
The NetDevOps automation platform relies on enterprise services such as: 
• ServiceNow (Vanilla) 
• AAP 
• Github repositories 
• NSoT (will be deployed and Resource planning is done considering the CI volume). 
Any outage, latency, or degradation in these systems can directly impact automation 
execution or service availability. 
4.6.3. 
Data and Metrics 
N/A - At the current stage of the NetDevOps service design, no data or service performance 
metrics are defined or collected independently. 
4.6.4. 
Trend analysis 
Historical analysis and trend identification, where required, are performed using existing 
ServiceNow Vanilla capabilities (e.g., incident, change trend analysis). 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 24 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
No other dedicated NetDevOps‑specific trend analysis or forecasting mechanisms are 
defined at this stage. 
4.6.5. 
Peak load management 
NetDevOps will use APP that have ability to handle parallel job bursts during peak execution 
windows without queue saturation. 
Multiple tickets, Multiple Changes can be processed simultaneously. 
4.7. Monitoring 
All KPI related measurement related to NetDevOps will reside in ServiceNow which is the 
triggering point for Provisioning tasks and Healing workflows and will be used to generate 
KPI-based performance reports and dashboards. 
(a)Incident Management: 
We will track the trend of incidents resolved manually versus those resolved through 
automation. Automated resolutions should show a steady increase quarter-on-quarter. 
We will also compare resolution time (MTTR)  for automated use cases against historically 
manual resolutions for Incidents. 
(b)Change Management: 
We will monitor the trend of changes executed manually versus those delivered through the 
NetDevOps platform, with an expected upward trend in automated changes & requests 
delivery. Time to implement automated changes will be compared against historical manual 
change durations.  
4.7.1. 
Infrastructure monitoring 
NSoT Servers monitoring will  be done via Monitoring tools based on ICMP, SNMP. 
4.7.2. 
Application integration monitoring 
N/A as this Network Infrastructure Automation framework. 
4.7.3. 
Performance monitoring 
NSoT VMs will be monitored using iObserve monitoring tools for performance health checks. 
4.8. Logging 
We are leveraging Ansible Automation Platform here which generates detailed logs for all 
automation activities including job execution, status, errors, and results. 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 25 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
4.9. Disaster Recovery 
The solution relies on: - Built-in redundancy capabilities of AAP and availability on other integrated tools. - High-availability deployment of NSoT - Enterprise backup and recovery mechanisms 
In the event of a failure: - Automation services may be temporarily unavailable - Network operations continue unaffected, as automation is not in the data plane. 
Recovery procedures include: - Restoration of platforms from backup & validation of integrations and workflows. - Controlled resumption of automation activities 
5. Service Design 
5.1. General  
The NetDevOps IaaC service is delivered as a centralized, enterprise-grade automation and 
orchestration capability across all in-scope Network domains within the Volvo environment. 
The service establishes a unified operating model Network Reliability Engineering (NRE) 
practices (operate and improve) that integrates NetDevOps (build and change automation) 
enabling a shift from manual, device-centric operations to a platform-driven, automated, 
and policy-based approach. 
5.2. Service levels 
The service has separate service levels , it is designed to improve network operations by 
standardizing automation, reducing manual effort and enabling faster, more consistent 
execution of changes and incident remediation activities for Service Level owners 
LAN/WLAN, WAN,DCLAN/F5,DDI,VM. 
✓ The service is designed such that even in the event of automation platform 
unavailability, network devices continue to operate normally, and manual operational 
processes remain available as a fallback mechanism.  
✓ Therefore, service impact is limited to automation and orchestration capabilities, 
without affecting the underlying network operations. 
5.3. Offerings 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 26 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Offering 
Description 
Applicable 
Domains 
Automation-as-a
Service (Incidents, 
Changes , Tasks) 
Centralized execution of approved 
automation workflows triggered via 
ServiceNow and executed through 
Ansible Automation Platform (AAP). 
LAN, WAN, WLAN, DCLAN, 
DDI, F5 
Configuration 
Management & 
Standardization 
Version-controlled configuration 
management using GitHub Enterprise and 
NSoT for enforcing golden configurations 
and compliance. 
LAN, WAN, WLAN, DCLAN, 
DDI,F5 
Network Source of 
Truth (NSoT) 
Services 
All network domains 
Centralized Network repository for 
inventory, topology, metadata, and 
golden configuration baseline. 
5.4. Application-level SLA 
Application-level SLAs are not applicable since it’s a CORE service. 
5.5. Deployment options 
Automation: 
• Pilot Deployment - Limited rollout to selected sites and use cases as per the assessments, 
BAU and Service Owners confirmation. 
• Controlled Expansion- Increase automation coverage to additional domains and sites by the 
NRE elected team from BAU, Post Handover. 
5.6. Charging 
No Cost (tooling, platform, license, run costs) as this Project is under Fenix Contract Model. 
If there anything else not in this contract that should be taken separately between Service 
Leaders and HCL Service Owners. 
5.7. Tenant agreements 
NSoT solution is On-Prem deployment and will be integrated with existing tools stack only. 
Also, NetDevOps act as Central Automation platform for Network with NRE ways of working. 
So, tenant agreements is not applicable. 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 27 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
5.8. Tenant Onboarding 
Tenant agreements are not applicable for this Solution. 
6. Landscape Dependencies 
6.1. General  
# 
Material 
Manged 
by 
Description  
1 
ServiceNow Vanilla 
Integration 
Volvo 
REST APIs are required for integration between 
ServiceNow Vanilla and NSoT environments. 
2 
Tools Integration 
AAP, NSoT ,Network 
Tools, GitHub 
HCL 
REST APIs are required for integration between 
these components. 
3 
NSoT Hosting 
HCL 
VMs in selected DC location  
6.2. Dependency  
1) ServiceNow Vanilla (Changes, RITMs) Catalog Creations for Automation 
Volvo Group – Digital & IT 
 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 28 (31) 
NAME OF DOCUMENT ISSUE DATE 
 2024 5/18/2026 
3:28 PM  
 
  
Classification: Internal 
7. Architectural Decisions 
Area Decision- Central Automation platform via Tools chain 
Standardization  (GitHub + NSoT + Ansible Automation 
Platform+ ServiceNow Vanilla + Network Management Tools) 
Topic Id AD1 
Design Decision Central automation platform (e.g., Ansible Automation Platform) to be used for 
network changes, tasks, Incidents 
Assumptions Controllers such as Catalyst Center /ACI /Velocloud and Servicenow Vanilla 
expose APIs 
Motivation repeatable workflows aligned to reliability goals and change governance. 
Alternatives 
rejected 
manual change processes, Incident handling , Network Task completion 
Justification NSoT and Ansible automation platform as core enablers 
Implications  
Related Project 
Gates 
NS Business (detailed design) 
Related 
Decisions 
AD‑2 (NSoT) 
 
Area Network Source of Truth (SoT) Technology  Topic Id AD2 
Design Decision Select the HCL appointed NSoT technology and define the data model & 
governance (devices, templates, golden configs, metadata) 
Assumptions Inventory and configuration baselines can be synchronized with NSoT with 
devices, Controllers. 
Motivation Eliminate configuration inconsistency, accelerate changes, and provide a single 
point of governance for policy/config across domains. 
Alternatives 
rejected 
No Source of Truth  
Justification Charter milestones explicitly call for NSoT build & device onboarding, golden 
templates, and drift management. 
Implications Data quality ownership, onboarding playbooks, integration and controllers 
Related Project 
Gates 
NS Business (detailed design) 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 29 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
Area 
Network Source of Truth (SoT) Technology  
Related 
Decisions 
Topic Id AD2 
AD‑1 (Toolchain Standardization) 
Area  
Integration ServiceNow Vanilla and NSoT 
Topic Id AD3 
Design Decision  
Assumptions  
Motivation  
Alternatives 
rejected 
Justification  
Implications  
Related Project 
Gates 
Related 
Decisions 
Classification: Internal 
Volvo Group – Digital & IT 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 30 (31) 
NAME OF DOCUMENT 
2024 
ISSUE 
DATE 
5/18/2026 
3:28 PM 
8. Impacting non-architectural decisions 
Add one table for each significant non-architectural decision agreed as part of this solution. 
These may not affect or change the architecture as such but may still have an important 
impact on the solution and hence their rational is to be documented for traceability. Use a 
new table for every decision to be logged. 
Don’t leave empty fields on the table. 
Area  
Design Decision  
Topic 
Id 
Assumptions  
Motivation  
Alternatives 
rejected 
Justification  
Implications  
Related Project 
Gates 
Related 
Decisions 
Volvo Group – Digital & IT 
 
TYPE OF DOCUMENT PAGE 
Schedule 8-A 31 (31) 
NAME OF DOCUMENT ISSUE DATE 
 2024 5/18/2026 
3:28 PM  
 
  
Classification: Internal 
Style Conventions 
 
Convention Use 
Bold font Characters that are typed exactly as shown, including commands and 
switches. Bold can also be used to provide emphasis to normal text. 
Italic font In syntax statements, indicate placeholders for variable information. 
Italic fonts are also used to indicate a placeholder for variables where 
specific values need to be supplied. For example, Component indicates 
that you should replace the italicized Component with the name of your 
choice. 
ALL 
CAPITALS 
Indicate the names of keys, key sequences, and key combinations (for 
example, CTRL+ALT+DELETE) 
Monospace Represents code samples, examples of screen text, or entries that you 
type at a command prompt or in initialization files (sample: SQL 
queries, PowerShell code or INI files). 
[ ] In syntax statements, enclose optional items. For example, [filename] in 
command syntax indicates that you can choose to type a file name with 
the command. Type only the information within the braces, not the 
braces themselves. 
{ } In syntax statements, enclose required items. Type only the information 
within the braces, not the braces themselves. 
(Sample: {Component}, {Component Module}, {CI} or {File Name} 
< > Input parameters enclose required items. Type only the input value not the 
angle brackets themselves. 
Sample: <DomainName>, <Scope> 
| In syntax statements, separates an either/or choice (Yes | No). 
 Use standard bullets for listing items 
 Use number bullets for listing the steps in an approach.
