# Upgrading Pulp

This document describes how to upgrade to a new version of Pulp. **Benchmarks are
optional but highly recommended.**

## Requirements

This assumes you have a working Katello installation and that you've checked
out all the necessary repositories including runcible.

0. It's probably a good idea to get some benchmarks before upgrading. This is
optional though if the change in pulp versions brings minor changes. See step 6
for a good idea of what to benchmark.

1. First, follow the pulp documentation on how to install the latest version
(https://pulp-user-guide.readthedocs.org/en/latest/installation.html). Once you
have the latest version installed, make you sure you update or reset the
database. 

2. Upgrade runcible by running the VCR tests in live mode
(https://github.com/Katello/runcible#testing). Fix any broken tests.
Optionally, run some manual commands from a console to test pulp's
functionality. Once you're finished making changes, merge them into master.

3. If runcible changes were needed, Release a new version of runcible
(https://github.com/Katello/runcible#building-and-releasing) but do NOT build
new rpms in koji.

4. Upgrade Katello to your new version of runcible. Run the VCR tests in
Katello in live mode. Fix any failures. 

6. Perform the following tests. If you took benchmarks in step 0, compare the
times:
    * Create a yum repository with a URL and sync it.
    * Create a puppet repository and upload some puppet modules for it.
    * Enable a Red Hat repo (6server) and sync it.
    * Create a content view with these repositories. Create a filter and publish the content view.
    * Promote the content view to an environment other than Library. Confirm the content is on the filesystem and perform a content search.
    * Register a consumer and perform some package action
    * Ensure that errata applicability generation works

7. Fix any failures from above.

8. Do the following within a short time frame:
     * Update runcible requirement in katello.gemspec
     * Build new runcible rpm
     * Merge open PR for new runcible requirement and any katello change above
     * tag the following packages into el6

