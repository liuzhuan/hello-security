# The Subject Alternative Name Field Explained

https://www.digicert.com/subject-alternative-name.htm

The **Subject Alternative Name** field lets you specify additional host names (sites, IP addresses, common names, etc.) to be protected by a single SSL Certificate, such as a Multi-Domain (SAN) or Extend Validation Multi-Domain Certificate.

## Background

The Subject Alternative Name extension was a part of the X509 certificate standard before 1999, but it wasn't until the launch of Microsoft Exchange Server 2007 that it was commonly used; this change makes good use of Subject Alternative Names by simplifying server configurations. Now Subject Alternative Names are widely used for environments or platforms that need to secure multiple sites (names) across different domains/subdomains.

## What Can You Do with Subject Alternative Names?

**Secure Host Names on Different Base Domains in One SSL Certificate**: A Wildcard Certificate can protect all first-level subdomains on an entire domain, such as *.example.com. However, a Wildcard Certificate cannot protect both www.example.com and www.example.net.

**Virtual Host Multiple SSL Sites on a Single IP Address**: Hosting multiple SSL-enabled sites on a single server typically requires a unique IP address per site, but a Multi-Domain (SAN) Certificate with Subject Alternative Names can solve this problem. Microsoft IIS and Apache are both able to Virtual Host HTTPS sites using Multi-Domain (SAN) Certificates.

**Greatly Simplify Your Server's SSL Configuration**: Using a Multi-Domain (SAN) Certificate saves you the hassle and time involved in configuring multiple IP addresses on your server, binding each IP address to a different certificate, and trying to piece it all together.

## Where Can You See Subject Alternative Names in Action?

To see an example of Subject Alternative Names, in the address bar for this page, click the green padlock in your browser to examine our SSL Certificate. In the certificate details, you will find a Subject Alternative Name extension that lists both www.digicert.com and digicert.com plus some additional SANs secured by our certificate.

Because the name digicert.com is listed in our certificate, your browser will not complain if you visit our site at https://digicert.com without the 'www' in the name.

![SAN Screenshot](https://www.digicert.com/images/d3/san_screenshot_boxed-1.png)