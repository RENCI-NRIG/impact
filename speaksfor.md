Speaksfor is a useful mechanism for delegation among principals on the
level of speak~\cite{nexus-2011-sosp}: A principal (called **issuer**)
can use permitted **speaksfor** to issue a certificate of statements on
behalf of another principal (called **subject**). **Speaksfor** is used
by ImPACT mainly in two scenarios: 1) Notary service speaks for the
principals it modulates on a variety of security statements, including
a data provider's policies, an infrastructure provider's attestation,
an institutional governance's compliance assertions, and a researcher's
access requests; 2) Virtual instances launched by the infrastructure
providers speak for the researchers who run applications on them.

Statements of a SAFE certificate issued via speaksfor are normal
logical statements, with the only distinction that the speaker of each
statement is interpreted as the subject of the certificate, rather than
the issuer. SAFE does not impose restriction on how speaksfor statements
are issued: Any principal can issue any speaksfor statements at any
time. But a speaksfor statement is not valid, if the issuer does not
get the **speaksfor** permission from the subject it intends to speak
for.

Typical use of **speaksfor** involves three actions that happen in
order but do not need to be back-to-back: 1) subject grants speaksfor
to issuer; 2) the issuer publishes statements on behalf of the subject;
3) an authorizer who wants to use the statements checks statement
validity in accordance with speaksfor. In addition, SAFE represents a
**speaksfor** permission with a logical statement. Thus, validation of
speaksfor can be realized as a simpe invocation of a logic inference
engine. The following explains the built-ins and conventions available
in SAFE that can facilitate the implementation of each action.



