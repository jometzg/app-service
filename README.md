# Access Restrictions - recap
When building a service that uses several web apps, there is often the need to lock down some web apps from only receiving request of other Azure web apps. Azure app services have for a long time had an access restriction feature where requests in-bound to a web app can be restricted by IP address or IP address range. This is form of access restriction does not restrict to specific web apps, but it is nevertheless useful. 

Often though, it can be difficult to debug issues with access restriction. 
![Two web apps](https://github.com/jometzg/app-service/blob/master/web-to-web.png)

Is it a problem with the sender or the recipient?

This article hopes to provide some pointers.

# TL;DR
At a sender web app, if you want to know the actual IP address of that app service that is issuing a request, go in Kudu of the web app and open a CMD prompt:

`curl https://api.ipify.org?format=json`

This will then return that sender web app's IP address.
![Curl](https://github.com/jometzg/app-service/blob/master/curl.png)

At the receipient's end, its not quite so easy. In "Diagnostic Setting",  "AppServiceHTTPLogs" diagnostic logs don't appear to report blocked requests for a destination web app. So, to debug these, it is best to switch off access restrictions and then look through the logs for your requests - these will include the sender's IP address.

# More Details
Azure app service is a multi-tenant service for hosting web applications. This multi-tenancy provides many advantages for customers in terms of the density of web apps, but it also means that a number of web apps will share common in-bound and out-bound IP addresses.

App services provide a rich set of features, one of which is access restrictions. Access restrictions allow the definition of access control rules that allow or deny requests to a web app. These are based on IP addresses, IP address ranges and VNet subnets.

App services also provide rich diagnostic capabilities from diagnostic logs to the [Kudu engine](https://github.com/projectkudu/kudu/wiki)

We will look in details on how to setup diagnostics to look at inbound requests and then how to use the Kudu engine to send outbound requests from a web app - this can be used to discover routing issues and to establish the outbound IP address of the web app at that point in time.

# Diagnostics
In order to see what requests are inbound to a web app, requires that diagnostics is turned on for that web app. Diagnostics can be sent to storage, event hubs and log analytics.

Let's keep things simple and setup diagnostics to an Azure storage account. Diagnostics can be setup here:
![Set diagnostics to storage](https://github.com/jometzg/app-service/blob/master/web-app-set-storage-diagnostics.png)

The resultant diagnostics are created in a blob container:
![Diagnostics blob container](https://github.com/jometzg/app-service/blob/master/diagnostics-blob-container.png)

The actual blobs are under a fair number of folders:
![Diagnostics blob](https://github.com/jometzg/app-service/blob/master/diagnostics-blob-document.png)

Finally, if you download the blob, you can see each of the requests, which includes the sender's IP address.
![Diagnostics blob contents](https://github.com/jometzg/app-service/blob/master/diagnostics-blob-document-contents.png)

So you don't get lost, it may be better to send the requests to an uncommon URL, to make your searching easier. The diagnostics logs are ordered by date and time and get amended every 5 mimutes or so - so timing is important too when searching.

As stated earlier, these logs to do show denied requests from when an access restriction is set, so it may be better to remove the access restruction, run the test and then put the restriction back - this may not be a recommended approach for a production environment, however :-)

** There may be soon an extra log "AppServiceIPSecAuditLogs" which may show denied requests. This will mean that access restrictions can be tested without switching off the feature. **

# Kudu
Every web app has another endpoint which has a large number of useful management features. This can be found in the Azure portal under "Advanced Tools" and has a separate URL of https://web-app-name.scm.azurewebsites.net.

Kudu has a very rich set of features, which can be found [here](https://github.com/projectkudu/kudu/wiki)

You can also open up a command prompt to the Kudu console and then interact with Kudu on the command line. The most useful of these are:

* nslookup - DNS name lookup
* tcpping - a replacement for ping
* curl - send HTTP requests

Curl is really useful in that you can send HTTP requests elsewhere and look at the response. This is very good for debugging outbound connectivity issues from your web app to the outside world.

It is often useful to be able to know exactly what IP address your web app is using when making external requests. For some time, the portal, in the web app's "Properties" section a list of potential IP addresses that an app service may use. See below:

![Web app outbound IPs](https://github.com/jometzg/app-service/blob/master/web-app-properties-outbound.png)

But it is often more useful to know exactly which of these is being used. One way I have done this in the past is to write another small web app which, on receipt of a request, looks at the request headers and returns the sender's IP address. This works, but is extra work. As it is often better to buy/use than build, it would be handy if someone else has done this before. This is where the likes of the web site https://www.ipify.org/ comes to the rescuse. This has an easy to consume API request:

https://api.ipify.org?format=json which will return the caller's IP address as a nice piece of JSON. For example {"ip":"98.207.254.136"}

So in the Kudu console, this would look like below:
![A curl request in Kudu](https://github.com/jometzg/app-service/blob/master/kudu-cmd-curl.png)


# Summary
Azure web apps provide an easy on-ramp for deploying web and API services to Azure. But there is also considerable power behind the scenes. With the use of diagnostics logs and the Kudu console many web app issues can be debugged easily.
