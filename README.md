# aos-cd-jobs

This repository backs Jenkins jobs on a couple of Jenkins masters.

## Jenkins pipeline definitions under `jobs/`
 
An internal [Continuous Infrastructure Jenkins instance](https://atomic-e2e-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/) indexes Jenkinsfiles in the branches of this repository, which are generated from the Jenkinsfiles that live under the `jobs/` directory. Jobs under the `jobs/build/` directory are indexed at the [`aos-cd-builds`](https://atomic-e2e-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/aos-cd-builds/) grouping, while the jobs under `jobs/cluster/` are indexed at the general [`aos-cd`](https://atomic-e2e-jenkins.rhev-ci-vms.eng.rdu2.redhat.com/job/aos-cd/) grouping. A quick synopsis of these indexed jobs is as follows:

|          Job Name          | Description |
| -------------------------- | ----------- |
| `build/scheduled/ose`      | Runs build/ose daily. Presently used to build 3.6 for daily integration test environments.            |
| `build/scheduled/t-th`     | Runs build/ose every Tuesday and Thursday for particular builds of OCP.            |
| `build/ose`                | Main build task for OCP. Can build 3.3, 3.4, and so on up through the current ose/master branch. Also builds openshift-ansible artifiacts and jenkins images.            |
| `build/stage-to-prod`      | Promote RPMs from the staging repositories to the production repositories. |
| `build/make-puddle`        | Create an Atomic OpenShift puddle on `rcm-guest`. |
| `build/openshift-scripts`  | Builds RPMs and container images for the [OpenShift Online](https://github.com/openshift/online) team. |
| `build/ose-pipeline`       | R&D to replace build/ose shell scripts with Jenkins pipeline syntax.            |
| `build/stage-to-prod`      | Copies files from [latest/ in the enterprise online-stg](https://mirror.openshift.com/enterprise/online-stg/latest/) repo to [online-prod/lastest](https://mirror.openshift.com/enterprise/online-prod/latest/).<BR>Also copies files from [libra rhel-7-libra-stage](https://mirror.ops.rhcloud.com/libra/rhel-7-libra-stage/) to [libra's latest online-prod](https://mirror.ops.rhcloud.com/libra/online-prod/latest/) in a new directory based on the day's date.           |
| `build/refresh-images`     |             |
| `cluster/sprint-control`   | R&D task to drive full Spring cadence when human intervention is no longer required.            |
| `cluster/operation`        | Deletes/upgrades/installs specific Redhat clusters.            |
| `package-dockertested`     | Tests new Brew builds of Docker and tags them into a [mirror repo](https://mirror.openshift.com/enterprise/rhel/dockerextra/x86_64/os/Packages/) for use by the CI systems. |

## Jenkins Job Builder configuration under `jjb/`

Jenkins Job Builder definitions under the `jjb/` directory are not currently used to underpin any jobs, but were an investigation into how the JJB system was used by the AOS CI team to build and support CI jobs for the `openshift-ansible` repository.

## Custom XML Generator configuration under `sjb/`

A custom XML generator lives under the `sjb/` directory. This generator is meant to be a tightly scoped tool that would help us bridge the gap between monolithic scripts inside of Freestyle Jenkins Jobs and segmented Jenkins Pipelines driven by source-controlled Groovy scripts and libraries.

The generator understand a small set of `action`s, each of which is underpinned by a Python module under [`sjb/actions/`](sjb/actions). A configuration YAML file is read in by [`sjb/generate.py`](sjb/generate.py) and used to generate a set of input variables to the [Jinja job template XML](sjb/templates/test_case.xml). Jobs can depend on a parent to reuse configuration.

A typical workflow for a developer making changes to the job would look like:

 - make edits to a configuration file under `sjb/config/`
 - run `sjb/generate.sh`
 - commit changes
 - run `sjb/push-update-automatic.sh` once changes are approved and merged into `master`

In order to test a job, it is necessary to copy a configuration file under `sjb/config` to a new YAML file with a different name, then re-generate XML and use the following command to push only your test job up to the server:
```shell
sjb/push-update.sh sjb/generated/YOUR_TEST_JOB.xml
````
Cleanup of these jobs post-test is still manual.

Note: the `sjb/push-update{,-automatic}.sh` scripts expect `$USERNAME` and `$PASSWORD` to be set as envars when they are run. These are the credentials with which you log in to the Jenkins master at [ci.openshift](http://ci.openshift.redhat.com/) and are used for basic auth against the server on push actions.
