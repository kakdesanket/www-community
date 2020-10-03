---

title: Server Side Request Forgery
layout: col-sidebar
author:
contributors:
tags: attack
auto-migrated: 1
permalink: /attacks/Server_Side_Request_Forgery

---

{% include writers.html %}

## Overview

In a Server-Side Request Forgery (SSRF) attack, the attacker can abuse
functionality on the server to read or update internal resources. The
attacker can supply or modify a URL which the code running on the
server will read or submit data to, and by carefully selecting the URLs,
the attacker may be able to read server configuration such as AWS
metadata, connect to internal services like http enabled databases or
perform post requests towards internal services which are not intended
to be exposed.

## Description

The target application may have functionality for importing data from a
URL, publishing data to a URL or otherwise reading data from a URL that
can be tampered with. The attacker modifies the calls to this
functionality by supplying a completely different URL or by manipulating
how URLs are built (path traversal etc.).

When the manipulated request goes to the server, the server-side code
picks up the manipulated URL and tries to read data to the manipulated
URL. By selecting target URLs the attacker may be able to read data from
services that are not directly exposed on the internet:

  - Cloud server meta-data - Cloud services such as AWS provide a REST
    interface on <http://169.254.169.254/> where important configuration
    and sometimes even authentication keys can be extracted
  - Database HTTP interfaces - NoSQL database such as MongoDB provide
    REST interfaces on HTTP ports. If the database is expected to only
    be available to internally, authentication may be disabled and the
    attacker can extract data
  - Internal REST interfaces
  - Files - The attacker may be able to read files using <file://> URIs

The attacker may also use this functionality to import untrusted data
into code that expects to only read data from trusted sources, and as
such circumvent input validation.

## Security controls for stopping SSRF

Simple blacklists and regular expressions applied to user input are a bad approach to mitigating SSRF. In general, blacklists are a poor means of security control. Attackers will always find methods to bypass them. In this case, an attacker can use an HTTP redirect, a wildcard DNS service such as xip.io, or even alternate IP encoding.

Whitelists and DNS resolution : The most robust way to avoid Server Side Request Forgery (SSRF) is to whitelist the DNS name or IP address that your application needs to access. If a whitelist approach does not suit you and you must rely on a blacklist, it’s important to validate user input properly. For example, do not allow requests to private (non-routable) IP addresses (detailed in RFC 1918).

However, in the case of a blacklist, the correct mitigation to adopt will vary from application to application. In other words, there is no universal fix to SSRF because it highly depends on application functionality and business requirements.

Response handling :To prevent response data leaking to the attacker, you must ensure that the received response is as expected. Under no circumstances should the raw response body from the request sent by the server be delivered to the client.

Disable unused URL schemas :If your application only uses HTTP or HTTPS to make requests, allow only these URL schemas. If you disable unused URL schemas, the attacker will be unable to use the web application to make requests using potentially dangerous schemas such as file:///, dict://, ftp:// and gopher://.

Authentication on internal services:By default, services such as Memcached, Redis, Elasticsearch, and MongoDB do not require authentication. An attacker can use Server Side Request Forgery vulnerabilities to access some of these services without any authentication. Therefore, to ensure web application security, it’s best to enable authentication wherever possible, even for services on the local network.
