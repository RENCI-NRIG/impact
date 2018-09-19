# ImPACT (Infrastructure for Privacy-Assured CompuTations) 

ImPACT will free researchers to focus more fully on science by supporting the analysis of multi-institutional data while satisfying relevant regulations and interests. It is designed to facilitate secure cooperative analysis, meeting a pressing need in the research community. ImPACT seeks to develop an integrative model for management of trust, deploying a collection of supportive mechanisms at scale into a model cyber-infrastructure. The project develops methodologies with best practices in networking, data management, security, and privacy preservation to fit a variety of use cases.

## Installation: Major Components
1. Proconsul (or the poor man's substitute, described below)
1. Shibboleth with optional Grouper
1. Active Directory Domain Controller
1. Singularity Hub
1. Jenkins

Proconsul's author, [Rob Carter](https://github.com/carte018), has excellent instructions [here](https://github.com/carte018/Proconsul/tree/master/Dockerized). Condensed installation instructions documenting our experience installing and configuring the software are below.

### Shibboleth, Grouper

Your institution's Shibboleth Identity Provider is a prerequisite for running Proconsul, but depending on the configuration requirements of your institution you may need to install and configure a Shibboleth service provider on the Virtual Machine rather than using the service provided in the Docker container. In UNC's case, a custom MetadataProvider and additions to the attribute-map.xml file necessitated this.

Proconsul checks for the existence of `/etc/shibboleth/sp-key.pem` and will skip Shibboleth configuration during the build process.

### Active Directory Domain Controller

_(this section shamelessly stolen from [Rob Carter](https://github.com/carte018))_

Proconsul can be used for a number of purposes, from managing privileged access within
an AD and reducing the attack surface for domain admin accounts within the AD to 
providing federated, zero-client-install, access-controlled remote login access to 
RDP- and VNC-capable hosts. In each case, Proconsul does much of 
what it does by manipulating objects in Active Directory.  As such, there is some 
preparation required in your AD in order to deploy and use Proconsul.

You will need to identify or prepare a few objects in your AD for Proconsul's use. As 
you identify or prepare them, you'll want to note some information about them in order
to complete the configuration step below. Specific items you'll need to prepare are:

1. A UPN and password for Proconsul to use to bind to the AD.  This need not be a 
dedicated user for Proconsul's use, but it's strongly recommended -- using a bespoke
user for Proconsul will help avoid conflicting requirements between applications 
sharing the user account, and will allow you better audit options for actions Proconsul
performs in your AD.  **If you plan to use Proconsul to delegate domain admin 
access** the AD user that Proconsul binds with **must** have domain admin rights.  In 
that scenario, you may choose to make it the **only** persistent AD account with 
domain admin rights, but it needs that level of access in order to create and assign
domain admin rights to its dynamically-provisioned AD users.  Otherwise, this may be 
a "normal" AD user (to which specific rights will be assigned below).

2. An OU in the AD where Proconsul will provision and deprovision its dynamic users. This need not be a dedicated OU, but it is strongly recommended -- using a bespoke OU 
for Proconsul dynamic users will allow you to more easily track Proconsul activity,
and may allow you to limit the scope of rights the Proconsul server can exericise.
**If the bind user identified above is NOT a domain admin user** you will need to 
grant user management rights (at a minimum, create, delete, modify group membership, 
and reset password rights for user class objects) to that user in this OU.  
Proconsul will bind to the AD using the above user account and manipulate its dynamic 
users in this OU.

3. A group in the AD to which Proconsul will add its dynamic users when they are 
created.  This need not be a dedicated group, but it's strongly recommended -- using 
a bespoke group for this purpose will give you a simple and direct way to track 
Proconsul users from within the AD.  Depending on your use cases, you may find other 
uses for this group (eg., a common use case would involve adding this group to the 
RDP Users group of target Windows systems to allow all dynamically-provisioned 
Proconsul users RDP access to those hosts).  Regardless of whether this is a bespoke
or shared group, it **should not** be a highly-privileged group.  Proconsul can assign
arbitrary group memberships to its dynamic users based on its configuration and the 
properties of the "real" users it interacts with.  This group should be used only for 
general access and tracking purposes.

4. **Optionally** select an AD site within which to limit Proconsul's use.  If your 
AD environment supports multiple sites, you may find that limiting your Proconsul 
instance to accessing hosts in a single site provides better performance (at the 
expense of needing to operate a separate Proconsul instance for each site).  If your 
environment doesn't support multiple sites or if you want your Proconsul instance to 
span sites, you need not pick a site to use -- you may simply leave the associated 
configuration parameter empty in that case.

5. **Optionally** identify a non-standard group to use for domain admin privileging.  
In most cases, you will want to use the "standard" group `CN=Domain Admins,OU=Users,DC=your,DC=dom,DC=ain` to delegate domain admin rights to dynamically provisioned users.
In some circumstances, it may be desirable to define a secondary group for Proconsul
to use as its "domain admins" group, and add it to the "real" domain admins group. 
In either case, you will need to select a domain admin group for Proconsul to use.  
It will only actually be used in the event that you configure Proconsul to
manage delegated domain admin rights, but the tool will require a value for the 
configuration option, regardless.

6. **Optionally** create or identify a second user account for Proconsul's docker-gen
instance to use.  If you are not using a domain admin account as the bind user for 
Proconsul, you may simply re-use the bind user for this purpose.  If you are using 
a domain admin account for the former purpose, you may want to create a separate 
account with user management rights in the OU you created above for use by the 
docker-gen process.  Docker-gen is a separate process (courtesy of Jason Wilder) used
by the Proconsul distribution to handle clean-up of AD and MySQL artifacts when 
Proconsul's spawned Docker containers terminate.  A domain admin account will 
work just fine, but you may prefer to use a less privileged account with docker-gen
for purposes of compartmentalization.

### Proconsul

Prepare a Linux server running [Docker](https://www.docker.com/get-started). Proconsul requires the Docker web service API, so be sure `dockerd-current` launches with the following options: 

```--host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375```

For the impact project we're running a CentOS 7 VM with two cores and four GB of RAM, but the recommended rule is one core and one GB of RAM for the system plus one core and one GB of RAM for every 10 simultaneous users expected.

To build Proconsul you will need a JDK and maven, for instance:

```yum install java-1.8.0-openjdk-devel maven```

Retrieve the current source from GitHub:

```git clone https://github.com/carte018/proconsul```

### Shibboleth, Grouper

Your institution's Shibboleth Identity Provider is a prerequisite for running Proconsul, but depending on the configuration requirements of your institution you may need to install and configure a Shibboleth service provider on the Virtual Machine rather than using the service provided in the Docker container. In UNC's case, a custom MetadataProvider and additions to the attribute-map.xml file necessitated this.

Proconsul checks for the existence of `/etc/shibboleth/sp-key.pem` and will skip Shibboleth configuration during the build process.

### Active Directory Domain Controller

### SingularityHub

### Jenkins