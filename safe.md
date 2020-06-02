# SAFE (Secure Access For All)


SAFE is a trust platform that enables powerful and flexible policies for access to data in ImPACT, with a language to specify rules and policies for authorization and trust, and a logic engine to automate compliance checks.  More information about SAFE [here](https://github.com/RENCI-NRIG/SAFE).

SAFE policy rules may incorporate standard authorization models: access control lists, tokens or capabilities, roles, groups, and attributes.   But SAFE goes beyond that to integrate other elements important for data access in ImPACT: Data Usage Agreements, administrative approval workflows, infrastructure security posture, and software identity attestation.  In this way ImPACT may incorporate purpose and context into data access decisions.  SAFE policies may also incorporate and unify trust data from more established trust cyberinfrastructure, such as InCommon, Grouper, CoManage, and LDAP.

SAFE uses a declarative logic language to specify policy rules and assertions about principals and objects.  Examples of principals in ImPACT are data producers, data consumers, and various intermediaries including identity servers and other institutional authorities.   Examples of objects include datasets, data collections, research groups and projects, software programs and computers, subnets, and other infrastructure elements.

SAFE's logical model is a logic of belief: every assertion or rule is attributed to a principal---its speaker.  In this way, all reasoning about trust is end-to-end: compliance checks track and validate the sources of information at each step to generate a verifiable proof of compliance.     Various authentication mechanisms can plug into this framework: e.g., certificate signing, secure HTTP, and single sign-on attributes.

SAFE security policies are executable queries over data, based on a standard logical inference procedure over an assembled context of assertions and logical rules.   SAFE employs a domain-specific scripting language to provision logic into linked certificates and rule packages in an underlying storage engine, and to assemble the logic content for each authorization query. 
