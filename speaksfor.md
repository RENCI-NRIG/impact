A principal (called **issuer**) in SAFE can use permitted **speaksfor**
to issue a certificate of statements on behalf of another principal
(called **subject**). Statements in such a certificate are normal
logical statements, with the only distinction that the speaker of each
statement is interpreted as the subject of the certificate, rather than
the issuer. SAFE does not impose restrition on how speaksfor statements
are issued: Any principal can issue any speakfor statements at any
time. But the speaksfor statements are not valid, if the issuer does
not get the **speaksfor" permission from the subject it intends to
speak for.

A typical use of **speaksfor** involves three actions that happen in
order: 1) subject grants speaksfor to issuer; 2) the issuer publishes
statements on behalf of the subject; 3) an authorizer who want to use
the statements check their validity in accordance with speaksfor. In
addition, a **speakfor** permission granted from a suject is
represented as a logical statement. Thus, validation of speaksfor is 
realized as a simpe invocation of a logic inference engine. The
following explains the built-ins and conventions available in SAFE that
can facilitate the implementation of each action.      

