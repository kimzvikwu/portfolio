# Agnostic Identity Management

## Scenario

A healthcare technology company is deploying a sophisticated, scalable AI and data platform across multiple cloud instances. Each instance of the platform is called a "deployment," and deployments are remotely managed by a "control cloud" instance, which serves as the primary hub for managing, updating, and monitoring them. The platform will ingest, process, and store sensitive healthcare data and requires robust security controls to protect data at rest, in transit, and during processing.

---

## 1. Architecture

**Prompts**

- a. Sketch out a high-level security architecture for the platform, considering its remote management from the "control cloud" instance.
- b. Identify the key cloud services that would be integral to the design and explain the choices.

**High-level design components**

- **Trust sources:** Vendors, Cloud Providers, Customer Premise
- **Zero Trust layer:** User/App Auth, Device Auth
- **Identity pipeline:** Map → Sync → Persist/Cache
- **Identity store (PostgreSQL):** User / Service Accounts, Groups, Roles, Devices

**Key cloud services**

- **AWS:** IAM, Secrets Manager, RDS PostgreSQL, ElastiCache, Trusted Advisor, Lambda, Kinesis Data Firehose, CloudWatch, SNS, SQS, DynamoDB, Amazon MQ, API Gateway, EKS, GuardDuty, Terraform
- **GCP:** —
- **Azure:** —

---

## 2. Access Control

**Prompts**

- a. Describe how to ensure the right entities (e.g., services, administrators) have the appropriate permissions to access and update the platform, while preventing unauthorized access.
- b. Outline a strategy for managing and rotating cryptographic keys and credentials.

**Approach**

- Continuous testing of IT controls to ensure roles and permissions remain intact. Terraform Enterprise automatically reverts anomalies by reinstating state.
- Rotate keys every 90 days. The interval is configurable and can be specified by the customer.
- For Kubernetes clusters, RBAC can be synchronized with IAM roles and groups.

---

## 3. Remote Monitoring

**Prompts**

- a. Discuss the challenges of managing deployments remotely via a "control cloud" instance.
- b. Describe the security precautions to ensure the integrity of updates and communications between the control cloud and other instances.

**Tooling**

- **Grafana** — dashboards
- **Prometheus** — observability (o11y)
- **Slack** — notification escalation

---

## 4. Compliance and Auditing

**Prompts**

- a. Enumerate the steps necessary to ensure the system adheres to industry-standard compliance requirements.
- b. Suggest tools or methodologies to regularly audit the platform's security posture and detect anomalies or breaches.

**Approach**

- **HIPAA** — ensure the system is assessed for HIPAA compliance by an external auditor, with annual audits performed.
- **CISO controls** — require that all releases comply with security controls.
- **SOC 2** — conduct a SOC 2 audit to test internal IT controls.
- **Tools** — SD Elements, Nessus, Nightfall, Vanta
- **Methodology** — continuous validation using the code repository, plus external auditing

---

## 5. Operational Alignment

**Prompts**

- a. Detail how to align the operations team with the proposed security strategies and protocols.
- b. Describe the training or awareness initiatives to ensure the entire team is well-versed in security practices.

**Approach**

- Shift-left methodology
- Secure Code Warrior to train engineers
- Yearly infosec training
- Quarterly security testing

---

## 6. Security Enhancements

**Prompt**

- a. Elaborate on how technological innovations (e.g., Zero Trust Architecture, AI-driven threat detection) could be incorporated into the platform's security strategy.

**Approach**

- **Zero Trust** — secure all resources so that only identified service accounts and users may access them, including EC2 instances, databases, and EKS clusters.
- **SBOM** — maintain an available list of the libraries and versions included in every container and image.

---

## References

- Cloud Agnostic Design — Medium
- Top Identity Solutions — PC Magazine
- Zero Trust — NIST
