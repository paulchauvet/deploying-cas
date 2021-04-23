# Azure Conditional Access

Azure Conditional Access is these best place (that I know of) to enforce MFA on users.  You can enforce MFA on all users specific user groups, etc.  You can enforce or exclude by IP, and can use Duo, or Microsoft's MFA (and certainly others).  Doing this in Azure requires an Azure Active Directory P1 or P2 subscription (P1 at least is included as of this article being written in A3 and A5 but can be purchased separately for A1 users as well).

For our environment currently - we are using Duo for all users with the exception of a set of admins who are set to use the Microsoft MFA.

!!! caution "Blocking legacy authentication"
    If you're setting up these fancy conditional access policies - you may also want to block older legacy applications that don't support Microsoft's Modern Authentication.  If you don't - you'll be leaving holes (not CAS specific of course) that don't require MFA.

    Microsoft is supposed to block this for all services/protocols later in 2021 anyway so it may be a moot point.

## Need to do more here and decide if I just defer to the Duo docs here.

## Enabling Conditional Access

1. Go to the [Azure Portal](https://portal.azure.com)
2. In the search box type "Conditional Access" to find *Azure AD Conditional Access*
3. Create a new policy as follows:
    * Name: Require Duo MFA
    * Users and groups: at least initially - only include some test users or test groups.  Eventually you will want to have all groups that you want MFA on (i.e. faculty/staff/student groups as examples)
    * Cloud apps or actions: You probably want to include all cloud apps.  You may eventually want to *exclude* the CAS app you created for delegated authentication if you are forcing Duo at the CAS level.
    * Conditions: You may want to exclude some locations - especially those which you are doing PowerShell or other scripting against Azure/Office 365 on.  Don't set your exclusions too large either.
    * Grant: Set "Grant access" to re

!!! critical "Don't lock yourself out!"
    When you are enabling conditional access - initially only include test users.  As things are tested - you can eventually move to excluding certain users.

    You may or may not want to have a 'break glass' account that is not ever used in normal circumstances but is used when something breaks (due to bad configuration, problems in Azure, etc.)




## References

* [Duo: Azure Conditional Access](https://duo.com/docs/azure-ca)
* [How to: Block legacy authentication to Azure AD with Conditional Access](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/block-legacy-authentication)