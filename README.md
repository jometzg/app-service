# Access Restrictions - recap
Azure app services (web apps) have for a long time had an access restriction feature where requests in-bound to the app service can be restricted by IP address or IP address range. This now includes VNet subnets, but that is another story. 

Often when using a combination of app services it can be difficult to debug issues 
![Two web apps](https://octodex.github.com/images/yaktocat.png)
- is it a problem with the sender or the recipient?

This article hopes to provide some pointers.

# TL;DR
At a sender web app, if you want to know the actual IP address of that app service that is issuing a request, go in Kudu of the web app and open a CMD prompt:

`curl https://api.ipify.org?format=json`

This will then return the senders IP address.

At the receiptient's end, its not quite so easy. In "Diagnostic Setting",  "AppServiceHTTPLogs" diagnostic logs don't appear to report blocked requests for a destination web app. So, to debug these, it is best to switch off access restrictions and then look through the logs for your requests - these will include the sender's IP address.

# More Details
Azure app service is a multi-tenant service for hosting web applications. This multi-tenancy provides many advantages for customers in terms of the density of web apps, but it also means that a number of web apps will share common in-bound and out-bound IP addresses.

App services provide a rich set of features, one of which is access restrictions. Access restrictions allow the definition of access control rules that allow or deny requests to a web app. These are based on IP addresses, IP address ranges and VNet subnets.


# Diagnostics


# Kudu
Every web app has another endpoint which has a large numner of useful management features. This can be found in the Azure portal under "Advanced Tools" and has a separate URL of https://web-app-name.scm.azurewebsites.net.

Kudu has a very rich set of features, which can be found here 

You can also open up a command prompt to the Kudu console and then interact with Kudu on the command line. 

nslookup - DNS name lookup
tcpping - a replacement for ping
curl - send HTTP requests

Curl is really useful in that you can send HTTP requests elsewhere and look at the response. This is very good for debugging outbound connectivity issues from your web app to the outside world.

It is often useful to be able to know exactly what IP address your web app is using when making external requests. For some time, the portal, in the web app's "Properties" section a list of potential IP addresses that an app service may use. See below:


But it is often more useful to know exactly which of these is being used. One way I have done this in the past is to write another small web app which, on receipt of a request, looks at the request headers and returns the sender's IP address. This works, but is extra work. As it is often better to buy/use than build, it would be handy if someone else has done this before. This is where the likes of the web site https://www.ipify.org/ comes to the rescuse. This has an easy to consume API request:

https://api.ipify.org?format=json which will return the caller's IP address as a nice piece of JSON. For example {"ip":"98.207.254.136"}

So in the Kudu console, this would look like below:

