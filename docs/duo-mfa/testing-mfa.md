# Testing Duo MFA

First - go to your CAS client index page and select the *Duo MFA* test link.


<figure>
  <img src="/images/duo-mfa-0.png" alt="Screenshot showing CAS test client index page"/>
</figure>

Then, login with a user that exists in your directory and exists in Duo.  You should note the specific Duo test service name (*Apache Test - Duo MFA* in my example).  If you don't see that - you may have an error in your service or in your Apache config, causing another service to pick up instead.

<figure>
  <img src="/images/duo-mfa-1.png" alt="Screenshot showing CAS login page"/>
</figure>

You should, if all is working correctly, see the Duo prompt framed within the CAS window.  In the screenshot below, I'm prompted for three options of second factor.
* Send me a push - this will send a push notice to my phone with the Duo app installed.  I can click "Approve" or "Deny" there.  Note: If you use this option, the Duo app will show the client name that we called this within the Duo admin panel.
* Enter a passcode - this is a passcode either from the Duo app or a one-time-passcode (OTP) token
* The blue "Use your Security Key to login" message is because I also have a Yubikey Security Key, acting as a Fido U2F (Universal Two Factor).  

I can use any of these to login - but what options you or your users have depends on how you are using Duo.

<figure>
  <img src="/images/duo-mfa-2.png" alt="Screenshot of Duo prompt with push, passcode, and security key options"/>
</figure>

After logging in - you should see your php test page in the Duo client which will show the attributes that are resolved.