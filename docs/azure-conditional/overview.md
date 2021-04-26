# Azure Conditional Access

Azure Conditional Access is these best place (that I know of) to enforce MFA on users for services that use Azure AD for authentication.  You can enforce MFA on all users specific user groups, etc.  You can enforce or exclude by IP, and can use Duo, or Microsoft's MFA (and certainly others).  Doing this in Azure requires an Azure Active Directory P1 or P2 subscription (P1 at least is included as of this article being written in A3 and A5 but can be purchased separately for A1 users as well).

For our environment currently - we are using Duo for all users with the exception of a set of admins who are set to use the Microsoft MFA.

!!! caution "Blocking legacy authentication"
    If you're setting up these fancy conditional access policies - you may also want to block older legacy applications that don't support Microsoft's Modern Authentication.  If you don't - you'll be leaving holes (not CAS specific of course) that don't require MFA.

    Microsoft is supposed to block this for all services/protocols later in 2021 anyway so it may be a moot point.

Just be cautious about enabling MFA both via CAS and a conditional access policy as you'll get users having to MFA twice in a single login.

## Enabling Conditional Access

The [Duo documentation](https://duo.com/docs/azure-ca) was all I needed here - so I'm not going to reinvent the wheel by replicating them here.  It was pretty simple to implement.

!!! critical "Don't lock yourself out!"
    When you are enabling conditional access - initially only include test users.  As things are tested - you can eventually move to excluding certain users.

    You may or may not want to have a 'break glass' account that is not ever used in normal circumstances but is used when something breaks (due to bad configuration, problems in Azure, etc.)

## More complicated policies

There are times you may want more complicated policies.  One example of this would be our ticketing application.  I don't need MFA on regular users there (since doing so would prevent them from opening a ticket if the issue they were having was *related* to MFA), but I do need MFA on any IT user in the ticketing system.  So for this example, I would have two conditional access policies:

* The main one which includes all users *except* IT staff.  This policy includes all applications but excludes this ticketing app.
* A second policy which *only* includes IT staff.  This policy includes all applications without an exclusion for the ticketing app.

This is only one example - but you may think of others.  You may not want to get too fancy though as it may leave gaps in your policies.



## References

* [Duo: Azure Conditional Access](https://duo.com/docs/azure-ca)
* [How to: Block legacy authentication to Azure AD with Conditional Access](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/block-legacy-authentication)