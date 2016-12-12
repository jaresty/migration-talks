# Audience

* Bosh operators with complex deployments

# Purpose
* Share a series of lessons learned doing the migration of CF+Diego -> CF-Deployment.

# Questions
1. How complex is your environment, really?

Start state: Two deployments (CF and Diego) with multiple instances of VM types across 2-3 access zones.
- MySQL cluster (Gallera) ?
- Consul cluster (ssl?)
- etcd cluster (ssl?)

End state: One deployment (CF _with_ Diego), with multiple instances of VM types across 3 access zones.
- MySQL cluster (Gallera)
- Consul cluster (ssl)
- etcd cluster (ssl)

2. What did you learn?
- It's challenging to turn on SSL or rotate certificates in applications that have multiple bosh-managed instances using SSL because new instances which are brought up with a new CA do not trust the old instances which still rely on the prior CA for communication. We have found that it is necessary to scale down instances to 1 and then scale back up whenever we want to rotate certificates for clustered jobs like etcd or consul.
  - Should we submit an enhancement request to bosh to better support this?
- It is required to specify migrated_from in order to not have downtime during the migration when moving jobs from one deployment to another or renaming jobs
- To limit the kinds of problems you encounter to only migration problems:
  - It's helpful to try and keep credentials identical during the migration if you can and only rotate them later (as opposed to also including authentication issues)
  - It's helpful to try and keep versions of jobs as close as possible on both sides of the migration in order to limit the kinds of problems you encounter
3. Why are those insights at all useful?  Aren't they obvious?
- No.  We ran into each of these problems in turn, spending weeks in the process of debugging our migration.
4. What did you do?
- Migrated separate CF and Diego deployments into a single collocated CF+Diego deployment.
- We iterated on doing this migration over weeks running into various deployment issues that we resolved using the strategies we mentioned in the "what did you learn" section.
5. Who are you?
- The Release Integration team is an Open Source team responsible for [pithy statement here].  We own:
  - CATs: The Cloud Foundry Acceptance Tests
  - CF Release: Our current recommended release mechanism for Cloud Foundry
  - CF Deployment: Our next-gen recommended deployment of Cloud Foundry (it's boss)
  - Nats: Opens Source message bus used in Cloud Foundry
  - The CI pipeline: Ensures all changes to CF are tested together before release
  - Often the first point of contact for Open Source users dealing with CF issues via GitHub
  - We are Bosh User 0.1 (we may not be the first to consume new Bosh features, but we're among the vanguard)
6. What would you have done differently?
- We would have followed these guidelines.
- We would have issued a request for a certificate rotation feature in Bosh earlier
7. What kinds of issues did you run into?
* migrating jobs
* preventing downtime
* handling existing credentials with multi-instance jobs
* adding security where it didn't exist before with multi-instance jobs
8. What kinds of changes did you make in the migration?  Are there broad categories/types of changes that are useful to know about?
There are indeed categories of things that are useful to know about.
9. What changes are required when migrating from a bosh 1.0 manifest to a bosh 2.0 manifest?
10. What do you get from migrating to bosh 2.0?
11. Is it necessary to migrate from bosh 1.0 to bosh 2.0 to do what you did?
12. What techniques are useful for dealing with migrating jobs?
13. What techniques are useful for dealing with enabling security/adding certificates?
14. What techniques are useful for dealing with existing credentials?
15. What techniques are useful to prevent downtime?
16. Where can I find documentation on the process you went through and a step-by-step guide how do do it?
17. Are there IaaS-specific issues here?
No - bosh.
18. What is the meaning of 'migrated_from' and how is it used?

# Speech structure
1. Describe what we started with
2. Describe what we had to do
