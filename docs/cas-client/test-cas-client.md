# Testing the CAS Client

You're ready to test - just go to the root of your development CAS server (https://your-cas-server.domain.edu).

If it works, when you click "here" on the index.php page you'll see something like:

<figure>
  <img src="/images/basic-cas-login-wildcard.png" alt="Screenshot showing the basic CAS login page - with the service name shown (as defined in the .json file)"/>
</figure>

<hr>

If the login works - you'll see the following (with whatever your test user is):

<figure>
  <img src="/images/basic-cas-response-test.png" alt="Screenshot showing output from our CAS test php page, with the logged in user shown."/>
</figure>

If you get unauthorized messages, set the debug options in your CAS client config in /etc/httpd/conf.d and restart Apache, then look at the Apache logs.  When I was creating these docs with a new CAS server - I had forgotten to add the locally signed cert and was getting an unauthorized message due to that (the Apache CAS client was refusing to communicate with the CAS server since it had a non-commercial cert it had no idea about).