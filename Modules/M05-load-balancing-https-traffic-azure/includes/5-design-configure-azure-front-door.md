

Azure Front Door is a global, scalable entry-point that uses the Microsoft global edge network to create fast, secure, and widely scalable web applications. With Front Door, you can transform your global consumer and enterprise applications into robust, high-performing personalized modern applications with contents that reach a global audience through Azure.

![Azure Front Door layout diagram](../media/front-door-visual-diagram.png)

Front Door works at Layer 7 (HTTP/HTTPS layer) using anycast protocol with split TCP and Microsoft's global network to improve global connectivity. Based on your routing method you can ensure that Front Door will route your client requests to the fastest and most available application backend. An application backend is any Internet-facing service hosted inside or outside of Azure. Front Door provides a range of [traffic-routing methods](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-routing-methods) and [backend health monitoring options](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-health-probes) to suit different application needs and automatic failover scenarios. Like [Traffic Manager](https://docs.microsoft.com/en-us/azure/traffic-manager/traffic-manager-overview), Front Door is resilient to failures, including failures to an entire Azure region.

### What is the difference between Azure Front Door and Azure Application Gateway?

While both Front Door and Application Gateway are layer 7 (HTTP/HTTPS) load balancers, the primary difference is that Front Door is a global service whereas Application Gateway is a regional service. Front Door can load balance between your different scale units/clusters/stamp units across regions, and Application Gateway allows you to load balance between your VMs/containers that is within the scale unit.

## Configure redirection rules in Front Door

After establishing a connection and completing a TLS handshake, when a request lands on a Front Door environment one of the first things that Front Door does is determine which routing rule to match the request to and then take the defined action in the configuration. 

### Front Door route rules configuration structure

A Front Door routing rule configuration is composed of two major parts: a "left-hand side" and a "right-hand side". Front Door matches the incoming request to the left-hand side of the route. The right-hand side defines how Front Door processes the request.

**Incoming match** 

The following properties determine whether the incoming request matches the routing rule (or left-hand side):

- HTTP Protocols (HTTP/HTTPS)

- Hosts (for example, www.foo.com, *.bar.com)

- Paths (for example, /*, /users/*, /file.gif)

These properties are expanded out internally so that every combination of Protocol/Host/Path is a potential match set.

**Route data** 

Front Door speeds up the processing of requests by using caching. If caching is enabled for a specific route, it uses the cached response. If there is no cached response for the request, Front Door forwards the request to the appropriate backend in the configured backend pool.

**Route matching**

Front Door attempts to match to the **most-specific match first** looking only at the left-hand side of the route. It first matches based on HTTP protocol, then Frontend host, then the Path.

- **Frontend host matching:** 
  - Look for any routing with an exact match on the host. 

  - If no exact frontend hosts match, reject the request and send a 400 Bad Request error.

- **Path matching:** 
  - Look for any routing rule with an exact match on the Path.

  - If no exact match Paths, look for routing rules with a wildcard Path that matches.

  - If no routing rules are found with a matching Path, then reject the request and return a 400: Bad Request error HTTP response.

**If there are no routing rules for an exact-match frontend host with a catch-all route Path (/*), then there will not be a match to any routing rule**.

Azure Front Door redirects traffic at each of the following levels: protocol, hostname, path, query string. These functionalities can be configured for individual microservices since the redirection is path-based. This can simplify application configuration by optimizing resource usage and supports new redirection scenarios including global and path-based redirection.

![Azure Portal configure route details](../media/front-door-url-redirect.png)

### Redirection types

A redirect type sets the response status code for the clients to understand the purpose of the redirect. The following types of redirection are supported:

| **Redirection type** | **Action**         | **Description**                                              |
| -------------------- | ------------------ | ------------------------------------------------------------ |
| 301                  | Moved permanently  | Indicates that the target resource has been assigned a new permanent URI. Any future references to this resource will use one of the enclosed URIs. Use 301 status code for HTTP to HTTPS redirection. |
| 302                  | Found              | Indicates that the target resource is temporarily under a different URI. Since the redirection can change on occasion, the client should continue to use the effective request URI for future requests. |
| 307                  | Temporary redirect | Indicates that the target resource is temporarily under a different URI. The user agent MUST NOT change the request method if it does an automatic redirection to that URI. Since the redirection can change over time, the client ought to continue using the original effective request URI for future requests. |
| 308                  | Permanent redirect | Indicates that the target resource has been assigned a new permanent URI. Any future references to this resource should use one of the enclosed URIs. |




### Redirection protocol

You can set the protocol that will be used for redirection. The most common use case of the redirect feature is to set HTTP to HTTPS redirection.

- **HTTPS only:** Set the protocol to HTTPS only, if you're looking to redirect the traffic from HTTP to HTTPS. Azure Front Door recommends that you should always set the redirection to HTTPS only.

- **HTTP only:** Redirects the incoming request to HTTP. Use this value only if you want to keep your traffic HTTP that is, non-encrypted.

- **Match request:** This option keeps the protocol used by the incoming request. So, an HTTP request remains HTTP and an HTTPS request remains HTTPS post redirection.

### Destination host

As part of configuring a redirect routing, you can also change the hostname or domain for the redirect request. You can set this field to change the hostname in the URL for the redirection or otherwise preserve the hostname from the incoming request. So, using this field you can redirect all requests sent on https://www.contoso.com/* to https://www.fabrikam.com/*.

### Destination path

For cases where you want to replace the path segment of a URL as part of redirection, you can set this field with the new path value. Otherwise, you can choose to preserve the path value as part of redirect. So, using this field, you can redirect all requests sent to https://www.contoso.com/\* to https://www.contoso.com/redirected-site.

### Destination fragment

The destination fragment is the portion of URL after '#', which is used by the browser to land on a specific section of a web page. You can set this field to add a fragment to the redirect URL.

### Query string parameters

You can also replace the query string parameters in the redirected URL. To replace any existing query string from the incoming request URL, set this field to 'Replace' and then set the appropriate value. Otherwise, keep the original set of query strings by setting the field to 'Preserve'. As an example, using this field, you can redirect all traffic sent to https://www.contoso.com/foo/bar to https://www.contoso.com/foo/bar?&utm_referrer=https%3A%2F%2Fwww.bing.com%2F.

## Configure rewrite policies

Azure Front Door supports URL rewrite by configuring an optional Custom Forwarding Path to use when constructing the request to forward to the backend. By default, if a custom forwarding path isn't provided, the Front Door will copy the incoming URL path to the URL used in the forwarded request. The Host header used in the forwarded request is as configured for the selected backend. Read Backend Host Header to learn what it does and how you can configure it.

The powerful part of URL rewrite is that the custom forwarding path will copy any part of the incoming path that matches to a wildcard path to the forwarded path (these path segments are the green segments in the example below):

![URL rewrite example](../media/front-door-rewrite-policy.png)

## Configure health probes, including customization of HTTP response codes

To determine the health and proximity of each backend for a given Front Door environment, each Front Door environment periodically sends a synthetic HTTP/HTTPS request to each of your configured backends. Front Door then uses these responses from the probe to determine the "best" backend resources to route your client requests.

Since Front Door has many edge environments globally, health probe volume for your backends can be quite high - ranging from 25 requests every minute to as high as 1200 requests per minute, depending on the health probe frequency configured. With the default probe frequency of 30 seconds, the probe volume on your backend should be about 200 requests per minute.

### Supported HTTP methods for health probes

Front Door supports sending probes over either HTTP or HTTPS protocols. These probes are sent over the same TCP ports configured for routing client requests and cannot be overridden.

Front Door supports the following HTTP methods for sending the health probes:

**GET:** The GET method means retrieve whatever information (in the form of an entity) is identified by the Request-URI.

**HEAD:** The HEAD method is identical to GET except that the server MUST NOT return a message-body in the response. Because it has lower load and cost on your backends, for new Front Door profiles, by default, the probe method is set as HEAD.

### Health probe responses

The following table describes responses to the health probe:

| **Response**       | **Description**                                              |
| ------------------ | ------------------------------------------------------------ |
| Determining Health | A 200 OK status code indicates the backend is healthy. Everything else is considered a failure. If for any reason (including network failure) a valid HTTP response isn't received for a probe, the probe is counted as a failure. |
| Measuring Latency  | Latency is the wall-clock time measured from the moment immediately before the probe request is sent to the moment the last byte of the response is received. A new TCP connection is used for each request, so this measurement isn't biased towards backends with existing warm connections. |




Azure Front Door uses the same three-step process below across all algorithms to determine health.

1. Exclude disabled backends.
2. Exclude backends that have health probe errors:

   - This selection is done by looking at the last n health probe responses. If at least x are healthy, the backend is considered healthy.

   - n is configured by changing the SampleSize property in load-balancing settings.

   - x is configured by changing the SuccessfulSamplesRequired property in load-balancing settings.
3. For the sets of healthy backends in the backend pool, Front Door additionally measures and maintains the latency (round-trip time) for each backend.

If you have a single backend in your backend pool, you can choose to disable the health probes reducing the load on your application backend. Even if you have multiple backends in the backend pool but only one of them is in enabled state, you can disable health probes.

## Secure Front Door with SSL 

Use the HTTPS protocol on your custom domain (for example, https://www.contoso.com), you ensure that your sensitive data is delivered securely via TLS/SSL encryption when it's sent across the internet. When your web browser is connected to a web site via HTTPS, it validates the web site's security certificate and verifies that it was issued by a legitimate certificate authority. This process provides security and protects your web applications from attacks.

Some of the key attributes of the custom HTTPS feature are:

- **No extra cost:** There are no costs for certificate acquisition or renewal and no extra cost for HTTPS traffic.

- **Simple enablement:** One-click provisioning is available from the Azure portal. You can also use REST API or other developer tools to enable the feature.

- **Complete certificate management:** All certificate procurement and management is handled for you. Certificates are automatically provisioned and renewed before expiration, which removes the risks of service interruption because of a certificate expiring.

You can enable the HTTPS protocol for a custom domain that's associated with your Front Door under the frontend hosts section.

For more information on how to configure HTTPS on Front door, see [Tutorial - Configure HTTPS on a custom domain for Azure Front Door | Microsoft Docs](https://docs.microsoft.com/en-us/azure/frontdoor/front-door-custom-domain-https).

 

## quiz title: Check your knowledge



## Multiple Choice 

What is the difference between Azure Front Door and Azure Application Gateway?
(x) Front Door is a global service, Application Gateway is a regional service.  {{Correct, Front Door and Application Gateway are layer 7 (HTTP/HTTPS) load balancers, Front Door is a global service whereas Application Gateway is a regional service.}} 
( ) Front Door is a regional service, Application Gateway is a global service.  {{Incorrect, Front Door and Application Gateway are layer 7 (HTTP/HTTPS) load balancers, Front Door is a global service whereas Application Gateway is a regional service.}} 
( )  Front Door uses health probes to montior the health of backends, Application Gateway does not.  {{Incorrect, Both Front Door and Application gatweay use health probes to monitor the health of backends.}}

## Multiple Choice 

Front Door route rules determine whether the incoming request matches the routing rule and route traffice accordingly. What properties are matched?
(x) HTTP protocols (HTTP/HTTPS), Hosts, and Paths.  {{Correct, When evaluting routing rules, Front Door looks for matches in HTTP protocols (HTTP/HTTPS), Hosts, and Paths.}} 
( ) HTTP protocols (HTTP/HTTPS), Hosts, and time stamp.  {{Incorrect, When evaluting routing rules, Front Door does not look for matches in time stamps.}} 
( ) Hosts, paths, and user certificates.  {{Incorrect, When evaluting routing rules, Front Door does not look for matches in user certificates.}}

