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
3. What did you do?
4. Who are you?
5. What would you have done differently?
6. What kinds of issues did you run into?
* migrating jobs
* preventing downtime
* handling existing credentials with multi-instance jobs
* adding security where it didn't exist before with multi-instance jobs
7. What kinds of changes did you make in the migration?  Are there broad categories/types of changes that are useful to know about?
There are indeed categories of things that are useful to know about.
8. What techniques are useful for dealing with migrating jobs?
9. What techniques are useful for dealing with enabling security/adding certificates?
10. What techniques are useful for dealing with existing credentials?
11. What techniques are useful to prevent downtime?
12. Where can I find documentation on the process you went through and a step-by-step guide how do do it?
13. Are there IaaS-specific issues here?
No - bosh.
14. What is the meaning of 'migrated_from' and how is it used?

# Speech structure
1. Describe what we started with
2. Describe what we had to do
