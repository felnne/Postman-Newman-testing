
# Postman Newman testing

Simple project to use Postman's [Newman](https://github.com/postmanlabs/newman/) tool to test the BAS People API.

## Background

This project will be developed using the Government Digital Service (GDS) design phases methodology. This defines five stages:

* *Discovery* - user requirements gathering
* *Alpha* - prototyping based on user requirements
* *Beta* - refinement of prototype to build a stable project
* *Live* - launch of project, improvements and maintenance
* *Retirement* - either because it will be replaced or discontinued

This project is part of a project to add a testing and deployment workflow to BAS APIs in order to reduce manual testing/verification of changes, improve the speed changes are delivered and improve the robustness of our services.

This project is in *Alpha*, when it reaches *Beta* it most likely be merged into this larger project. It is therefore not expected this project will move beyond *Beta* and move straight to *Retirement*.

The tests performed by Newman will need to executed by *something*, ideally this would be some sort of Continuous Integration (CI) service, such as [Semaphore](semaphoreci.com) or [Bamboo](https://www.atlassian.com/software/bamboo), though manual testing will still be needed.

This API testing will form part of a Continuous Deployment (CD) process (i.e. only deploy if tests are green). Ideally the CI and CD process would both be performed by the same service (in additional to manually) to reduce costs and configuration.

## TODO

Roughly from highest to lowest priority:

* Automate installation of `g++`, using `build-essentials`?, and `newman`
* Use Postman's *data* file format for testing the getByID ad getByAlias methods of the People resource (i.e. don't hard-code in environment)
* Test with Semaphore
* Add JSON Schema validation to methods to better test payloads
* Testing with Bamboo
* Format README using normal conventions

## Requirements

* Typical project requirements (Vagrant, Ansible, etc.)
* A Postman collection and environment suitable for testing
* Ansible vault password file [1]

## Setup
[1] This playbook uses an Ansible vault managed variables file to set the API user credentials. The password for this vault is contained in `provisioning/vault_pass.txt` and passed to the `ansible-playbook` at run time.

For obvious reasons this file is **MUST NOT** be checked into source control and instead be manually copied into place. Users can request this file by contacting the BAS Web & Applications Team, see the *Feedback* section of this README for details.

As *Controller*:

```shell
$ ansible-playbook -i provisioning/development provisioning/site-dev.yml --vault-password-file provisioning/.vault_pass.txt
$ sudo apt-get install g++
$ sudo npm install -g newman --unsafe-perm
```

## Usage

As *App*:

```shell
$ newman -c collection.json -e environment.json
```

## Feedback

Please log all feedback to the BAS Web and Applications Team:

* If you are a BAS/NERC staff member please use our [Jira project](https://jira.ceh.ac.uk/browse/BASWEB) with the *Research* and *Projects* components.
* If you are external to BAS/NERC please email [basweb@bas.ac.uk](mailto:basweb@bas.ac.uk) to log feedback directly.

## License

Copyright 2015 NERC BAS.

Unless stated otherwise, all documentation is licensed under the Open Government License version 3 and all code licensed under the MIT License.

Copies of all licenses are included in this project's root directory.
