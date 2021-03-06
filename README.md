# Open Innovation Labs Big Data

The goal of this repository is to:

 1. provide an integrated collection of common tools for data analysis on OpenShift
 2. serve as a reference implementation of the [openshift-applier](https://github.com/redhat-cop/casl-ansible/tree/master/roles/openshift-applier) model for Infrastructure-as-Code (IaC) 

A few additional guiding principles:

* This repository doesn't handle CI/CD infrastructure. Please see [Labs CI/CD](https://github.com/rht-labs/labs-ci-cd)
* This repository is built as a monolith i.e. all the individual components are designed to be deployed together. When adding a new tool, care should be taken to integrate that tool with the other tools. 
* To deploy a subset of components, you can:
  * Use the tags provided in the inventory to filter certain components. See [the docs](https://github.com/redhat-cop/casl-ansible/tree/master/roles/openshift-applier#filtering-content-based-on-tags) in openshift-applier
  * _coming soon_ - Use the provided tooling to generate a new inventory with a user defined subset of components

  
## How it Works

There is ansible inventory which identifies all components to be deployed to an OpenShift cluster. The ansible layer is very thin. It simply provides a way to orchestrate the application of [OpenShift templates](https://docs.openshift.com/container-platform/3.6/dev_guide/templates.html) across one or more [OpenShift projects](https://docs.openshift.com/container-platform/3.6/architecture/core_concepts/projects_and_users.html#projects). All configuration for the applications should be defined by an OpenShift template and the corresponding parameters file. 

Currently, the following components are included in this inventory:

* Resources from radanalytics.io 
  * [Generic OpenShift templates](https://radanalytics.io/get-started) 
  * [Jupyter Notebook Demo - PySpark 2](https://radanalytics.io/examples/var) 
    * note: this deploy is single tenant - i.e. each user needs their own notebook deployed. work on Jupyter Hub will resolve this
  * [Java/Spring Spark Demo](https://radanalytics.io/assets/my-first-radanalytics-app/sparkpi-java-spring.html) & [Python/Flask Spark Demo](https://radanalytics.io/assets/my-first-radanalytics-app/sparkpi-python-flask.html), both of which do the following, just using different languages/frameworks:
    * dynamically spin up a Spark cluster (via Oshinko CLI) when the app boots in OpenShift
    * create a Spark session on the new cluster and calculate `Pi`
* Explorations from Justin Holmes
  * [Jupyter PySpark 3](https://github.com/sherl0cks/jupyter-notebook-py3.5)
    * this is a temporary build until https://github.com/radanalyticsio/base-notebook/issues/12 is resolved
    


TODO
* Oshinko WebUI, which currently [does not work with `oc apply`](https://github.com/radanalyticsio/oshinko-s2i/issues/157)

## Prerequisites

* [Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html)
* [OpenShift CLI Tools](https://docs.openshift.com/container-platform/3.6/cli_reference/get_started_cli.html)
* Access to the OpenShift cluster 

## Usage 

1. Log on to an OpenShift server `oc login -u <user> https://<server>:<port>/`
    1. Your user needs permissions to deploy ProjectRequest objects
2. Clone this repository
3. Install the required [casl-ansible](https://github.com/redhat-cop/casl-ansible) dependency
    1. `[labs-big-data]$ ansible-galaxy install -r requirements.yml --roles-path=roles`
4. Run the ansible playbook provided by the casl-ansible
    1. `[labs-big-data]$ ansible-playbook roles/casl-ansible/playbooks/openshift-cluster-seed.yml -i inventory/`

After running the playbook, the pipeline should execute in Jenkins, build the spring boot app, deploy artifacts to nexus, deploy the container to the dev stage and then wait approval to deploy to the demo stage. See Common Issues


## Running a Subset of the Inventory 

See [the docs](https://github.com/redhat-cop/casl-ansible/tree/master/roles/openshift-applier#filtering-content-based-on-tags) in casl-ansible

## Layout
- `inventory`: a standard [ansible inventory](http://docs.ansible.com/ansible/latest/intro_inventory.html). 
  - the `group_vars` are written according to [the convention defined by the openshift-applier role](https://github.com/redhat-cop/casl-ansible/tree/master/roles/openshift-applier#sourcing-openshift-object-definitions).
  -  the `hosts` file reflects the fact that the playbook will use the OpenShift CLI on your localhost to interact with the cluster
- `openshift-templates`: a set [OpenShift templates](https://docs.openshift.com/container-platform/3.6/dev_guide/templates.html) to be sourced from the inventory. OpenShift provides a lot of templates out of the box, and [the Labs team curates a repository](https://github.com/rht-labs/labs-ci-cd/tree/master/templates) as well. These should be favored before writing custom/new templates to be kept here.
- `params`: a set of [parameter files](https://docs.openshift.com/container-platform/3.6/dev_guide/templates.html#templates-parameters) to be processed along with their respective OpenShift template. the convention here is to group files by their application.

## Common Issues

- S2I Build fails to push image to registry with `error: build error: Failed to push image: unauthorized: authentication required`
  - See [this issue](https://github.com/openshift/origin/issues/4518)

## License
[ASL 2.0](LICENSE)