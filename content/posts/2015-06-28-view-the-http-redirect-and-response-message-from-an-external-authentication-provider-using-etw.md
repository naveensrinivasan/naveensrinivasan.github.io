---
title: View the http redirect and response message from an external authentication provider using ETW
author: admin
type: post
date: 2015-06-29T02:57:26+00:00
url: /?p=1510
categories:
  - ETW
tags:
  - ETW
  - IIS

---
Recently I had to troubleshoot messages that were being sent from an web application hosted on IIS to an external authentication provider. The logs from the application wasn&#8217;t something closer to the metal and wasn&#8217;t really providing all the details. I really wanted something like fiddler for the webserver. I could have a ran network traces to troubleshoot the issue but the problem was it wasn&#8217;t happening consistently. It was sporadic. I knew there would be ETW traces that would have this information. The IIS web logs don&#8217;t capture this information.

Here is a example of the SAML authentication process

[<img class=" size-full wp-image-1511 aligncenter" src="https://naveensrinivasan.files.wordpress.com/2015/06/500px-saml.jpg" alt="500px-SAML" width="500" height="469" />][1]

In the application I was working with, IIS was the relying party and the user was to be authenticated with Identity Provider.

I wanted to troubleshoot the &#8220;AuthnRequest&#8221; and &#8220;Auth Resp&#8221; from and to the IIS. This can be applied to any external authentication like credit card authentication.

I fired my favorite tool <a href="http://blogs.msdn.com/b/vancem/archive/tags/perfview/" target="_blank">Perfview</a> and captured all the IIS traces along with other defaults. I wasn&#8217;t really interested in the .NET Code.

Here is the command line for Perfview to the IIS Providers
  
[gist id = &#8220;86a6d7daac73484ef504&#8221;]

If for some reason that does not work.  You could always use the additional providers in Perfview and add these providers which are IIS and HTTP providers.

[gist id = &#8220;5ac34bdd047d2d80cc44&#8221;]

I let perfview do its job and then stopped the trace when there was an issue.

Here are the ETW events that capture the SAML Request that was sent from IIS to the IDP

Event Name

  1. IIS\_Trace/IISGeneral/GENERAL\_RESPONSE_HEADERS
  2. Microsoft-Windows-IIS/EventID(47)
  3. IIS\_Trace/IISGeneral/GENERAL\_RESPONSE\_ENTITY\_BUFFER
  4. Microsoft-Windows-IIS/EventID(49)
  5. IIS\_Trace/IISGeneral/GENERAL\_REQUEST_HEADERS

[<img class="alignleft size-full wp-image-1515" src="https://naveensrinivasan.files.wordpress.com/2015/06/samlrequest.jpg" alt="SamlRequest" width="660" height="231" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlrequest.jpg 1157w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlrequest-300x105.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlrequest-768x269.jpg 768w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlrequest-1024x358.jpg 1024w" sizes="(max-width: 660px) 100vw, 660px" />][2]

Here are the ETW events that capture the SAML Response that was being posted from the IDP to the IIS

  1. IIS\_Trace/IISGeneral/GENERAL\_REQUEST_ENTITY
  2. Microsoft-Windows-IIS/EventID(51)

[<img class="alignleft size-full wp-image-1514" src="https://naveensrinivasan.files.wordpress.com/2015/06/samlresponse.jpg" alt="samlresponse" width="660" height="39" srcset="https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlresponse.jpg 1177w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlresponse-300x18.jpg 300w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlresponse-768x46.jpg 768w, https://www.naveensrinivasan.com/wp-content/uploads/2015/06/samlresponse-1024x61.jpg 1024w" sizes="(max-width: 660px) 100vw, 660px" />][3]

With this I was able to troubleshoot message that was being sent and received to the IIS.

&nbsp;

 [1]: https://naveensrinivasan.files.wordpress.com/2015/06/500px-saml.jpg
 [2]: https://naveensrinivasan.files.wordpress.com/2015/06/samlrequest.jpg
 [3]: https://naveensrinivasan.files.wordpress.com/2015/06/samlresponse.jpg