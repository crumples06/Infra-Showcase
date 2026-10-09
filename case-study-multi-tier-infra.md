# Case Study: Provisioning, Hardening, and Monitoring a Multi-Tier Infrastructure from Scratch

## The Problem

Most infrastructure work isn't glamorous — it's making sure that when a server gets rebuilt, when a service crashes, or when someone new joins the team, nothing has to be figured out from memory. I wanted to prove I could build that kind of infrastructure end-to-end: provisioned automatically, secured by default, and observable enough that a failure shows up on a dashboard instead of in a support ticket.

So I built a five-tier system — two web servers behind a load balancer, a database, and a monitoring stack — and treated it exactly like production infrastructure, even though it was a personal project. Every host is provisioned from a single command. Every configuration change is version-controlled. Every claim about "it recovers automatically" is backed by a recorded demo, not a description.

## The Approach

**Everything is defined as code, not clicked together.** Ansible playbooks and roles describe the desired state of every host — install this, configure that, start this service — so the entire five-host environment can be torn down and rebuilt from nothing in one command. Docker containers stand in for real servers, connected over their own network, so the setup can be tested and demonstrated without touching real infrastructure or costing anything to run.

**The build order mirrored how I'd actually prioritize a client engagement.** I got the core system working first — web tier, load balancer — then deliberately paused feature work to add security hardening (SSH key-only auth, disabled root login, disabled password auth) before touching the database tier. The reasoning: a database holding real data is exactly the kind of thing you don't want sitting on a host you haven't hardened yet. Observability (Prometheus + Grafana) came last, once there was something worth monitoring.

**Secrets are never stored in plain text.** Database and dashboard passwords are encrypted with Ansible Vault and only decrypted at deploy time with a password that's never committed to the repository — including in the automated CI pipeline, where it's injected from a secrets store and destroyed the moment the job ends.

**Every change is verified automatically before it can reach the main branch.** A GitHub Actions pipeline rebuilds the entire environment from scratch on every pull request, runs the full deployment, and checks that traffic is actually load-balanced across both web servers before anything can be merged. 

## Proof It Works, Not Just a Claim

The best way to show infrastructure is resilient is to break it on purpose and record what happens. So I did: with the full system running normally, I stopped the web server on one of the two web hosts mid-traffic — simulating exactly the kind of failure that happens in production, at the worst possible time.

- **Before:** requests alternate evenly between both web servers.
- **During the failure:** the load balancer detects the failed host and routes every request to the healthy one automatically — zero dropped requests, zero manual intervention.
- **After recovery:** traffic resumes across both servers automatically, no configuration changes needed.


## The Outcome

A monitored, security-hardened, multi-tier deployment that can be rebuilt from nothing in one command, verified automatically on every change, with its own recorded incident response demo.  

