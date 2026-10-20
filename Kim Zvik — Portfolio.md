**Kim Zvik**

- [VANQUISH](#vanquish)
- [Case studies](#case-studies)
- [Contact](#contact)

# Kim Zvik

Cloud and infrastructure security leader. 15+ years turning operational gaps into programs that hold up under audit, outage and scale.

- Kensington, CA
- [kimzvik@gmail.com](mailto:kimzvik@gmail.com)
- [linkedin.com/in/kimzvik](https://www.linkedin.com/in/kimzvik)
- [github.com/kimzvikwu](https://github.com/kimzvikwu/vanquish)

My work, logged the way I’d log a finding: what was wrong, and how it got closed. Select a row to read the full case study.

| Ref | Years | Organization | Finding | Status |
| --- | --- | --- | --- | --- |
| KZ-00 | 2026 | [VANQUISH](#vanquish) | FedRAMP vulnerability data scattered across scanners, spreadsheets and manual POA&M reports | Built |
| KZ-01 | 2021–2022 | [FDIC](#fdic) | Two prior cloud migrations failed; 35 engineers stuck in one meeting cadence | Resolved |
| KZ-02 | 2018–2020 | [Salesforce](#salesforce) | 13 teams running divergent, unpatched forks of Apache ZooKeeper under critical systems | Resolved |
| KZ-03 | 2015–2017 | [Moody’s Analytics](#moodys) | HTTPS rollout silently failing in Europe and Asia, blocking secure collaboration | Resolved |
| KZ-04 | 2012–2015 | [Western Union Digital](#western-union) | 25,000 send-money paths tested by hand over four days per release | Resolved |
| KZ-05 | 2001–2003 | [MSCI Barra](#msci) | A late flagship web launch with no shared account of why it ran late | Resolved |

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

VANQUISH also includes a DLP sandbox component. I’ve applied the same agent design to real vulnerability remediation work at Keeper Security and Mercor.

[View the code on GitHub](https://github.com/kimzvikwu/vanquish)

**FDIC**Senior Director, DevOps\
2021–2022, via Vivsoft

## Restructuring a stalled Azure migration so engineers could build

Scale

$5M contract; 35 contractors hired in 4 weeks

Platform

Microsoft Azure

Outcome

New team model still in place a year later; contract extended past its checkpoint

### Situation

The FDIC had to close an on-premises data center by the end of 2023. Two earlier cloud transformations, one on AWS and one on Tanzu, had failed. Vivsoft won a $5M contract to move the agency’s infrastructure to Azure, and I led the delivery.

### Building the team

We had four weeks to hire 35 contractors to meet our first milestone. I ran interviews alongside three architects and assigned each hire to the architect who wanted to lead them, so the structure grew organically. When a sourcing vendor challenged a decision, we formalized our interview questions with the vendor and HR so the process was fair and the hiring bar held.

### What I found

All 35 engineers and five product managers sat in a single, long sprint-grooming meeting every two weeks. Roughly 80% of the team’s time went to ceremony instead of engineering. Engagement dropped, debates got heated, and strong engineers started looking elsewhere. Talking to the product managers individually, I learned they wanted more autonomy, and I came to see the meeting structure as a likely contributor to the earlier failures.

### What I did

I reorganized the program into mini teams, each aligned to a single product manager and the FDIC department that manager represented. Each project manager and business analyst carried at most two teams. I socialized the org chart with my CTO first, then with the FDIC, and framed the change for agency leadership in terms of what it saved in meeting cost, since every dollar the FDIC spends is answerable to taxpayers.

### Outcome

The FDIC agreed to trial the structure, and it was still in place a year later. My CTO adopted the org chart as the company’s de facto reporting structure, and Vivsoft’s contract continued past its November checkpoint.

**Salesforce**Principal Engineer, Core Infrastructure\
2018–2020

## Making the case for a centralized ZooKeeper team before the outages made it for me

Context

Storage Engineering, Salesforce Database (Postgres-based Oracle replacement)

Tech

Apache ZooKeeper, mTLS, Puppet, NIST runbooks

Outcome

Centralized ZooKeeper team created after the proposal

### Situation

I joined a startup-style team of about 100 building the Salesforce Database, a Postgres fork meant to replace the company’s Oracle dependency and save millions a year in licensing. The database couldn’t run without Apache ZooKeeper, and I led the DevOps work supporting it. ZooKeeper wasn’t well understood on my team or across the company.

### What I found

To anticipate what my team would face, I studied ZooKeeper outages elsewhere at Salesforce. Thirteen teams depended on it, each running a slightly different version, and none had contributed their changes back upstream, so every upgrade meant porting and retesting proprietary changes. We were a major version behind the stable release, and that newer release fixed race conditions and performance problems that matched many of the outages I’d catalogued.

I also noticed that public contributions to an open source project can reveal which companies run it, and at which version. I treated that as an exposure worth weighing when deciding what to contribute.

### What I did

- Worked with the other 12 teams on a joint proposal for a centralized ZooKeeper team.
- Built relationships with the organizers of the annual ZooKeeper meetup, which led to Salesforce engineers speaking on our C++ service discovery module and on mTLS, to help recruit scarce talent.
- Configured our ZooKeeper fleets for mTLS using Puppet, alongside the cybersecurity Fellow who handled certificate issuance and enforcement, a requirement for federal customers.
- When asked to pause the proposal, wrote the ZooKeeper runbooks for our air-gapped federal installations and NIST requirements, published as Markdown in GitHub rendered to Confluence.

### Outcome

Later that year, a split-brain incident hit our production pilot, and a race condition in another team’s cluster caused a production-wide outage. Because the proposal had circulated, teams turned to us. I used that moment to make the case again with the affected team’s leadership. After I moved on to FICO, Salesforce created a centralized ZooKeeper team.

**Moody’s Analytics**Senior DevOps Engineer\
2015–2017

## Tracing a silent TLS handshake failure across a global WAN

Systems

Confluence, Jira, Bitbucket; Riverbed SteelHead WAN appliances

Tools

Wireshark, ServiceNow

Outcome

HTTPS enforced globally; InfoSec adopted the platform

### Situation

I administered Moody’s Analytics’ Atlassian platform for offices across North America, Europe and Asia, and migrated a MediaWiki holding more than 20 years of intellectual property, including research on how credit ratings are calculated, into Confluence. An information security engineer refused to join until the sites used HTTPS, rightly: over plain HTTP, his credentials would travel in clear text.

### What I found

HTTPS worked in North America but failed in Europe and Asia. Coming from software engineering rather than networking, I taught myself Wireshark, mapped the healthy handshake as a state diagram, and compared it against traces from the failing regions. The client hello went out; the acknowledgment never came back.

Over six months I opened 20 tickets. Each was closed without anyone contacting me.

The firewall team was measured on SLA, and the issue wasn’t theirs, but the new ServiceNow process had no escalation path to route it elsewhere while keeping it open. During an unrelated outage bridge, I documented every hop by name and IP, then worked with a security network engineer to pinpoint the drop: the Riverbed SteelHead WAN optimization appliances needed a rule to handle TLS rather than only legacy SSL.

### What I did

Networking leadership initially declined the change. I brought in Information Security for a second opinion, they backed the diagnosis, and the rule was applied across the appliances.

### Outcome

Every Atlassian site moved to HTTPS, and the security team started using the platform. Because I documented the fix in Confluence, my manager was able to apply the same change to his own servers a year later from a single link.

**Western Union Digital**Head of Quality and Release Engineering\
2012–2015

## Replacing four days of manual release testing with one dashboard of truth

Scale

\~50-person team; 25,000 send-money test paths; web, iOS and Android

Tools

Jira, Zephyr, Confluence, Sauce Labs, Jenkins

Recognition

Employee of the Month

### Situation

Western Union was building a new digital technology division in San Francisco. I inherited a team of about 50 contractors, mostly in India, who manually tested the 25,000 ways a customer can send money. A release took four days of testing before being handed over the wall to IT Operations.

### What I found

Whole scenarios couldn’t be tested at all. Offshore testers had no access to certain US card types or to many of the Android models customers actually used. The mobile app carried a two-star rating, largely because it misbehaved across Android devices.

### What I did

- Replaced HP Quality Center with Jira for the division.
- Drawing on my device-testing work at Verizon Wireless, persuaded corporate to provision 40 phone models, against a one-phone-per-employee policy.
- When budget for mobile test automation was denied, secured funding from the VP of Marketing, who owned the app rating, and kept my SVP informed.
- Worked with our security manager to approve firewall access for external test VMs and to let a locked device lab run on the network without employee authentication.
- Split the 25,000-path web suite across parallel VMs to run four times faster.

### Outcome

I designed a Jira dashboard with three levels of detail that aggregated manual, automated web (Sauce Labs via the Zephyr API) and mobile results side by side, showing per build whether it was ready to release. When overnight offshore testing surfaced new bugs, I called it the offshore gift: better our testers find them than our customers.

**MSCI Barra**Senior Program Manager\
2001–2003

## A project retrospective people trusted enough to be honest in

Product

BarraOne, the firm’s first web-based portfolio analytics platform

Partners

Audit, Legal, IT, Engineering, Product

Outcome

Report released company-wide with CTO approval

### Situation

Barra’s equity and fixed income risk models were used by portfolio managers worldwide, delivered as client-server desktop software. BarraOne was the move to the web, and it was late. I joined as it finished implementation and was asked to run the project retrospective.

Alongside the retrospective, I prepared the company for running a hosted service: audit-ready processes, terminology with Legal for storing trading and customer data, a support escalation procedure for the new technical support center, IT’s business continuity plan, and customer communications with Product.

### What I did

The culture was consensus-driven and analytical, and people were slow to speak up in groups, so I interviewed each team member individually. I sent each person my notes to confirm I’d captured them accurately, then wrote the report close to their own words, with identifying details removed. Complaints that recurred were weighted higher, so the most common problems rose to the top: chief among them, scope creep from requirements added well beyond the release, and decisions that could be quietly reversed after the meeting that made them.

### Outcome

With the CTO’s approval, the report was released internally and spread across the company overnight. A year later, a former architect told me that report had made me a hero to the team.

## Get in touch

I’m open to senior cloud security, FedRAMP and platform leadership roles, preferably in person in the San Francisco Bay Area.

[kimzvik@gmail.com](mailto:kimzvik@gmail.com) | [LinkedIn](https://www.linkedin.com/in/kimzvik) | [GitHub](https://github.com/kimzvikwu)

Education: Bachelor’s Degree, University of California, Berkeley