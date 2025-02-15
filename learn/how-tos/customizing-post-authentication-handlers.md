---
title: "Customizing Post Authentication Handlers"
id: ""
---

In a Security enabled WaveMaker app, post-authentication the following actions are performed:

1. The Default Success Handler, which includes generation of CSRF token, storing the session context, etc., gets invoked.
2. Next, any custom authentication success handlers provided by the app developer are triggered.
3. Post authentication redirection handler will be triggered. This can either be the default redirection handler provided by WaveMaker or any custom redirection handler provided by the app developer.

This section shows how custom post-authentication success handler and custom redirection handler can be implemented.

## Custom Post-Authentication Success Handler

Post-Authentication Success Handlers, in addition to the default one, can be implemented as per app requirements. At app runtime, WaveMaker will automatically trigger these custom handlers.

Creating custom post-authentication success handler involves the following steps:

- Creation of a package structure in **src/main/java**_._
- Creating the interface implementation in that package.
- Declaring the custom post-authentication success handler implementation (along with the package name) in **project-user-spring.xml**.

Note: Multiple implementations can be provided as per your app requirements by following the above-mentioned steps for each handler.

### Creating a Package Structure

Create the package folder structure under **src/main/java**. If you want to name your package, see the following example:

[![](https://www.wavemaker.com../assets/Java-src-file.png)](https://www.wavemaker.com../assets/Java-src-file.png)

### Interface to Implement

After creating the package structure, the following interface needs to be implemented in that package for creating a custom post-authentication success handler.

public interface WMAuthenticationSuccessHandler {
void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, 
                             WMAuthentication authentication) throws IOException, ServletException;
}

For example, the following _MyCustomAuthenticationSuccessHandler_ fetches _lastAccessedTime_ of the authenticated user and sets it in the custom attributes.

// change the package name according to your requirements
package com.mycompany.myapp.security;

import com.wavemaker.runtime.security.handler.WMAuthenticationSuccessHandler;
import com.wavemaker.runtime.security.WMAuthentication;

public class MyCustomAuthenticationSuccessHandler implements WMAuthenticationSuccessHandler {

 /\*\* 
  \*It's a database service, which contains user information. 
  \*/
  @Autowired
  private UserInfoService userInfoService; 

  @Override
   public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, 
                                       WMAuthentication authentication) throws IOException, ServletException {

        UsernamePasswordAuthenticationToken authenticationToken = 
               (UsernamePasswordAuthenticationToken)authentication.getAuthenticationSource();

        String username = authenticationToken .getPrincipal();
        long  lastAccessedTime = userInfoService.getById(username).getLastAccesedTime();

       /\*\*
        \* Adding lastAccessedTime to custom attributes, which is visible both to client and server.
        \*/
        authentication.addAttribute(“lastAccessedTime” , lastAccessedTime , Attribute.AttributeScope.ALL);

       /\*\*
        \* Adding one more attribute which with scope SERVER\_ONLY
        \*/
        authentication.addAttribute(“lastValidatedTime” , System.currentTimeMillis() , 
                                Attribute.AttributeScope.SERVER\_ONLY);

   }
}

### Custom Handler Declaration

Declare the above-created custom post-authentication success handler implementation (along with the package name) in **project-user-spring.xml**.

<bean id="customAuthenticationSuccessHandler" 
      class="<package\_name>.MyCustomAuthenticationSuccessHandler"/>

At app runtime, WaveMaker will automatically trigger these custom handlers. Follow the above approach for adding multiple success handlers.

## WMAuthentication Class

WMAuthentication wrapper class holds authentication information like principal, loginTime, userId and the original authentication object. This wrapper class has the following structure:

public class WMAuthentication extends AbstractAuthenticationToken {
   private Map<String, Attribute> attributes = new HashMap<>();
   private String principal;    
   private long loginTime;
   private String userId;
   @JsonIgnore
   private transient Authentication authenticationSource;
   public WMAuthentication() {
   }
   public WMAuthentication(Authentication authenticationSource) {
   }
   @Override
   public Object getCredentials() {
       return null;
   }
   @Override
   public String getPrincipal() {
       return principal;
   }
   public Map<String, Attribute> getAttributes() {
       return attributes;
   }
   public String getUserId() {
       return userId;
   }
   public Authentication getAuthenticationSource() {
       return authenticationSource;
   }
   public long getLoginTime() {
       return loginTime;
   }

   public void setLoginTime(long loginTime) {
       this.loginTime = loginTime;
   }

   public void addAttribute(String key, Object value, Attribute.AttributeScope scope) {
       attributes.put(key, new Attribute(scope, value));
   }
}

You can add custom attributes using the _addAttribute_ method. You need to implement methods in the WMAuthenticationSuccessHandler interface and call the below method of WMAuthentication object to add any custom attributes.

public void addAttribute(String key, Object value, Attribute.AttributeScope scope) {
    attributes.put(key, new Attribute(scope, value));
}

### Adding Custom Attributes

You can attach additional information to the logged in user using the custom attribute. These attribute are made available in the logged-in user context and they can be retrieved in both UI & backend as per your needs.

#### Attribute Class

Each attribute is associated with a key, value, and scope.

public class Attribute implements Serializable{
   private AttributeScope scope;
   private Object value;
   public Attribute(AttributeScope scope, Object value) {
       this.scope = scope;
       this.value = value;
   }
   public AttributeScope getScope() {
       return scope;
   }
   public Object getValue() {
       return value;
   }
   public enum AttributeScope {
       /\*
       \*  This attributescoped variables will be visible to both client and server.
       \* \*/
       ALL,
       /\*
       \* This attributescoped variables will be visible only to the server.
       \* \*/
      SERVER\_ONLY;
   }
}

#### Attribute Scope

AttributeScope determines whether the attribute is server only property or can be visible to both client and server. You can filter out the custom attributes from being visible to the client by setting Attribute.AttributeScope property.

public enum AttributeScope {
   /\*
   \*  This attributescoped variables will be used both in backend and frontend.
   \* \*/
   ALL,
   /\*
   \* This attributescoped variables get's persisted only in backend.
   \* \*/
   SERVER\_ONLY;
}

### Attaching to the Logged-in User

You can add custom attributes using the _addAttribute_ method. You need to implement methods in the WMAuthenticationSuccessHandler interface and call the below method of WMAuthentication object to add any custom attributes.

public void addAttribute(String key, Object value, Attribute.AttributeScope scope) {
    attributes.put(key, new Attribute(scope, value));
}

## Post-Authentication Redirection Handler

Post authentication, the default Redirection Handler redirects to the appropriate [landing page](/learn/app-development/app-security/login-configuration/#landing-page) based upon the logged-in users' role.

To customize the redirection, implement the following interface and declare as a bean with id: **wmAuthenticationSuccessRedirectionHandler** in **project-user-spring.xml.**

#### Interface to implement

public interface WMAuthenticationRedirectionHandler {
void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, 
                             WMAuthentication authentication) throws IOException, ServletException;
}

### Handler declaration

Declare the following bean in the **project-user-spring.xml**.

<bean id="wmAuthenticationSuccessRedirectionHandler" 
      class="<package\_name>.MyAuthenticationRedirectionHandler"/>

Custom Authentication Handler

- [i. Custom Post-Authentication Success Handler](#successhandler)
    - [Creating a Package Structure](#creating-package-structure)
    - [Interface to implement](#interface-implement)
    - [Custom Handler Declaration](#custom-handler-declaration)
- [ii. WM Authentication Class](#authclass)
    - [Adding Custom Attributes](#add-custom-attribute)
        - [Attribute Class](#attribute-class)
        - [Attribute Scope](#attribute-scope)
        - [Attaching to the Logged-in User](#attaching-loggedin-user)
- [iii. Redirection Handler](#redirection)
    - [Interface to Implement](#interface-implement)
    - [Handler declaration](#handler-declaration)
