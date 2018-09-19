# ImPACT (Infrastructure for Privacy-Assured CompuTations) 

ImPACT will free researchers to focus more fully on science by supporting the analysis of multi-institutional data while satisfying relevant regulations and interests. It is designed to facilitate secure cooperative analysis, meeting a pressing need in the research community. ImPACT seeks to develop an integrative model for management of trust, deploying a collection of supportive mechanisms at scale into a model cyber-infrastructure. The project develops methodologies with best practices in networking, data management, security, and privacy preservation to fit a variety of use cases.

## Installation: Major Components
1. Proconsul (or the poor man's substitute, described below)
1. Shibboleth with optional Grouper
1. Active Directory Domain Controller
1. Singularity Hub
1. Jenkins

### Proconsul

Prepare a Linux server running [Docker](https://www.docker.com/get-started). Proconsul requires the Docker web service API, so be sure `dockerd-current` launches with the following options: 

```--host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375```

For the impact project we're running a CentOS 7 VM with two cores and four GB of RAM, but the recommended rule is one core and one GB of RAM for the system plus one core and one GB of RAM for every 10 simultaneous users expected.

### Shibboleth, Grouper

Your institution's Shibboleth Identity Provider is a prerequisite for running Proconsul, but depending on the configuration requirements of your institution you may need to install and configure a Shibboleth service provider on the Virtual Machine rather than using the service provided in the Docker container. In UNC's case, a custom MetadataProvider and additions to the attribute-map.xml file necessitated this.

Proconsul checks for the existence of `/etc/shibboleth/sp-key.pem` and will skip Shibboleth configuration during the build process.

### Active Directory Domain Controller

### SingularityHub

### Jenkins