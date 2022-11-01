---
layout: default
title: Request Tracker SSO configuration
By: Karl
---

This is a pretty old post but some might find it useful!

I really like Request Tracker from bestpractical, it's the only request/ticketing system that fits the way I work (everything is in email!). It's also super lightweight and very extensible (provided you can climb the perl learning curve). Despite nearly everything being done in email, there is usually some reason you'll need to login to the interface. I hate forcing users to have separate passwords, and since I've got ADFS and active directory, there's no reason they shouldn't use their AD login to access it.

This is a little write-up is my experience getting SSO (microsoft ADFS, to be specific) to work to login users 'automatically' as well as provide a backup username/password option (since you'll want to get to the 'root' account from time to time).

# Environment

I'm using RT 4.4, CentOS 7, and Apache httpd 2.4 for the server side, and ADFS 3.0 (windows 2012 R2), so your mileage may vary using different distributions or versions of ADFS. I'd initially tried using nginx because I prefer its speed and simplicity, but there are no good supported SAML providers for it. Also most RT documentation is written as if Apache is used.
As anything else in technology, there are dozens of ways to skin this cat depending on your situation. RT has a couple of different providers on their github that might suit  you better:

* [Auth via OpenID](https://github.com/bestpractical/rt-authen-openid)
* [OAuth2](https://github.com/bestpractical/rt-authen-oauth2)
* [ExternalAuth with Cookie](https://github.com/bestpractical/rt-authen-externalauth)

I used SAML/ADFS because that's what I use to federate my other applications. I'm quickly beginning to see the need for a general-purpose Windows-based OpenID provider, but it looks like Azure Active Directory may have this feature before ADFS does (if it ever gets it). I may eventually revisit this and use Azure AD as the login (since it supposedly supports OAuth2

The downside to my approach is its reliance on the webserver config vs. native RT configuration for everything. Troubleshooting becomes an exercise in server administration as well as application administration. However, I think the developers behind Mellon probably have a larger installbase than an RT plugin used by a small handful of folks, so I'm more likely to encounter bugfixes and updates (plus in CentOS, Mellon is installed using a package manager). Using a well-established Apache plugin seems reasonably secure.

# Architecture
At a very high level this works by extending RT's login page to include a redirect to our SAML endpoint (a virtual directory that activates the Mellon SAML plugin). The SAML endpoint then hands off the browser to the user's ADFS to perform authentication and create an 'assertion' or message back to the Mellon endpoint that the user is authenticated and has attributes. The assertion is sent in the SAML response via redirect back to Mellon. Finally Mellon sets the REMOTE_USER variable in the web session based on the identity returned by ADFS. RT looks for this variable and uses it to identify the authorized user.

# Configuration

## Apache Configuration
Apache needs a number of extensions to work correctly, and I'm assuming you've already got RT up and running with standard user/password login.

* mod_auth_mellon - SAML SP provider for apache that supports single IdP configurations.
* mod_fcgid - fast CGI manager for RT's fcgi server. For a standalone one-server RT instance, this is fine.
* mod_ssl - always try to run services over TLS

Mellon and Apache are where the action happens in this configuration. Making them play together actually turned out a  little easi er than I thought. Mellon's configuration is about as complicated as most other SAML providers and requires a bit of background in SAML to use.

Essentially SAML is a protocol for having a third-party handle authentication and send you a cryptographically-signed assertion that the user authenticated successfully and has certain attributes (such as email address, group memberships, fullnames, etc). The federation is built by exchanging metadata from the SP and IdP so they know what to expect and how to authenticate each other.  Mellon acts as a service provider, con suming an IdP assertion to authorize a user.

If you installed mellon using a package (and please do), there will be a script supplied in /usr/libexec/mod_auth_mellon to generate your metadata, certificate, and private key files. The script also outputs the 'endpoints' or URL's you'll need to actually use the authentication service. Keep this output since it'll be useful when configuring ADFS.

`./mellon_create_metadata.sh "myRTinstance" "https://rt.example.com/auth"`

Your entity ID is simply an arbitrary unique name that won't conflict with any other names in ADFS. It can be pretty much any string of characters. The second parameter is the URL of the Mellon trusted endpoint. Make sure this doesn't conflict with any directories or virtual locations in RT or Apache.

## RT Authentication virtual directory

The core virtualserver configuration for RT on Apache (at least in my setup) is under /etc/httpd/conf.d/ssl.conf. The location is set to allow all users without requiring webserver-based authentication. I'm assuming this is where you've got your scriptroot for FCGI as well.

The [configuration parameters](https://github.com/Uninett/mod_auth_mellon) you'll need to look at are:
```
MellonEnable "info"
```
This sets the Mellon authentication to 'info' mode, which passes through the SAML-provided values if SAML authentication has occurred, but otherwise is inactive if no SAML authentication has occurred. This way RT can process the authentication if needed without requiring SSO.

```
MellonSPMetad "/etc/httpd/mellon/myRTinstance_metadata.xml"
MellonSPPrivate "/etc/httpd/mellon/myRTinstance.key"
MellonSPC "/etc/httpd/mellon/myRTinstance_metadata.cert"
```

These correspond to the metadata, certificate, and private key created by the mellon_create_metadata script and moved by you into a corresponding directory. Important to make sure the .key file can only be read by root.
```
MellonEndpointPath "path/auth"
```
This is the relative path of the Mellon endpoint you configured in the metadata script - NOTE: we may need to whitelist this in the RT configuration when converting to a plugin.

```
MellonIdPMetad "/etc/httpd/mellon/adfs.xml"
MellonIdPPublic "/etc/httpd/mellon/adfs.pem"
```
These you'll need to obtain your metadata XML and public signing cert from your ADFS server and place on the filesystem of the RT server (see ADFS configuration below)
```
MellonSignatureMethod sha256
```
This sets the signature to SHA256, which is the default for ADFS

Finally we have the 'auth' endpoint we're using for Mellon authentication. Set the directory security to ensure 'satisfy any' allows access from any client and disable Mellon authentication.

```
<Location path/auth/>
    AuthType "Mellon"
    MellonEnable "off"
    Order Deny,Allow
    Allow from all
    Satisfy any
</location>
```

## ADFS Configuration

### Mellon Config with ADFS Metadata

To configure Mellon, you need your XML from the IdP you're using. For ADFS, the default URL is below, just browse and save the XML file
```
https://<your ADFS Server>/FederationMetadata/2007-06/FederationMetadata.xml
```

You'll also need to copy your ADFS server's public signing certificate as a PEM (base64-encoded) file to import into Mellon. The best place to get this is directly from the server (ADFS > Service  > Certificates) open the current Token-signing certificate, select Details then Copy to file. Select base64-encoded and save to your filesystem.

![adfs certificate export](/images/adfs1.png)

Copy these two files to somewhere the webserver can read from them (I put them in /etc/httpd/mellon)

### Configure ADFS

In most cases ADFS can import the metadata XML and pre-populate the SP (or Relying Party, in ADFS terms) relationship. This doesn't work with Mellon's XML, but fortunately the configuration's relatively easy. I'm assuming your endpoint will be called 'myRTinstance' and the endpoint URL will be yourserver/auth. After running the mellon_create_metadata.sh, the endpoints are shown, which you'll need to setup the trust in ADFS:

```
$ ./mellon_create_metadata.sh "myRTinstance" "https://rt.example.com/auth"
Output files:
Private key:                            myRTinstance.key
Certificate:                            myRTinstance.cert
Metadata:                               myRTinstance.xml
Host:                                   rt.example.com
Endpoints:
SingleLogoutService (SOAP):             https://rt.example.com/auth/logout
SingleLogoutService (HTTP-Redirect):    https://rt.example.com/auth/logout
AssertionConsumerService (HTTP-POST):   https://rt.example.com/auth/postResponse
AssertionConsumerService (HTTP-Artifact):   https://rt.example.com/auth/artifactResponse
AssertionConsumerService (PAOS):        https://rt.example.com/auth/paosResponse
```
Copy the Certificate generated to your ADFS server (it's just plan text, so you can copy and paste into notepad if needed).

Create a new Relying Party Trust and choose to enter the data manually. Create the trust as an 'AD FS profile', do not check WS Federation or SAML WebSSO URLs,  use the identifier above (myRTinstance , in this case), do not configure multi-factor, permit all users to access the relying party. Create the RP and view the properties to verify it below:

Select the Identifiers tab. Enter an appropriate Display name (this is only local to ADFS). Enter the first parameter you used for the create metatdata script (in this case 'myRTinstance') as the Relying party identifier and click Add:

![adfs RP identifier](/images/adfs2.png)

Select the Endpoints tab and click Add SAML… select Endpoint Type of 'SAML Assertion Consumer then enter the AssertionConsumerService (HTTP-POST) URL you got from the metadata creation script

![adfs URL](/images/adfs3.png)

Add another endpoint for logout (see Logout below). Note the difference between the trusted URL (which is the logout endpoint of the Mellon endpoint) and the Response URL which is the return to RT's login page.

![adfs logount endpoint URL](/images/adfs4.png)

Finally select the Signature tab and import the Certificate that Mellon created.

### ADFS Claims Rules
Once you've configured the association, you'll need to add claims rules which specify how ADFS sends information to the SP, like what format and information to include in the assertion. Since RT is almost always based on email address, we'll use that as our NameID for simplicity sake.

Select your RP and click Edit Claim Rules. We'll need to create two rules, one that selects the email address from LDAP (Active Directory) and the next that transforms the email address into the name ID:

![adfs transform claim wizard](/images/adfs5.png)
![adfs email attribute selection](/images/adfs6.png)
Then create a rule to Transform an Incoming Claim:

![adfs transform rule](/images/adfs7.png)
![adfs claim transformation](/images/adfs8.png)

It's very important to use 'transient identifier' as the outgoing nameID format, since that's what Mellon uses by default to consume the NameID and identify the user.

You can add additional claims rules to pass through groups, phone numbers, or pretty much anything else stored in AD.

### Troubleshooting SAML

I highly recommend using a SAML aware plugin like SAML Message Decoder for your browser (chrome and firefox versions of the plugins are available). This way you can see if there are errors between ADFS and  your Mellon configuration.

Things to look for:
```
<Subject> <NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient">user@domain.com</NameID>
```
The NameID is the most important parameter. This is your user ID according to SAML and most often corresponds to an email address. The format must match on both sides of the federation. For historical reasons, I recommend transient, since it's Mellon's default format. When you're using ADFS the default NameID is your Windows login name (DOMAIN\user) and if you've misconfigured the claim rules above, you'll probably get these errors.
```
<samlp:Status> <samlp:StatusCode Value="urn:oasis:names:tc:SAML:2.0:status:Success" /></samlp:Status>
```

StatusCode generally describes the status or error that occurred. Most often these are invalidNameIDFormat which indicates the NameID format does not match

## RT Configuration

RT is pretty much an out-of-the box configuration with external authentication and fallback to RT login configured. External auth should be included in RT4.4 so no additional modules are necessary. This provides us with a lot of flexibility in authenticating the user (either pass through REMOTE_USER or try a login form). RT handles the authenticated session internally, so it can act independently of the webserver's authentication if needed. I'm assuming you've setup RT to the point where you can login to it. I'm also assuming you're running RT under the root directory, so if you're running under /rt or some other directory be sure modify those values below.

### RT Authentication Configuration
In RT_SiteConfig.pm make sure WebRemoteUserAuth and WebFallbacktoRTLogin are enabled:
```
Set( $WebRemoteUserAuth, 1);
Set( $WebFallbackToRTLogin, 1);
```
Optionally, you can set web remote user autocreate. This may be relatively safe since you are authenticating users against an IdP that you own, so presumably everyone who authenticates is a user you know. This could probably be expanded significantly [in an extension similar to the rt-authn-OpenID module on Github](https://github.com/bestpractical/rt-authen-openid).
```
Set($WebRemoteUserAutocreate, 1);
```
Also you must set your referrer whitelist to your ADFS server to allow the redirect to work properly
```
Set( @ReferrerWhitelist, qw(adfs.example.com:443));
```
Otherwise you can use RT's built-in ExternalAuth module to import from LDAP or  manually create users in RT, which giv es you the most control but is highest administrative burden (this is what I did since I only have a handful of actual RT users).

### RT Login Information
RT needs a slight customization to handle the SAML login, since we want to be able to switch between RT and SAML logins. In my configuration the user is prompted for what to do and not automatically logged-in via SSO. The login override is based on the OpenID module with some modifications

The Login element (/opt/rt4/share/html/Elements) controls what's displayed on RT's login page, and we're [going to create a callback](https://rt-wiki.bestpractical.com/wiki/MyCallbacks) to Default that adds our logon button to the default login page when webremotelogin is enabled.
After updating the custom callback don’t forget to [clear the Mason cache (delete /opt/rt4/var/mason_data/obj/*)](https://rt-wiki.bestpractical.com/wiki/CleanMasonCache), and restart Apache.

Create the following as the file /opt/rt4/local/html/Callbacks/SSO/Elements/Login/Default
```
<%init>
return unless (RT->Config->Get("WebRemoteUserAuth"));
</%init>
<h3><&|/l&>Login with <em>Windows Account</em></&></h3>
<p><% loc('Select Login with Windows to login automatically with your Windows account') %> </p>
<div class="button-row">
         <span class="input">
           <input type="button"
onclick="location.href='/auth/login?ReturnTo=/?next=<% $DECODED_ARGS->{next} %>';" class="button" value="<&|/l&>Login with Windows</&>" /></span>
</div>
```

This will display the selection as an additional button and will just direct to the URL on click. The next argument is a hash of the URL the browser will return to when authentication is completed and I should probably add some checking to see if it's null. The button is changed to type button so it doesn't try to submit the form, just redirect to the SAML login target. If all goes well it should look like this:

![RT logon dialog](/images/adfs-logon.png)

Another way to handle the SAML login (if you don't need the RT login prompt) would be to set the Require valid-user directive in Apache's configuration and redirect to the login using an override of the 401 page. You could also handle this with a meta refresh, js refresh or redirect, or any other way to get the person to the Mellon login endpoint. Unfortunately there's no easy way to leverage the 'next' querystring in this case so users will be dumped to the root after authentication rather than returning to their original link.
```
ErrorDocument 401 "\
<html>\
<title>Access Restricted</title>\ <body>\
<h1>Login with your Windows account.</h1>\
<p>\
<a href=\"/auth/login?ReturnTo=/\"><strong>Click here to login via single sign-on<strong>
</a><br /><br /><br />\
        </p>\
        </body>\
        </html>"
```

### RT Logout
Logging-out requires a little more code, since you have to coordinate two systems being logged-out at the same time: RT's internal session manager and the webserver's REMOTE_USER variable, which is controlled by Mellon.

By default, if you click the standard logout, it clears the RT session, but refreshes back to logged-in since REMOTE_USER is still populated. Furthermore, we want to clear the SSO session at the same time, since logout in SAML should end-up on an IdP URL to really correctly clear the user's session.

The sequence should be (technically this will be an SP-initiated logout):
* RT logout  (clear RT's session)
* Mellon logout (clear the SSO session locally)
* SAML logout (redirect the user to the logout target in the IdP)

To do this, we'll have to create another override, this time for the Logout.html page

Note the callback called 'beforesessiondelete' and assuming by the name we execute our callback just before the RT session is terminated:
```
# Allow a callback to modify the URL we redirect to, which is useful for
# external webauth systems
$m->callback( %ARGS, CallbackName => 'ModifyLoginRedirect', URL => \$URL );
$m->callback( %ARGS, CallbackName => 'BeforeSessionDelete' );
```

If we override beforesessiondelete with a callback that redirects to the SP logout URL, we trigger the SP logout mechanism which clears REMOTE_USER. This elegantly clears the RT session, the Apache/Mellon session, and the IdP login session to the application!

Create the following file /opt/rt4/local/html/Callbacks/SSO/NoAuth/Logout.html/BeforeSessionDelete and add the appropriate logout endpoints (note the callout function override is identified as a file) Window.location.replace() preserves the referrer headers so everything *should* line up properly.
```
<script type="text/javascript"> window.location.replace("/auth/logout?ReturnTo=/NoAuth/Login.html") </script>
```
One thing to note: the Logout URLs need to reflect your RT's logon page as the final response URL in ADFS, otherwise ADFS won't redirect properly to the RT login page.
* The Trusted URL is the referrer of a logout request that will come from the SP to initiate logout
* Response URL is the referral the ADFS server will return to the browser to complete the logout (this should be /NoAuth/Login.html to allow re-logins to the app)

![adfs logout endpoint](/images/adfs-logout.png)

## References and More information
Some references that helped with the creation of this

https://mattslifebytes.com/tag/mod_auth_mellon/ General intro to Mellon with examples

https://github.com/Uninett/mod_auth_mellon/wiki/GenericSetup Mellon's official basic setup article