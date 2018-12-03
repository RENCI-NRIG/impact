An examplary slang script that uses ldap for authorization based on group membership

[listing]
----

import("../strong/strong.slang").

//defenv Selfie() :-
//  spec('Loading principal keypair'),
//  principal('path/to/pem').

defcon addTokenToSubjectSet(?Token) :-
  spec("Add a token to a subject set. Invoke after the user gets a delegation"),
    {
        link($Token).
	    label("subject($Self)").
	      }.

defpost updateSubjectSet(?Token) :- [addTokenToSubjectSet(?Token)].

//
// COManage Group Policy
//

defcon coManageGroupPolicySet() :-
  spec('Policy set for membership on COManage'),
    {
        membership(?Group, ?User) :-
	      membership(?Group, ?User, _).

    membership(?Group, ?User, ?Delegatable) :-
          ?COManage: groupMember(?Group, ?User, ?Delegatable),
	        trustedCOManage(?COManage).

    label("comanage-group-policy").
      }.

definit coManageGroupPolicySet().

//
// Local policy
//

defcon localPolicySet() :-
  spec('Local Policy set for membership on COManage'),
    {
        trustedCOManage("registry-test.cilogon.org").
	label("local-policy").
    }.

definit localPolicySet().


defguard authorizeByMembership(?Group, ?PID) :-
  ?MembershipPolicy := label("comanage-group-policy"),
  ?LocalPolicy := label("local-policy"),
  {
    link($MembershipPolicy).
    link($LocalPolicy).
    link("ldap://registry-test.cilogon.org:389/employeeNumber=ImPACT1000001,ou=people,o=ImPACT,dc=cilogon,dc=org").
    membership($Group, $PID)?
    //link("http://152.3.145.253:8098/types/safesets/buckets/safe/keys/7O1XHsTVBokBtFugmufandrvfPVRHwUaBc1pr_rn9YU").
    //endorseAttesterImage($Group, $PID)?
   }.