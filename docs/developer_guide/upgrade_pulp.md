# Upgrading Pulp

This document describes how to upgrade to a new version of Pulp. **Benchmarks are
optional but highly recommended.**

## Requirements

This assumes you have a working Katello installation and that you've checked
out all the necessary repositories including Runcible (https://github.com/Katello/runcible).

0. We recommend capturing some benchmarks before upgrading. This is
optional though if the change in Pulp versions brings minor changes. See step 6
for a good idea of what to benchmark.

1. Follow the Pulp documentation on how to install the latest version
(https://pulp-user-guide.readthedocs.org/en/latest/installation.html). If upgrading
from a prevous installation, typically a 'yum update *pulp*' will suffice.  Once you
have the latest version installed, make you sure you update or reset the
database. (Its a good idea to keep track of all packages installed as part of this upgrade,
as it will package tagging easier in step 8)

2. Upgrade Runcible by running the VCR tests in live mode
(https://github.com/Katello/runcible#testing). Fix any broken tests.
Optionally, run some manual commands from a console to test Pulp's
functionality. Once you're finished making changes, merge them into master (going through the
Pull Request process).

3. If Runcible changes were needed, release a new version of Runcible
(https://github.com/Katello/runcible#building-and-releasing) but do NOT build
new rpms in koji.

4. Upgrade Katello to your new version of Runcible. Run the Katello VCR tests
in live mode. Fix any failures.

5. Perform the following tests. If you took benchmarks in step 0, compare the
times:
    * Create a yum repository with a URL and sync it.
    * Create a puppet repository and upload some puppet modules for it.
    * Enable a Red Hat repo (6server) and sync it.
    * Create a content view with these repositories. Create a filter and publish the content view.
    * Promote the content view to an environment other than Library. Confirm the content is on the filesystem and perform a content search.
    * Register a consumer and perform some package action.
    * Ensure that errata applicability generation works.

6. Fix any failures from above.

7. Do the following within a short time frame:
     * Update Runcible requirement in katello.gemspec.
     * Build new Runcible rpm.
     * Merge open PR for new Runcible requirement and any katello changes from the fixes from steps 4 & 5.
     * Tag into these katello tags:
       * katello-thirdparty-pulp-rhel6
       * katello-thirdparty-pulp-rhel7
       * katello-thirdparty-pulp-fedoraXX:

      The following packages:
       * pulp
       * pulp-rpm
       * pulp-puppet
       * pulp-nodes
       * gofer (if applicable)

      Occasionally other dependencies (such as these https://github.com/pulp/pulp/tree/master/deps) are needed.
      If you kept track of what was updated as part of your upgrade and you noticed any new deps pulled in from
      the pulp repositories, tag those in now.
     * In order for the client repo to be populated properly, tag the new pulp-rpm package into the following tags:
      * katello-nightly-rhel6
      * katello-nightly-rhel7
      * katello-nightly-fedoraXX

     * Sample tagging commands:
       ```
            koji-katello  tag-pkg  katello-thirdparty-pulp-rhel6   pulp-2.4.0-0.23.beta.el6 pulp-rpm-2.4.0-0.23.beta.el6  pulp-puppet-2.4.0-0.23.beta.el6 pulp-nodes-2.4.0-0.23.beta.el6  gofer-1.3.0-1.el6
            koji-katello  tag-pkg  katello-thirdparty-pulp-rhel7   pulp-2.4.0-0.23.beta.el7 pulp-rpm-2.4.0-0.23.beta.el7  pulp-puppet-2.4.0-0.23.beta.el7 pulp-nodes-2.4.0-0.23.beta.el7  gofer-1.3.0-1.el7
            koji-katello  tag-pkg  katello-thirdparty-pulp-fedora19   pulp-2.4.0-0.23.beta.fc19 pulp-rpm-2.4.0-0.23.beta.fc19  pulp-puppet-2.4.0-0.23.beta.fc19 pulp-nodes-2.4.0-0.23.beta.fc19  gofer-1.3.0-1.fc19
            koji-katello tag-pkg katello-nightly-rhel6 pulp-rpm-2.4.0-0.23.beta.el6
            koji-katello tag-pkg katello-nightly-rhel7 pulp-rpm-2.4.0-0.23.beta.el6
            koji-katello tag-pkg katello-nightly-fedora19 pulp-rpm-2.4.0-0.23.beta.fc19
       ```
8.  After 4 hours have passed and the new pulp rpms have hit the katello repos, perform a test install to ensure
    there are no issues.

9. Enjoy your new pulp build and congratulate yourself for making it this far!