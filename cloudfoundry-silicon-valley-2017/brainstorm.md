# Audience

Bosh operators with complex deployments

# Purpose
Share a series of lessons learned doing the migration of CF+Diego -> CF-Deployment.

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
    - The Release Integration is an Open Source team responsible for [pithy statement here].  We own:
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
    * database job gets replaced, resulting in data loss without migrated_from

8. What kinds of changes did you make in the migration?  Are there broad categories/types of changes that are useful to know about?
    - There are indeed categories of things that are useful to know about:
        - Understanding the semantics of `static` and `reserved` IP address ranges, and how they affect the pre- and post-migration jobs.  Some jobs we wanted to keep alive during the migration had IP addresses that needed to be reserved to prevent collisions.  Which jobs those were and why we needed to keep them alive is a mystery that we want to investigate for this talk. ??
    - Ensuring datastore (etcd/mysql) continuity during the upgrade so you don't lose applications
        - Migrating Diego from `etcd` -> `mysql` before migration [outline the steps to do this here]
        - Ensuring continuity through migration via Bosh's `migrated_from`
    - How we learned to stop worrying and love dual Diegos
    - Principled application of Bosh `vars` vs `ops`
    - Backups are absolutely necessary before embarking on this mission or else you may lose everything and be sad

9. What changes are required when migrating from a bosh 1.0 manifest to a bosh 2.0 manifest?
    - cloud-config has been moved to a separate file that must be provided to configure your IaaS settings
    - deployment manifests should not be IaaS-specific anymore
    - templates have been renamed to jobs
    - targets have been renamed to environments
    - valid certificate required on Bosh director in order to use the CLI against it
    - instance_groups now are used to define VMs which will be created and may specify a list of AZs instead of having to specify a different virtually-identical VM (to have it on a different AZ) like we did with bosh 1.0??

10. What do you get from migrating to bosh 2.0?
    - bosh links
        - allow bosh to manage IP addresses for you so you no longer have to specify and use static IP addresses to connect jobs (like mysql and dependent apps) to one another
        - allow jobs to provide and consume properties so you no longer have to maintain duplicate configuration (like password configuration so e.g. mysql and dependent apps agree on a password value) across jobs
    - bosh variables
        - allow bosh to generate passwords, signed certificates, and keys for your deployment so you no longer have to generate these values by hand
        - with with the CLI's `--var-errs`, can be used to quickly find problems in your manifest where any variable has not been specified (all variables are considered necessary for deployments to succeed)
    - bosh ops files
        - allow repeatable in-place modifications of manifests so transformations to manifests can be shared as their own kind of tool (e.g. an ops file to deploy on GCP, or bosh-lite, or an ops-file to remove all secrets for example)
    - bosh links!  Link all the things!

11. Is it necessary to migrate from bosh 1.0 to bosh 2.0 to do what you did?
    - If you interpret "do do what you did" to mean "migrating from separate CF an diego deployments to a single deployment" then strictly speaking, no.  However, the real task we had was to migrate from cf-release (our current git-submodule-based release strategy) to cf-deployment (our new manifest-based release awesomesauce). So, Yes?

12. What techniques are useful for dealing with migrating jobs?
    - keep all the job names the same
    - use migrated_from for everything to keep things alive rather than tearing down and creating new for all jobs

13. What techniques are useful for dealing with enabling security/adding certificates?
    - use bosh --var-store to generate credentials if possible, because it will automatically generate them for you making your life easier.
    - make sure to specify correctly the ext key usage properties for servers, clients, or both as required by each job.  This may require some trial and error unless you are using cf deployment (which already has been customized this way). Feel free to use it as an example for your own needs.
    - make sure that all domain names are properly specified for server certs as alternate names or else the certificate will be rejected by clients
    - you can use bosh logs -f | grep --line-buffered -i tls to help discover problems with certificates throughout your entire deployment when reproducing errors

13. What techniques are useful for dealing with rotating certificates?
    - when rotating credentials, you need to do two deployments; first, scale clustered jobs down to a single instance to avoid problems such as etcd split-brain; then, scale back up.

14. What techniques are useful for dealing with existing credentials?
    - Keep all of your credentials the same to avoid introducing credential rotation problems while dealing with the migration, to make sure you are not trying to debug multiple kinds of errors at once.

15. What techniques are useful to prevent downtime?
    - use migrated_from properly(?) to specify jobs that are being migrated to prevent them from being torn down and recreated, which would result in downtime
    - consider duplicating your entire deployment and migrating one side using a load balancer to keep traffic going to the other side.
    - consider separating your database out into a separate deployment so it doesn't need to be redeployed at the same time?
    - ask pivotal tracker team about what they be doing when migrating to PWS?

16. Where can I find documentation on the process you went through and a step-by-step guide how do do it?
[ ] Insert link to CF deployment migration document that we are updating to answer this question

17. Are there IaaS-specific issues here?
    - No - bosh.
18. What is the meaning of 'migrated_from' and how is it used?
    - 'migrated_from' allows bosh to recognize a relationship between jobs that have been renamed or moved and so would be difficult otherwise for bosh to migrate.  This is important to add in order to maintain data and uptime.
19.  What if we had all separate deployments?

# Speech structure
1. Describe what we started with
2. Describe what we had to do
