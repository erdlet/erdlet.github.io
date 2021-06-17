---
layout: post
title:  "Getting started with Cookies in JAX-RS"
date:   2021-06-17 09:00:00 +0200
categories: jakarta-ee
author: Tobias Erdle
---

This post will show you the basics of using Cookies in JAX-RS / Jakarta RESTful Web Services and pitfalls I fell into.
Because most of the projects still use the `javax` namespace, I'll use it for full qualified class names. Also I'll use `JAX-RS`
when I refer to the API.

## The basics: `Cookie`, `NewCookie` and `@CookieParam`

Basically, the API's Cookie mechanism consist of two _immutable_ classes and one annotation:

- `javax.ws.rs.core.Cookie` [JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/ws/rs/core/Cookie.html)
- `javax.ws.rs.core.NewCookie` [JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/ws/rs/core/NewCookie.html)
- `javax.ws.rs.CookieParam` [JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/ws/rs/CookieParam.html)

The basic information of a Cookie is stored in `javax.ws.rs.core.Cookie`, which contains name, value, version, domain and the path on which the Cookie is valid.
Maybe someone notices, that some important attributes, e.g. `max-age` or `secure`, are missing. These information can be set in `javax.ws.rs.core.NewCookie`.
But why the difference? `javax.ws.rs.core.Cookie` is meant to be a kind of request DTO (Data Transfer Object), where in most cases the name and value are important to the
service processing the Cookie. On the other side, `javax.ws.rs.core.NewCookie` is designed to be transferred with the HTTP response, so it acts as a response DTO. Because the client processing the response needs more data than the service receiving the HTTP request, the `NewCookie` class contains more information.

But what is `@CookieParam` doing? This will be shown in the next sections.


## Adding a new Cookie to the response

First lets have a look on how to add a Cookie into the HTTP response, so e.g. a browser can use it. Therefore the following code snippet will be used:

```java
[1] import javax.ws.rs.core.Response;
import javax.ws.rs.core.NewCookie;

@GET
public Response addCookie() {
	[2] final NewCoookie cookie = new NewCookie("myCookie", "myCookieValue");

	[3] return Response.ok().cookies(cookie).build();
} 
```

In `[1]` the JAX-RS `Response` and `NewCookie`classes are imported, which provide convenient methods for building the HTTP response and the Cookie. In `[2]` a new, insecure Session-Cookie  will be created. As a last step (`[3]`), the `Response` is built and
transfers the Cookie passed in the `cookies()` method to the client.

But what can be done when a more advanced setup is needed? The answer are the different constructors of `NewCookie`. Lets create a secure, HTTP only Cookie with maximum age set:

```java
import javax.ws.rs.core.Response;
import javax.ws.rs.core.NewCookie;

@GET
public Response addCookie() {
	final NewCoookie cookie = NewCookie("myCookie", "myCookieValue", "/app", "example.com", "An example cookie with advanced configuration", 3600, true, true);

	return Response.ok().cookies(cookie).build();
} 
```

In this example, the constructor `NewCookie(String name, String value, String path, String domain, String comment, int maxAge, boolean secure, boolean httpOnly)` is used to configure the additional security settings. The Cookie will be valid for one hour (because of `maxAge` set to 3600 seconds) on path `/app` (and subpaths) on the domain `example.com` (and subdomains). Also, it is secure and HTTP only, which means it is transferred only over HTTPS connections. 

## Reading Cookies from the request

After transferring a Cookie to the client, an application normally wants to use its data, so the Cookie value must be read. To achieve this, there are two simple approaches existing in JAX-RS:

1. use `@CookieParam` for injecting the `javax.ws.rs.core.Cookie` into the method
2. use `@CookieParam` for injecting the value of the Cookie directly into the method

However, there is a third one, which is based on the `javax.ws.rs.core.HttpHeaders` interface, which gives programmatic access to the request's Cookies. I won't show this in detail, as I don't think that someone needs this very often. Also, there are a lot of other blogs or tutorial showing it, so I don't need to be an additional one ;)

Lets get back to approach _1_ and _2_. The following snippet shows approach _1_, which injects the whole Cookie into the JAX-RS method:

```java
import javax.ws.rs.core.Response;
[1] import javax.ws.rs.core.Cookie;
	import javax.ws.rs.CookieParam;

@GET
public Response addCookie(
	[2] @CookieParam("myCookie") final Cookie myCookie) {

	[3] final String value = myCookie.getValue();

	//...
} 
``` 

Instead of `NewCookie`, the classes `Cookie` and `CookieParam` are imported at `[1]`. At marker `[2]` the Cookie gets injected into the method by using its name as value of `@CookieParam`. This annotations searches in the HTTP headers for a Cookie with that name and creates the `Cookie` instance out of its values. In `[3]` the application just accesses the value of the Cookie which can further processed later. This method has the advantage, that other attributes of the Cookie can be processed by the method too. But in case we are only interested in the value, `@CookieParam` can help with a shortcut:

```java
import javax.ws.rs.core.Response;
[1] import javax.ws.rs.CookieParam;

@GET
public Response addCookie(
	[2] @CookieParam("myCookie") final String myCookieValue) {

	//...
} 

```

In this case, only the annotation is imported at `[1]`. Instead of a `Cookie`, the `@CookieParam` will now be applied to a `String` parameter, which indicates the runtime to grab the Cookie's value. If other data types shall be used, a look into
[JavaDoc](https://javaee.github.io/javaee-spec/javadocs/javax/ws/rs/CookieParam.html) will show all possible types the API can inject.

## Renewing a Cookie's value

Sometimes, the value of an existing Cookie has to be updated, e.g. if a Cookie stores an users preferred language. Because the JAX-RS classes are immutable, working with a combination out of `Cookie` and `NewCookie` is necessary:

```java
[1] import javax.ws.rs.core.Response;
	import javax.ws.rs.core.Cookie;
	import javax.ws.rs.core.NewCookie;
	import javax.ws.rs.CookieParam;

@GET
public Response addCookie(
	[2] @CookieParam("myCookie") final Cookie myCookie) {

	[3] final String value = myCookie.getValue();

	[4] final String newValue = value + "New";

	[5] final NewCoookie updatedCookie = NewCookie("myCookie", newValue, "/app", "example.com", "An example cookie with advanced configuration", 3600, true, true);

	[6] return Response.ok().cookies(updatedCookie).build();

	//...
} 
``` 

In this example, all classes used before are imported at `[1]`. At `[2]` the current Cookie is injected and the value read at `[3]`. In `[4]` the current value gets updated and set into a `NewCookie` with the same settings as before at `[5]`. At the end, the Cookie is added into the `Response` and transferred to the client. The browser should now update the Cookie. In case this doesn't happen, have a look at the **Troubleshooting** section below.

## Deleting a Cookie

In case of e.g. a Cookie storing favorites, it may be intended to delete the Cookie after the last favorite was removed. But Cookies don't have an explicit flag or setting for marking it for deletion, so the `maxAge` attribute has to be used.

```java
import javax.ws.rs.core.Response;
import javax.ws.rs.core.Cookie;
import javax.ws.rs.core.NewCookie;
import javax.ws.rs.CookieParam;

@GET
public Response addCookie(@CookieParam("myCookie") final Cookie myCookie) {

	...

	[1] final NewCoookie cookieForDeletion = NewCookie("myCookie", "", "/app", "example.com", "An example cookie with advanced configuration", 0, true, true);

	[2] return Response.ok().cookies(cookieForDeletion).build();

	//...
} 
``` 

In this example, which is similar to the "update" scenario, the Cookie responded to the client shall be deleted. Therefore, the `maxAge` attribute of the Cookie has to be set to `0`. This indicates the client, that the Cookie shall be invalidated and removed.
To ensure that the Cookie gets deleted even in old and unsupported browsers, the `expirationDate` attribute can be set to a date in the past too.

## Troubleshooting

### I want to inject `NewCookie` by means of `@CookieParam` but it doesn't work. Whats wrong?

It seems desirable to inject `NewCookie`, because it has more attributes than `Cookie` and copy constructors could be easily implemented. Unfortunately, `NewCookie` can't be injected, because the browser submits only the core attributes defined in `Cookie`.
Nevertheless, if you try to inject `NewCookie`, the attributes will be cluttered and won't be useful.

### Cookie not visible in Browser

There can be two reasons for this behavior:

#### 1. The `path` and / or `domain` attribute are not matching the path and domain opened in the browsers. 

To resolve this, adapt these values in your `NewCookie`. Because often there are different domains and paths for different environments, it could make sense to use a factory to create the Cookies and configure `path`, `domain` by means of MicroProfile Config.

#### 2. The `secure` flag is set to `true` but the connection is unsecure. 

Probably the `NewCookie` attribute `secure` is set to `true` but the connection the browser opened is unsecure, so the Cookie won't be stored. To resolve this, one can simply set the Cookie to be unsecure, which is OK for local development environments, but should be avoided for productive environments. To have a configurable solutions, MicroProfile Config and a factory can be used to create `NewCookie`s which set the `secure` attribute depending on the current environment.

### Cookie won't get deleted

In this case I ran into the issue that `domain` and `path` didn't match. To fix this, the `domain` and `path` need to match the values openend in the browser. Another problem could be a browser not supporting `maxAge`, although this should not be the case with modern browsers. In case of doubt, the documentation of the browser should help to exclude or confirm this.

## More information on Cookies

- [RFC 2109 - HTTP State Management Mechanism](https://www.ietf.org/rfc/rfc2109.txt)
- [MDN: Using HTTP Cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
