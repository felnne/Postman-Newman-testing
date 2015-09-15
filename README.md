
# Postman Newman testing

Master - [![Build Status](https://semaphoreci.com/api/v1/projects/d8c4baad-0adf-4c81-9c37-4e0cdfbb7c01/540592/badge.svg)](https://semaphoreci.com/felnne/postman-newman-testing)
Develop - [![Build Status](https://semaphoreci.com/api/v1/projects/d8c4baad-0adf-4c81-9c37-4e0cdfbb7c01/540590/badge.svg)](https://semaphoreci.com/felnne/postman-newman-testing)

Simple project to use Postman's [Newman](https://github.com/postmanlabs/newman/) tool to test the BAS People API.

## Background

This project is part of a project to add a testing and deployment workflow to BAS APIs in order to reduce manual testing/verification of changes, improve the speed changes are delivered and improve the robustness of our services.

The tests performed by Newman will need to executed by *something*, ideally this would be some sort of Continuous Integration (CI) service, such as [Semaphore](semaphoreci.com) or [Bamboo](https://www.atlassian.com/software/bamboo), though manual testing will still be needed.

This API testing will form part of a Continuous Deployment (CD) process (i.e. only deploy if tests are green). Ideally the CI and CD process would both be performed by the same service (in additional to manually) to reduce costs and configuration.

Currently, Newman's tests can be run:

* Manually*, through a *local VM** - see this README for setup & usage instructions
* Automatically*, through **Semaphore CI** - see this [project](https://semaphoreci.com/felnne/postman-newman-testing/) and this README for additional comments

This project is already a cautious success in that it has highlighted one major error (API tokens cannot be blacklisted) and several minor defects (incorrect status codes being returned etc.).

### Why do these tests always fail?

These tests include assessing the performance of the API, which, as the results of these tests show, isn't currently very good.

Until this can be addressed **it is expected** when these tests are run together they will fail (i.e. if any one test fails, they all collectively fail).

However, this only applies to the *performance* related tests, all the other *functional* tests should pass [1].

### Project management

This project will be developed using the Government Digital Service (GDS) design phases methodology [2].

This project is in *Alpha*, when it reaches *Beta* it most likely be merged into this larger project. It is therefore not expected this project will move beyond *Beta* and move straight to *Retirement*.

The Project Maintainer for this project is: [Felix Fennell](mailto:felnne@bas.ac.uk) [3].

[1] Pedantic readers will spot that the test to 'blacklist a token' always fails - this is a genuine bug with the People API and needs resolving. It is not a bug with these tests and demonstrates how badly we've needed this!

[2] This defines five stages:

* *Discovery* - user requirements gathering
* *Alpha* - prototyping based on user requirements
* *Beta* - refinement of prototype to build a stable project
* *Live* - launch of project, improvements and maintenance
* *Retirement* - either because it will be replaced or discontinued

[3] Please use the information in the *Feedback* section, rather than direct contact.

## TODO

Roughly from highest to lowest priority:

### Alpha

* Add JSON Schema validation to methods to better test payloads

### Beta

* Testing with Bamboo
* ~~Test with Semaphore~~ (done, but not sub-tasks)
    * Test using JUnit output format and if this helps us 
* Format README using normal conventions
* Work on a proper way to store and distribute Ansible vault password files (add to prelude role? Download from team directory? [Private key for authentication] or S3 bucket? [ENV_VAR for authentication])
* If Newman proves useful, convert into a proper BARC role

## Requirements

### Manually through a *local VM*

* Typical project requirements (Vagrant, Ansible, etc.)
* Ansible vault password file [1]

### Automatically through *Semaphore CI*

* Access to the [project on Semaphore](https://semaphoreci.com/felnne/postman-newman-testing/) [2]

[1] This playbook uses an Ansible vault managed variables file to set the API user credentials. The password for this vault is contained in `provisioning/vault_pass.txt` and passed to the `ansible-playbook` at run time.

For obvious reasons this file is **MUST NOT** be checked into source control and instead be manually copied into place. Users can request this file by contacting the BAS Web & Applications Team, see the *Feedback* section of this README for details.

[2] This project is open-sourced, so anyone can access the build results for example, access is therefore only needed to create the project (which is only needed once and isn't documented here), or to modify the project.

If this applies to you, please contact the *Project Maintainer* for collaboration rights.

## Setup

### Manually through a *local VM*

```shell
$ vagrant up
$ ansible-playbook -i provisioning/development provisioning/site-dev.yml --vault-password-file provisioning/.vault_pass.txt
```

### Automatically through *Semaphore CI*

The setup of the Semaphore project will not be discussed at length here. However there are two elements that are worth documenting; configuration files and build settings.

### Configuration files

A single, encrypted, [configuration file](https://semaphoreci.com/docs/adding-custom-configuration-files.html) is used in this project. It contains the password to the Ansible vault passed to the `ansible-playbook` command at run-time. See the *Requirements > Manually through a local VM* section for more details.

### Build settings

As is possible to use Ansible with Semaphore, much of the configuration of the test environment is documented through the tasks in `provisioning/site-ci.yml`. However there are some steps that don't make sense to run through Ansible, or are needed to set Ansible up, that must be performed within Semaphore itself.

These steps are documented here (and should be kept in-sync):

> `sudo pip install ansible`
> Installs Ansible.
> 
> `ansible-playbook -i provisioning/local provisioning/site-ci.yml --vault-password-file provisioning/.vault_pass.txt --syntax-check`
> Ensures the playbook to configure the test environment complies with Ansible's syntax rules.
> 
> `ansible-playbook -i provisioning/local provisioning/site-ci.yml --vault-password-file provisioning/.vault_pass.txt`
> Configures the test environment, installing Newman and its dependencies and generating the environment file with sensitive information such as API credentials.
> 
> `newman -c collection.json -d data.json -e environment.json --noColor --exitCode`
> Runs Newman tests. The `--noColor` option ensures its output can be read within Semaphore, the `--exitCode` option will cause Semaphore to fail this step if any Newman test fails.

## Usage

This project includes a Postman collection, data file [1] and environment [2] suitable for testing the BAS People API. These are feed into Newman and the various tests specified in requests in the collection are executed.

### Manually through a *local VM*

```shell
$ ssh app@postman-newman-dev-node1.v.m
$ newman -c collection.json -d data.json -e environment.json
```

Note: Newman is executed manually, rather than through Ansible, to prevent Ansible swallowing all output to `stdout` (i.e. the test results).

### Automatically through *Semaphore CI*

Semaphore will execute Newman on each commit made to branches it is configured to build from. By default all branches are built from, unless a white list is made. During this early testing phase this default is being used.

[1] This data file is simply a JSON array with each value replacing an environment variable on each iteration [3]. The number of iterations is defined by the number of items in the JSON array.

[2] This environment file **SHOULD** include entries for API user credentials, however the values for these entries are stored in clear-text and therefore cannot be checked into source control.

To overcome this, the environment file is stored as a Ansible template with variable substitutions used for sensitive values, an Ansible vault managed variables file stores these sensitive values. This variables file a template task are included in the `site-dev.yml` playbook to dynamically create a *filled in* environment file. This file **MUST NOT** be checked into source control.

[3] The variables set by this data file do not need to exist within the environment, values for the current iteration and *static* values defined in the environment file will be merged together. Consequently there is no difference in accessing a variable from the environment file or data file, both use the `{{variable}}` syntax.

## Feedback

Please log all feedback to the BAS Web and Applications Team:

* If you are a BAS/NERC staff member please use our [Jira project](https://jira.ceh.ac.uk/browse/BASWEB) with the *Research* and *Projects* components.
* If you are external to BAS/NERC please email [basweb@bas.ac.uk](mailto:basweb@bas.ac.uk) to log feedback directly.

## License

Copyright 2015 NERC BAS.

Unless stated otherwise, all documentation is licensed under the Open Government License version 3 and all code licensed under the MIT License.

Copies of all licenses are included in this project's root directory.
