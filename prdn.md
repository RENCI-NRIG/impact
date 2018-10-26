# Protected Research Data Network (PRDN)

## Installation: Major Components 

ImPACT has configured two types of protected research data networks (PRDN) to support researchers in sharing data and collaborating on its analysis. Both PRDNs attempt to accomplish the same functions.

- Both provide secure authentication and authorization of users. 
- Both provide a remote desktop view into a private network for sharing data and accessing services and software. 
- Both are firewalled against unwanted inflitration and exfiltration of sensitve data. 

The first one discussed on this page is the _Abridged PRDN_. It requires fewer resources to implement and was ImPACT's first step toward the full version, the _PRDN with Proconsul_. Choose the environment that works best given your resource situation. 

**Abridged PRDN**

- Linux VMs - _Runs all the software (e.g., TurboVNC) necessary for the private network and hosts a secure data staging server._
- TurboVNC - _Provides secure authentication and authorization of users, and a remote desktop view (with the Linux operating system) into the private network._
- Singularity Client - _Specially configured software that allows access to download data analysis applications stored in Odum's Singularity Hub._
- Odum's Singularity Hub - _Stores Singularity images (see https://singularity.lbl.gov/) of end-user data analysis applications._

**PRDN with Proconsul**

- Prerequisites for Identity Management
  - Shibboleth with optional Grouper — _Provides secure authentication and authorization of users._
  - Active Directory Domain Controller — _Holds temporary accounts used for temporal virtual machine login._
- Odum's Singularity Hub — _Stores Singularity images of end-user data analysis applications. See [https://singularity.lbl.gov/](https://singularity.lbl.gov/) for more information._
- ImPACT's Instantiation of Proconsul - _A secured interface (i.e., remote desktop view, either with Linux or Windows operating systems) into the private network, firewalled against unwanted data infiltration/exfiltration._
  
  Proconsul's author, [Rob Carter](https://github.com/carte018), has excellent, detailed instructions [here](https://github.com/carte018/Proconsul/tree/master/Dockerized). 

**Testing and Maintenance**
- Jenkins — _for continuous integration testing of Singularity images_

Condensed installation instructions documenting our experiences installing and configuring the software are below.


## Abridged PRDN

### Linux VMs

Test VMs are installed with CentOS 7, and are accessed through a bastion host. At present users connect via ssh using X11 forwarding, then launch a TigerVNC client to access the target VM graphically:

	$ /opt/TurboVNC/bin/vncviewer target-hostname:display

### TurboVNC

In order to improve performance and minimize the incidence of software bugs, we compiled more current TurboVNC RPMs from the [TurboVNC GitHub repo](https://github.com/TurboVNC/turbovnc/blob/master/BUILDING.md) and launched three TurboVNC listeners per VM with systemd unit files such as [this sample file](turbovnc.service.sample). Note that VMware VMs in particular will achieve the best performance by enabling 3D support in the VM's configuration.

### Singularity Client

From the EPEL repository, provided by the **epel-release** package:

	$ yum install epel-release
	$ yum install singularity

### Odum's Singularity Hub

Installed per [https://github.com/singularityhub/sregistry](https://github.com/singularityhub/sregistry)

## PRDN with Proconsul

### _Prerequisite:_ Shibboleth, optional Grouper

Your institution's Shibboleth Identity Provider is a prerequisite for running Proconsul, but depending on the configuration requirements of your institution you may need to install and configure a Shibboleth service provider on the Virtual Machine rather than using the service provided in the Docker container. In UNC's case, a custom MetadataProvider and additions to the attribute-map.xml file necessitated this.

Proconsul checks for the existence of `/etc/shibboleth/sp-key.pem` and will skip Shibboleth configuration during the build process.

If you are able to use the included Shibboleth service provider, you may configure it by setting appropriate values in [master.config](master.config.sample). Proconsul supports bilateral federation (with a single IDP) or multilateral.

### _Prerequisite:_ Active Directory Domain Controller

This section shamelessly stolen from [Rob Carter](https://github.com/carte018)).

Proconsul can be used for a number of purposes, from managing privileged access within an AD and reducing the attack surface for domain admin accounts within the AD to providing federated, zero-client-install, access-controlled remote login access to RDP- and VNC-capable hosts. In each case, Proconsul does much of 
what it does by manipulating objects in Active Directory.  As such, there is some  preparation required in your AD in order to deploy and use Proconsul.

You will need to identify or prepare a few objects in your AD for Proconsul's use. As  you identify or prepare them, you'll want to note some information about them in order to complete the configuration step below. Specific items you'll need to prepare are:

1. A UPN and password for Proconsul to use to bind to the AD.  This need not be a dedicated user for Proconsul's use, but it's strongly recommended -- using a bespoke user for Proconsul will help avoid conflicting requirements between applications sharing the user account, and will allow you better audit options for actions Proconsul performs in your AD.  **If you plan to use Proconsul to delegate domain admin access** the AD user that Proconsul binds with **must** have domain admin rights. In that scenario, you may choose to make it the **only** persistent AD account with domain admin rights, but it needs that level of access in order to create and assign domain admin rights to its dynamically-provisioned AD users. Otherwise, this may be a "normal" AD user (to which specific rights will be assigned below).

2. An OU in the AD where Proconsul will provision and deprovision its dynamic users. This need not be a dedicated OU, but it is strongly recommended -- using a bespoke OU for Proconsul dynamic users will allow you to more easily track Proconsul activity, and may allow you to limit the scope of rights the Proconsul server can exericise.
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

6. **Optionally** create or identify a second user account for Proconsul's docker-gen instance to use.  If you are not using a domain admin account as the bind user for Proconsul, you may simply re-use the bind user for this purpose.  If you are using a domain admin account for the former purpose, you may want to create a separate account with user management rights in the OU you created above for use by the docker-gen process. Docker-gen is a separate process (courtesy of Jason Wilder) used by the Proconsul distribution to handle clean-up of AD and MySQL artifacts when Proconsul's spawned Docker containers terminate.  A domain admin account will work just fine, but you may prefer to use a less privileged account with docker-gen for purposes of compartmentalization.

It is necessary to extend the Active Directory schema to include POSIX mappings for users who will use Proconsul to log in to Linux machines, configured with [SSSd](sssd.conf). The minimum attribute set required includes uidNumber, gidNumber, homeDirectory, and loginShell. homeDirectory may be left blank but would relegate users to a default home directory at the root of the filesystem.

### Proconsul

Prepare a Linux server running [Docker](https://www.docker.com/get-started). Proconsul requires the Docker web service API, so be sure `dockerd-current` launches with the following options: 

```--host=unix:///var/run/docker.sock --host=tcp://127.0.0.1:2375```

For the impact project we're running a CentOS 7 VM with two cores and four GB of RAM, but the recommended rule is one core and one GB of RAM for the system plus one core and one GB of RAM for every 10 simultaneous users expected.

To build Proconsul you will need a JDK and maven, for instance:

`$ yum install java-1.8.0-openjdk-devel maven`

Retrieve the current source from GitHub:

`$ git clone https://github.com/carte018/proconsul`

Proconsul requires a public/private key pair and a signed SSL certificate in order to operate, and the subject of the certificate must match the name users will use when connecting to the service. You will need to provide the names of two files -- one containing a PEM- formatted version of the server's private key and one containing a PEM-formatted version of the SSL certificate you'll be using for your Proconsul server in order to build the Proconsul installation.

##### Populate [master.config](master.config.sample)

Please find Rob Carter's expansion of each setting below:

**fqdn** - the hostname users will use in URLs to contact your Proconsul instance  
**certfile** - full pathname to a file containing your SSL certificate  
**keyfile** -- full pathname to a file containing your SSL private key  
**adldapurl** -- LDAP URL for connecting to your AD (usually "ldaps://domain.name:636")  
**adbinddn** -- UPN of the bind user Proconsul should use in the AD  
**adbindcred** -- password for the user in adbinddn  
**addomain** -- DNS name of your AD domain (eg. "my.dom.ain")  
**adsearchbase** -- DN of the root of you AD domain (eg., "DC=my,DC=dom,DC=ain")  
**usedelegatedadmin** -- Unless you're at Duke, enter "n" here.  
**addeptbase** -- Unless you're at Duke, leave this empty  
**adorgbase** -- Unless you're at Duke, leave this empty  
**adtargetbase** -- OU where Proconsul will create its dynamic users (full DN)  
**adldapdcs** -- comma-separated list of LDAP URLs for individual DCs in your domain  
**addagroupdn** -- DN of the AD group to use as the domain admins group   
**adproconsuldefgrp** -- DN of the AD group Proconsul should add its dynamic users to   
**adsiteprefix** -- AD site (or unique prefix thereof) to restrict Proconsul to (if any)  
**rdpdockerimage** -- name/tag to use for the normal-sized RDP client container  
**rdplargedockerimage** -- name/tag to use for the large-sized RDP client container  
**vncdockerimage**  -- name/tag to use for the normal-sized VNC client container  
**vnclargedockerimage** -- name/tag to use for the large-sized VNC client container  
**dockerhost** -- IP address of the host where Docker containers should be run (almost always 127.0.0.1)  
**dockercpuset** -- cpuset value for your dockerhost (eg., "0-1" for a 2-core server)  
**novnchostname** -- host where novnc connections should be sent (usually == $fqdn)  
**mysqlhost** -- IP of host where Proconsul MySQL server runs (usually 127.0.0.1)  
**proconsuluser** -- MySQL userid to create for use with Proconsul schema  
**proconsuldbpw** -- password to use for proconsuluser in the MySQL DB  
**logouturl** -- URL to anchor behind the "sign out" link on Procosnul pages  
**pcadminlist** -- list of REMOTE_USER values for users who should have rights to the Proconsul administrative UI -- PROBABLY NEEDS TO INCLUDE YOURS  
**authnmode** -- authentication mode (for now, this **must** be "saml")  
**federationmode** -- "bilateral" to federate with one IDP, "multilateral" for a full SAML federation  
**spentityid** -- unique entityID to use for the Proconsul SAML relying party  
**idpentityid** -- entityID of your IDP (if bilateral above); empty otherwise  
**discoveryurl** -- URL of SAML federation discovery service (if multilateral federation -- empty otherwise)  
**federationmdurl** -- URL to retrieve IDP or federation metadata  
**mdsigningkey** -- file containing certificate to validate metadata signature  
**dockergenuser** -- UPN of AD user for docker-gen to use for cleanup  
**dockergenpw** -- password for dockergenuser  

##### Build Docker Containers

To build a fresh install:  `./build -c master.config`

To rebuild for reconfiguration without wiping out the MySQL database:  `./build -c master.config -r`

Fun with semantics: in this case, _rebuild_ means _rebuild everything except the database_ and is probably what you'll run most often.

##### Launch Proconsul

`./run3.sh`

### SingularityHub

Installed per [https://github.com/singularityhub/sregistry](https://github.com/singularityhub/sregistry)

### Jenkins

A standard (CentOS RPM-managed) Jenkins installation receives webhooks from each push to a GitHub repo containing Singularity source files. The Jenkins job pulls the source for the image, compiles the image, and on successful compilation uploads the image to the SingularityHub for end-user consumption.

### Linux VMs

The full PRDN uses Linux target VMs as well, but they require further configuration. They'll need the following additional packages:

`$ yum install xrdp xorgrdp samba-libs samba-common-tools sssd-ad sssd-ldap sssd-krb5 oddjob-mkhomedir`

You may use this [sssd.conf sample](sssd.conf.sample) as a sign-post toward a working sssd configuration.

You'll need to bind the Linux VM to active directory, specifying the FQDN of the controller if your DNS, like ours, isn't 100% controlled by Active Directory:

`$ sudo net ads join -s activedirectory.fqdn -k`

- Note that the machine's "short" hostname must not exceed the Microsoft maximum of 15 characters, or the machine's entry in Active Directory will be truncated.

## Firewall Rules

For the Abridged PRDN the firewall requirements are minimal (SSH at the perimeter, a VNC port range large enough to accomodate the anticipated maximum number of simultaneous users).

For the full PRDN the incoming requirements are both https (443/tcp) AND a VNC port range twice as large as the anticipated maximum number of simultaneous users (in our case, 5900-6000/tcp).

Our PRDN is behind a Juniper firewall so we've purposefully set the default outgoing rule to DENY. We then had to open ports for the following services:

 - campus DNS (53 tcp/udp)
 - campus NTP (123 tcp/udp)
 - HTTPS (443/tcp) to our SingularityHub, campus CentOS mirror, Shibboleth, etc.
 