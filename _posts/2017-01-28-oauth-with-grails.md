---
layout: post
title: title of the post, don't use markdown
post_author: Martin Grześlowski
category: OAuth
tags: ['OAuth', 'Grails'] 
published: false 
---

short overview TODO

## Versions

| Software | Version |
|----------|---------|
| Gradle   | v3.0    |
| Grails   | v3.2.4  |
| Spring Security OAuth Provider | org.grails.plugins:spring-security-oauth2-provider:3.0.0-RC2 |
| Spring Security REST | org.grails.plugins:spring-security-rest:2.0.0.M2 |


## Create project

```bash
mkdir oauth
cd oauth
grails create-app oauth-provider --profile rest-api
grails create-app oauth-client --profile angular
touch build.gradle
touch settings.gradle
```

```build.gradle``` is empty. There is no need to put code there. This file is only to create main entry point for gradle.

```settings.gradle``` content:

```groovy
rootProject.name = 'OAuth'

include ":oauth-provider"
include ":oauth-client"
```

### OAuth provider configuration

Change starting port to **9000**, because we will be starting both, client and server on the same machine. 

```oauth-provider\grails-app\conf\application.yml```:

```yml
server:
  port: 9000
```

### Gradle dependencies

In ```oauth-provider/build.gradle``` add:

```groovy
repositories {
    maven { url "https://dl.bintray.com/bluesliverx/grails-plugins" }
}

dependencies {
	compile 'org.grails.plugins:spring-security-oauth2-provider:3.0.0-RC2'
}
```

In ```oauth-client/build.gradle``` add:

```groovy
dependencies {
    compile "org.grails.plugins:spring-security-rest:2.0.0.M2"
}
```

## Configuring OAuth Provider

### Configure Spring Security Core

Run ```s2-quickstart```, more [here](http://grails-plugins.github.io/grails-spring-security-core/v3/index.html#s2-quickstart):

```bash
./oauth-provider/grailsw/s2-quickstart oauth.provider User Role
```

In ```oauth-provider\grails-app\domain``` you should have this classes:

 - Role
 - User
 - UserRole

Also you should have newly created file ```oauth-provider\grails-app\conf\application.groovy```, which contains something similar to this:

```groovy
// Added by the Spring Security Core plugin:
grails.plugin.springsecurity.userLookup.userDomainClassName = 'oauth.provider.User'
grails.plugin.springsecurity.userLookup.authorityJoinClassName = 'oauth.provider.UserRole'
grails.plugin.springsecurity.authority.className = 'oauth.provider.Role'
grails.plugin.springsecurity.controllerAnnotations.staticRules = [
		[pattern: '/',               access: ['permitAll']],
		[pattern: '/error',          access: ['permitAll']],
		[pattern: '/index',          access: ['permitAll']],
		[pattern: '/index.gsp',      access: ['permitAll']],
		[pattern: '/shutdown',       access: ['permitAll']],
		[pattern: '/assets/**',      access: ['permitAll']],
		[pattern: '/**/js/**',       access: ['permitAll']],
		[pattern: '/**/css/**',      access: ['permitAll']],
		[pattern: '/**/images/**',   access: ['permitAll']],
		[pattern: '/**/favicon.ico', access: ['permitAll']]
]

grails.plugin.springsecurity.filterChain.chainMap = [
		[pattern: '/assets/**',      filters: 'none'],
		[pattern: '/**/js/**',       filters: 'none'],
		[pattern: '/**/css/**',      filters: 'none'],
		[pattern: '/**/images/**',   filters: 'none'],
		[pattern: '/**/favicon.ico', filters: 'none'],
		[pattern: '/**',             filters: 'JOINED_FILTERS']
]
```

### Create OAuth domain classes

Run ```s2-init-oauth2-provider```, more [here](http://bluesliverx.github.io/grails-spring-security-oauth2-provider/v3/manual/ref/Scripts/s2-init-oauth2-provider.html):

```bash
./oauth-provider/grailsw s2-init-oauth2-provider oauth.provider Client AuthorizationCode AccessToken RefreshToken
```

In ```oauth-provider\grails-app\domain``` you should have this classes (besides those created by Spring Security Core):

 - AccessToken
 - AuthorizationCode
 - Client
 - RefreshToken
 
Also this template added some configuration in file  ```oauth-provider\grails-app\conf\application.groovy```:

```groovy
// Added by the Spring Security OAuth2 Provider plugin:
grails.plugin.springsecurity.oauthProvider.clientLookup.className = 'oauth.provider.Client'
grails.plugin.springsecurity.oauthProvider.authorizationCodeLookup.className = 'oauth.provider.AuthorizationCode'
grails.plugin.springsecurity.oauthProvider.accessTokenLookup.className = 'oauth.provider.AccessToken'
grails.plugin.springsecurity.oauthProvider.refreshTokenLookup.className = 'oauth.provider.RefreshToken'
```

### (Optional) Create mock User and Client for testing purposes
 
 In file ```oauth-provider\grails-app\init\oauth\provider\BootStrap.groovy``` add this:
 
 ```groovy
def init = { servletContext ->
    final user = new User(username: 'magx2', password: 'xxx').save(failOnError: true)
    final role = new Role("ROLE_USER").save(failOnError: true)
    new UserRole(user, role).save()

    new Client(
            clientId: 'client-key',
            clientSecret: 'client-secret',
            authorizedGrantTypes: ['authorization_code', 'refresh_token'],
            authorities: ['ROLE_CLIENT'],
            scopes: ['read', 'write'],
            redirectUris: ['http://localhost:8080/oauth/callback/oauthProvider']
    ).save(flush: true, failOnError: true)
}
 ```