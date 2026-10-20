**Kim Zvik**

Centralized vulnerability management and POA&M automation for FedRAMP environments
- [VANQUISH](#vanquish)

- [Case Studies](#case-studies)
-- [Restructuring a stalled Azure migration so engineers could build](#Restructuring a stalled Azure migration so engineers could build)
-- [Making the case for a centralized ZooKeeper team before the outages made it for me](#Making the case for a centralized ZooKeeper team before the outages made it for me)
-- [Tracing a silent TLS handshake failure across a global WAN](#Tracing a silent TLS handshake failure across a global WAN)
-- [Replacing four days of manual release testing with one dashboard of truth](#Replacing four days of manual release testing with one dashboard of truth)
-- [A project retrospective people trusted enough to be honest in](#A project retrospective people trusted enough to be honest in)

- [Talks](#talks)
-- [One identity model for every cloud, vendor and customer site](#One identity model for every cloud, vendor and customer site)
-- [A multi-tenant clickstream pipeline from global apps to trusted reports](#A multi-tenant clickstream pipeline from global apps to trusted reports)
-- [Engineering excellence and digital instrumentation](#Engineering excellence and digital instrumentation)
-- [Putting the “Sec” into DevOps](#Putting the “Sec” into DevOps)

- [Contact](#contact)

# Contact

## Kim Zvik

Cloud and infrastructure security leader. 15+ years turning operational gaps into programs that hold up under audit, outage and scale.

- Kensington, CA
- [kimzvik@gmail.com](mailto:kimzvik@gmail.com)
- [linkedin.com/in/kimzvik](https://www.linkedin.com/in/kimzvik)
- [github.com/kimzvikwu](https://github.com/kimzvikwu/vanquish)

My work, logged the way I’d log a finding: what was wrong, and how it got closed. Select a row to read the full case study.

| Ref | Era | Organization | Finding | Status |
| --- | --- | --- | --- | --- |
| KZ-00 | 2026 | [VANQUISH](#vanquish) | FedRAMP vulnerability data scattered across scanners, spreadsheets and manual POA&M reports | Built |
| KZ-01 | Early 2020s | [Federal banking agency](#federal-cloud) | Two prior cloud migrations failed; 35 engineers stuck in one meeting cadence | Resolved |
| KZ-02 | Late 2010s | [Global SaaS company](#distributed-db) | 13 teams running divergent, unpatched forks of Apache ZooKeeper under critical systems | Resolved |
| KZ-03 | Mid 2010s | [Credit ratings firm](#tls) | HTTPS rollout silently failing in Europe and Asia, blocking secure collaboration | Resolved |
| KZ-04 | Early 2010s | [Global payments company](#release) | 25,000 send-money paths tested by hand over four days per release | Resolved |
| KZ-05 | Early 2000s | [Investment analytics firm](#retrospective) | A late flagship web launch with no shared account of why it ran late | Resolved |
| KZ-06 | Early 2020s | [Healthcare AI platform](#identity) | Sensitive health data across many remotely managed cloud deployments, with no single identity model | Designed |
| KZ-07 | Early 2020s | [Streaming media company](#clickstream) | Clickstream data from many regions and tenants needed one path to BI that could hold up to financial reporting | Designed |

# VANQUISH

**VANQUISH**Independent project, 2026

## Centralized vulnerability management and POA&M automation for FedRAMP environments

Stack

React + Vite, Express, PostgreSQL (JSONB), Docker Compose, MCP server

Standards

OCSF, NIST NVD, FedRAMP POA&M

Design

Designed in Figma

Source

[kimzvikwu/vanquish](https://github.com/kimzvikwu/vanquish)

### The problem

In a FedRAMP program, vulnerability findings arrive from many scanners, researchers and customers in many formats. Someone has to normalize them, set severity, rule out false positives and keep the Plan of Action and Milestones current for the authorizing official. That work is usually a spreadsheet and a lot of hours.

### What I built

VANQUISH ingests vulnerability reports as CSV or JSON, normalizes every finding into the Open Cybersecurity Schema Framework (OCSF), stores it in PostgreSQL, enriches severity from the National Vulnerability Database, and exports a POA&M report on demand.

### How the agents are governed

The AI layer scans the codebase to detect false positives and flag needed operational risk adjustments. I designed it with separation of duties built in, the same way I’d design it for a human team:

- One agent works inside the FedRAMP boundary and another outside it.
- One agent changes code and opens the pull request; a different agent reviews and approves it.
- Guardrails are roles and groups defined in YAML. Activating an agent is requested through the identity governance system, like a service account.
- The schema ties departments (automation, data, process) to the identity rather than directly to the vulnerability ID, so accountability follows the actor.

VANQUISH also includes a DLP sandbox component. I’ve applied the same agent design to real vulnerability remediation work in my recent security roles.

[View the code on GitHub](https://github.com/kimzvikwu/vanquish)

**U.S. federal banking agency**Senior Director, DevOps\
Early 2020s, via a federal systems integrator

# Case Studies

## Restructuring a stalled Azure migration so engineers could build

Scale

$5M contract; 35 contractors hired in 4 weeks

Platform

Microsoft Azure

Outcome

New team model still in place a year later; contract extended past its checkpoint

### Situation

The agency had to close an on-premises data center on a hard deadline. Two earlier cloud transformations, one on AWS and one on Tanzu, had failed. My employer, a federal systems integrator, won a $5M contract to move the agency’s infrastructure to Azure, and I led the delivery.

### Building the team

We had four weeks to hire 35 contractors to meet our first milestone. I ran interviews alongside three architects and assigned each hire to the architect who wanted to lead them, so the structure grew organically. When a sourcing vendor challenged a decision, we formalized our interview questions with the vendor and HR so the process was fair and the hiring bar held.

### What I found

All 35 engineers and five product managers sat in a single, long sprint-grooming meeting every two weeks. Roughly 80% of the team’s time went to ceremony instead of engineering. Engagement dropped, debates got heated, and strong engineers started looking elsewhere. Talking to the product managers individually, I learned they wanted more autonomy, and I came to see the meeting structure as a likely contributor to the earlier failures.

### What I did

I reorganized the program into mini teams, each aligned to a single product manager and the agency department that manager represented. Each project manager and business analyst carried at most two teams. I socialized the org chart with my CTO first, then with the agency, and framed the change for agency leadership in terms of what it saved in meeting cost, since every dollar the agency spends is answerable to taxpayers.

### Outcome

The agency agreed to trial the structure, and it was still in place a year later. My CTO adopted the org chart as our company’s de facto reporting structure, and our contract continued past its November checkpoint.

**Global SaaS company**Principal Engineer, Core Infrastructure\
Late 2010s

## Making the case for a centralized ZooKeeper team before the outages made it for me

Context

Storage Engineering, in-house distributed database (Postgres-based Oracle replacement)

Tech

Apache ZooKeeper, mTLS, Puppet, NIST runbooks

Outcome

Centralized ZooKeeper team created after the proposal

### Situation

I joined a startup-style team of about 100 building a new in-house database, a Postgres fork meant to replace the company’s Oracle dependency and save millions a year in licensing. The database couldn’t run without Apache ZooKeeper, and I led the DevOps work supporting it. ZooKeeper wasn’t well understood on my team or across the company.

### What I found

To anticipate what my team would face, I studied ZooKeeper outages elsewhere in the company. Thirteen teams depended on it, each running a slightly different version, and none had contributed their changes back upstream, so every upgrade meant porting and retesting proprietary changes. We were a major version behind the stable release, and that newer release fixed race conditions and performance problems that matched many of the outages I’d catalogued.

I also noticed that public contributions to an open source project can reveal which companies run it, and at which version. I treated that as an exposure worth weighing when deciding what to contribute.

### What I did

- Worked with the other 12 teams on a joint proposal for a centralized ZooKeeper team.
- Built relationships with the organizers of the annual ZooKeeper meetup, which led to our engineers speaking on our C++ service discovery module and on mTLS, to help recruit scarce talent.
- Configured our ZooKeeper fleets for mTLS using Puppet, alongside the cybersecurity Fellow who handled certificate issuance and enforcement, a requirement for federal customers.
- When asked to pause the proposal, wrote the ZooKeeper runbooks for our air-gapped federal installations and NIST requirements, published as Markdown in GitHub rendered to Confluence.

### Outcome

Later that year, a split-brain incident hit our production pilot, and a race condition in another team’s cluster caused a production-wide outage. Because the proposal had circulated, teams turned to us. I used that moment to make the case again with the affected team’s leadership. After I moved on, the company created a centralized ZooKeeper team.

**Global credit ratings and analytics firm**Senior DevOps Engineer\
Mid 2010s

## Tracing a silent TLS handshake failure across a global WAN

Systems

Confluence, Jira, Bitbucket; Riverbed SteelHead WAN appliances

Tools

Wireshark, ServiceNow

Outcome

HTTPS enforced globally; InfoSec adopted the platform

### Situation

I administered the firm’s Atlassian platform for offices across North America, Europe and Asia, and migrated a MediaWiki holding more than 20 years of intellectual property, including research on how credit ratings are calculated, into Confluence. An information security engineer refused to join until the sites used HTTPS, rightly: over plain HTTP, his credentials would travel in clear text.

### What I found

HTTPS worked in North America but failed in Europe and Asia. Coming from software engineering rather than networking, I taught myself Wireshark, mapped the healthy handshake as a state diagram, and compared it against traces from the failing regions. The client hello went out; the acknowledgment never came back.

Over six months I opened 20 tickets. Each was closed without anyone contacting me.

The firewall team was measured on SLA, and the issue wasn’t theirs, but the new ServiceNow process had no escalation path to route it elsewhere while keeping it open. During an unrelated outage bridge, I documented every hop by name and IP, then worked with a security network engineer to pinpoint the drop: the Riverbed SteelHead WAN optimization appliances needed a rule to handle TLS rather than only legacy SSL.

### What I did

Networking leadership initially declined the change. I brought in Information Security for a second opinion, they backed the diagnosis, and the rule was applied across the appliances.

### Outcome

Every Atlassian site moved to HTTPS, and the security team started using the platform. Because I documented the fix in Confluence, my manager was able to apply the same change to his own servers a year later from a single link.

**Global money transfer company**Head of Quality and Release Engineering\
Early 2010s

## Replacing four days of manual release testing with one dashboard of truth

Scale

\~50-person team; 25,000 send-money test paths; web, iOS and Android

Tools

Jira, Zephyr, Confluence, Sauce Labs, Jenkins

Recognition

Employee of the Month

### Situation

The company was building a new digital technology division in San Francisco. I inherited a team of about 50 contractors, mostly in India, who manually tested the 25,000 ways a customer can send money. A release took four days of testing before being handed over the wall to IT Operations.

### What I found

Whole scenarios couldn’t be tested at all. Offshore testers had no access to certain US card types or to many of the Android models customers actually used. The mobile app carried a two-star rating, largely because it misbehaved across Android devices.

### What I did

- Replaced HP Quality Center with Jira for the division.
- Drawing on my device-testing work at a national wireless carrier, persuaded corporate to provision 40 phone models, against a one-phone-per-employee policy.
- When budget for mobile test automation was denied, secured funding from the VP of Marketing, who owned the app rating, and kept my SVP informed.
- Worked with our security manager to approve firewall access for external test VMs and to let a locked device lab run on the network without employee authentication.
- Split the 25,000-path web suite across parallel VMs to run four times faster.

### Outcome

I designed a Jira dashboard with three levels of detail that aggregated manual, automated web (Sauce Labs via the Zephyr API) and mobile results side by side, showing per build whether it was ready to release. When overnight offshore testing surfaced new bugs, I called it the offshore gift: better our testers find them than our customers.

**Investment analytics firm**Senior Program Manager\
Early 2000s

## A project retrospective people trusted enough to be honest in

Product

The firm’s first web-based portfolio analytics platform

Partners

Audit, Legal, IT, Engineering, Product

Outcome

Report released company-wide with CTO approval

### Situation

The firm’s equity and fixed income risk models were used by portfolio managers worldwide, delivered as client-server desktop software. The new platform was the move to the web, and it was late. I joined as it finished implementation and was asked to run the project retrospective.

Alongside the retrospective, I prepared the company for running a hosted service: audit-ready processes, terminology with Legal for storing trading and customer data, a support escalation procedure for the new technical support center, IT’s business continuity plan, and customer communications with Product.

### What I did

The culture was consensus-driven and analytical, and people were slow to speak up in groups, so I interviewed each team member individually. I sent each person my notes to confirm I’d captured them accurately, then wrote the report close to their own words, with identifying details removed. Complaints that recurred were weighted higher, so the most common problems rose to the top: chief among them, scope creep from requirements added well beyond the release, and decisions that could be quietly reversed after the meeting that made them.

### Outcome

With the CTO’s approval, the report was released internally and spread across the company overnight. A year later, a former architect told me that report had made me a hero to the team.

# Talks

**Healthcare AI and data platform**Security architecture engagement\
Early 2020s, independent consulting

## One identity model for every cloud, vendor and customer site

Scope

Security architecture for a multi-cloud AI and data platform handling healthcare data

Reference build

AWS: IAM, Secrets Manager, RDS Postgres, ElastiCache, EKS, API Gateway, Lambda, GuardDuty, Terraform

Compliance

HIPAA, SOC 2

Slides

[View the deck](./agnostic-identity-management/README.md)

### The problem

The platform runs as many separate deployments across cloud instances, each remotely managed, updated and monitored from a central control cloud. Every deployment ingests, processes and stores sensitive healthcare data, so it needs strong controls at rest, in transit and during processing. Each cloud provider, vendor and customer site brings its own notion of who a user or service is.

### The design

I proposed a cloud-agnostic identity layer: map and sync users, service accounts, groups, roles and devices from every cloud provider, vendor and customer premises into one model, persisted in PostgreSQL with a cache in front. Zero trust sits on top, authenticating both the user or application and the device before any access is granted.

### Access control and keys

- Kubernetes RBAC synchronized with IAM roles and groups, so cluster access follows the same identity model.
- Continuous testing of IT controls to confirm roles and permissions are intact, with Terraform Enterprise automatically reverting drift by reinstating state.
- Keys and credentials rotated every 90 days by default, configurable per customer.
- Zero trust extended to EC2 instances, databases and EKS clusters, so only identified users and service accounts can reach them.

### Operating it

For remote monitoring, I specified Prometheus for observability, Grafana for dashboards and Slack for escalation. Compliance runs as continuous validation from the code repository plus external audit: an annual third-party HIPAA audit, a SOC 2 audit of internal IT controls, and a requirement that every release meets the security controls. Tooling included SD Elements, Nessus, Nightfall and Vanta, with an SBOM maintained for every container and image.

To align operations with the strategy, I proposed shifting security left: Secure Code Warrior training for engineers, annual security awareness training for everyone, and quarterly security testing.

**Global streaming media company**Infrastructure architecture design\
Early 2020s

## A multi-tenant clickstream pipeline from global apps to trusted reports

Recommendation

AWS data lake with AWS BI tooling and logging

Alternatives weighed

eazyBI, Adobe Analysis Workspace, Domo; Grafana, Prometheus, Splunk

Compliance

GDPR

Slides

[View the deck](./multi-tenant-clickstream-design/README.md)

### Requirements

- Capture click data from client applications in multiple geographic regions.
- Support a BI tool that queries the data warehouse, with automated report workflows.
- Support workflows that need extremely high integrity, such as financial reporting.
- Provide logging and monitoring across the pipeline.

### The design

I recommended an AWS data lake with AWS BI tooling and logging. The core of the design is a multi-tenant identity header stamped on every event at the source, so each click carries who it belongs to and where it came from before it ever reaches the lake:

```
Customer-ID | Region | Click data …
```

With tenant and region on every record, events can be partitioned, retained and governed per customer and per jurisdiction.

### Considerations

The design accounts for GDPR, high availability, synthetic data for testing, model training, gaps in the data, data lifecycle and log archival, and seasonality in viewing traffic.

### Cost tradeoffs

I compared options at three price points. For BI: eazyBI as a low-cost Jira plugin, Domo in the middle as a challenger on the Gartner Magic Quadrant, and Adobe Analysis Workspace (Omniture) at the top end. For logging: Grafana or Prometheus at low cost, and Splunk at the top end.

**Talk**Late 2010s

## Putting the “Sec” into DevOps

Topics

DevSecOps culture, OWASP Top 10 (2017), penetration testing, security governance

Format

Interactive talk with live “find the vulnerability” puzzles

Slides

[View the deck](./digital-instrumentation/README.md)

### The argument

Security in DevOps is a social problem before it’s a technical one. I traced it through the resistance I’ve seen at each stage of the industry: developers asking why they had to check code into version control at all, then why they had to branch, then why commit messages mattered once SOX IT controls arrived. Each control felt like friction until the incident that proved it necessary. My answer was to stop treating InfoSec as a gate and make it part of the team.

Have you hugged your InfoSec person today?

### What the talk covers

- The OWASP Top 10 (2017), each category illustrated with the kinds of findings real penetration tests in financial services turn up: a one-letter admin database password, unlimited login retries, XML external entity attacks reading other users’ data, default Tomcat directories left in production, admin access gained by deserializing a message, patches lagging behind known library vulnerabilities, and too little logging to notice any of it.
- How to track and score penetration test results so they get fixed: tagging findings in Jira by label so they can be sliced by team and severity, reporting issues through SD Elements, and scoring exploitability and severity on a 1-to-5 scale.
- Fixes that remove the social friction instead of adding process: scanning code diffs for secrets with git-secrets before they’re committed, and moving credentials into HashiCorp Vault behind its API.

### Hidden pictures

Rather than lecture on each vulnerability, I showed the audience real-looking code and asked them to find the flaw. One example:

```
SELECT * FROM Users
WHERE ((Username='1' or '1' = '1'))/*') AND (Password=MD5('$password')))
```

Other puzzles were lighter: which dog breed is tied to an SSL attack (the poodle, for POODLE), and which utensil protects sensitive data (salt, for password hashing). The goal was for engineers to leave recognizing these patterns in their own code reviews.

**Talk**Early 2020s

## Engineering excellence and digital instrumentation

Topics

Engineering leadership, developer experience, instrumentation patterns, built-in security

Format

Interactive, with questions welcome throughout

Panels

ThriveWise, on the Colonial Pipeline ransomware attack; Teguas, on the SVB failure

### Leadership philosophy

I lead as a servant leader: goal-oriented, focused on enablement and autonomy, reusing what works and improving what doesn’t. Following means listening, and leading means doing the work alongside the team. I’ve built and led teams of 30, 35 and 50, and a matrixed organization of 150.

Break down the walls.

An open culture, shared work and a cross-disciplinary approach, with shared frameworks, services and tools, make transparency automatic rather than a reporting exercise.

### The engineering experience

I treat delivery as one instrumented path, so every stage is visible from the one before and the one after:

### Instrumentation patterns

Like building blocks, these patterns snap into any platform and remove recurring toil:

- Automatic federation of identity across systems.
- Engineering sandboxes, so teams can experiment safely.
- Smart certificate renewal, before expirations become outages.
- Synthetic data, so testing never needs real customer records.

### Security baked in

Security belongs in the pipeline, not at the end of it: build pipelines scanning with SonarQube, Checkmarx and Black Duck; standard patterns for encryption at rest and in transit; the OWASP Top 10; multi-factor authentication; and secure coding practices reinforced with Secure Code Warrior training.

**Talk**Early 2020s

## eBPF for optimal cluster use

Topics

Kubernetes performance tuning on GPUs, eBPF tracing, kernel and runtime security

Tools

eBPF, gVisor (runsc), seccomp, eBPF LSM, Kubernetes RuntimeClass

Sources

eBPF Summit 2023; Brendan Gregg, *Systems Performance*

Slides

[View the deck](./ebpf-cluster-use/README.md)

### The problem

GPU capacity is scarce and expensive, and high-performance computing hardware even more so, with procurement cycles measured in years. When Kubernetes workloads share NVIDIA Ampere or Hopper GPUs, every wasted cycle costs real money, and Ampere benchmarks up to 4.5 times faster than Hopper, so a poorly tuned cluster gives back much of what the hardware paid for.

Kubernetes tuning comes down to the tradeoff between requests and limits for memory, CPU and hugepages. Set them too loose and you waste capacity; set them too tight and workloads thrash.

What is the optimal request versus limit to avoid thrashing?

### eBPF to the rescue

The extended Berkeley Packet Filter lets you observe the system from inside the kernel without changing application code, and find the answer with data instead of guesswork:

- At the Kubernetes layer: cgroups, event tracing, syscalls and namespaces.
- At the kernel and network layer: sockets, TCP, IP and network paths.
- Wakeup stacks, to see exactly why work is waiting.

### eBPF has its own security surface

The same kernel access that makes eBPF powerful makes it a target. The talk covers malicious kernel requests, LD_PRELOAD injection, and time-of-check to time-of-use (TOCTTOU) races against ptrace, seccomp and eBPF probes on syscall entry, along with eBPF LSM for security monitoring on stable kernels.

My recommended mitigations: use gVisor to trace a workload’s kernel calls, then lock those calls down with seccomp under the runsc runtime; run non-default workloads under a separate Kubernetes RuntimeClass; and terminate violating processes with an asynchronous SIGKILL.

## Get in touch

I’m open to senior cloud security, FedRAMP and platform leadership roles, preferably in person in the San Francisco Bay Area.

[kimzvik@gmail.com](mailto:kimzvik@gmail.com) | [LinkedIn](https://www.linkedin.com/in/kimzvik) | [GitHub](https://github.com/kimzvikwu)

Education: Bachelor’s Degree, University of California, Berkeley