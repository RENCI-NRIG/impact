# ImPACT (Infrastructure for Privacy-Assured CompuTations) 

ImPACT will free researchers to focus more fully on science by supporting the analysis of multi-institutional data while satisfying relevant regulations and interests. It is designed to facilitate secure cooperative analysis, meeting a pressing need in the research community. ImPACT seeks to develop an integrative model for management of trust, deploying a collection of supportive mechanisms at scale into a model cyber-infrastructure. The project develops methodologies with best practices in networking, data management, security, and privacy preservation to fit a variety of use cases.

### Architecture Overview
The major architectural elements include:

- **<a href="https://github.com/RENCI-NRIG/impact/blob/master/prdn.md">Protected Data Network</a>** - Virtualizable infrastructure on which the infrastructure provider can instantiate an enclave at the request of the research team. The virtualized servers in the enclave can be remotely accessed by the research team and can also be reconfigured with new analysis software. Creating isolated enclaves in virtualized infrastructure requires setting up appropriate access control rules on the hosts and on the networks to which the hosts belong in order to guarantee that only trusted individuals can access the hosts from a controlled set of IP addresses and that protected data can’t leave the enclave. PRDNs include:

    - **Proconsul** - A browser-based remote desktop solution that provides responsive desktop access to the enclave, running either Windows or Linux. It uses federated identity solutions to authenticate and authorize users to specific hosts. 
    - **A Singularity-based software pipeline** - Allows researchers to customize their enclaves by easily adding new trusted analysis tools. The tools can be built by the researchers themselves or by their organization’s IT personnel. Tools can be built and tested outside of the enclave, digitally signed, and then made available in a repository within the enclave for researchers to use.

- **<a href="https://github.com/RENCI-NRIG/impact/blob/master/dataverse.md">Dataverse</a>** - A web application (http://dataverse.org) that is used by the data providers to register the protected datasets so they can be discovered by researchers. 

- **Notary Service** - Helps the principals negotiate data use agreement (DUA)  policies. Data providers can register their policies with the service, and other principals (researchers, infrastructure providers, representatives of institutional governance) can make digitally-signed statements or attestations confirming compliance with various parts of the policies. Statements recorded by the Notary Service can then be used to automatically grant or deny access to the protected data made by researchers directly or on their behalf from the enclave.

- **Secure Multiparty Computations (SMC)** - A special type of enclave that relies on cryptographic communications to selectively expose aggregate information about the data in data provider stores will be addressed in a separate series of blog posts.
