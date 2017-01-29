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
 
## Configuring OAuth Client

### Configuring filter chain

Create file ```oauth-client\grails-app\conf\application.groovy``` with content:

```groovy
grails.plugin.springsecurity.filterChain.chainMap = [
        //Stateless chain
        [
                pattern: '/api/**',
                filters: 'JOINED_FILTERS,-anonymousAuthenticationFilter,-exceptionTranslationFilter,-authenticationProcessingFilter,-securityContextPersistenceFilter,-rememberMeAuthenticationFilter'
        ],

        //Traditional, stateful chain
        [
                pattern: '/**',
                filters: 'JOINED_FILTERS,-restTokenValidationFilter,-restExceptionTranslationFilter'
        ]
]
```

### Create domain classes

In ```oauth-client\grails-app\domain\ouath\client``` create classes:

```groovy
package oauth.client

import groovy.transform.EqualsAndHashCode
import groovy.transform.ToString

@EqualsAndHashCode(includes = 'username')
@ToString(includes = 'username', includeNames = true, includePackage = false)
class User implements Serializable {
	@SuppressWarnings("GroovyUnusedDeclaration")
	private static final long serialVersionUID = 1

	String username
	Set<Role> roles = []

	static constraints = {
		username unique: true, nullable: false, blank: false
	}
}
```

```groovy
package oauth.client

import groovy.transform.EqualsAndHashCode
import groovy.transform.ToString
import org.springframework.security.core.GrantedAuthority

@EqualsAndHashCode(includes = 'authority')
@ToString(includes = 'authority', includeNames = true, includePackage = false)
class Role implements Serializable, GrantedAuthority {
	@SuppressWarnings("GroovyUnusedDeclaration")
	private static final long serialVersionUID = 1

	String authority
	
	static constraints = {
		authority unique: true, nullable: false, blank: false
	}
}
```

### GrailsOAuthClient

In ```oauth-client\src\main\groovy\oauth\client\GrailsOAuthClient.groovy``` create file (at this point some code won't compile. because of missing classes):

```groovy
package oauth.client

import groovy.transform.CompileStatic
import org.pac4j.core.context.WebContext
import org.pac4j.oauth.client.BaseOAuth20Client
import org.scribe.model.Token

@CompileStatic
class GrailsOAuthClient extends BaseOAuth20Client<GrailsOAuthProfile> {
	@SuppressWarnings("GroovyUnusedDeclaration")
	private static final long serialVersionUID = 1L

	GrailsOAuthClient(String key, String secret) {
		setKey(key)
		setSecret(secret)
	}

	@Override
	void internalInit(final WebContext context) {
		super.internalInit(context)
		this.service = new GrailsOAuthServiceImpl(new GrailsOAuthApi(), new GrailsOAuthConfig(getKey(), getSecret()))
	}

	@Override
	protected boolean hasBeenCancelled(WebContext context) {
		false
	}

	@Override
	protected String getProfileUrl(Token accessToken) {
		"http://localhost:9000/api/me"
	}

	@Override
	protected GrailsOAuthProfile extractUserProfile(String body) {
		new GrailsOAuthProfile(body)
	}

	@Override
	protected GrailsOAuthClient newClient() {
		new GrailsOAuthClient(getKey(), getSecret())
	}
}
```

### GrailsOAuthServiceImpl

In ```oauth-client\src\main\groovy\oauth\client\GrailsOAuthServiceImpl.groovy``` create file:

```groovy
package oauth.client

import org.scribe.builder.api.DefaultApi20
import org.scribe.model.OAuthConfig
import org.scribe.model.OAuthConstants
import org.scribe.model.OAuthRequest
import org.scribe.model.Response
import org.scribe.model.Token
import org.scribe.model.Verifier
import org.scribe.oauth.OAuth20ServiceImpl

class GrailsOAuthServiceImpl extends OAuth20ServiceImpl {
	private final DefaultApi20 api
	private final GrailsOAuthConfig config

	GrailsOAuthServiceImpl(DefaultApi20 api, GrailsOAuthConfig config) {
		super(api, config)
		this.api = api
		this.config = config
	}

	Token getAccessToken(Token requestToken, Verifier verifier) {
		OAuthRequest request = new OAuthRequest(api.getAccessTokenVerb(), api.getAccessTokenEndpoint())
		request.addQuerystringParameter(OAuthConstants.CLIENT_ID, config.getApiKey())
		request.addQuerystringParameter(OAuthConstants.CLIENT_SECRET, config.getApiSecret())
		request.addQuerystringParameter(OAuthConstants.CODE, verifier.getValue())
		request.addQuerystringParameter(OAuthConstants.REDIRECT_URI, config.getCallback())
		request.addQuerystringParameter("grant_type", "authorization_code") // This line is not added to request params in OAuth20ServiceImpl
		if (config.hasScope()) request.addQuerystringParameter(OAuthConstants.SCOPE, config.getScope())
		Response response = request.send()
		return api.getAccessTokenExtractor().extract(response.getBody())
	}
}
```

### GrailsOAuthApi

In ```oauth-client\src\main\groovy\oauth\client\GrailsOAuthApi.groovy``` create file:

```groovy
package oauth.client

import groovy.transform.CompileStatic
import org.scribe.builder.api.DefaultApi20
import org.scribe.extractors.AccessTokenExtractor
import org.scribe.model.OAuthConfig
import org.scribe.model.Verb

@CompileStatic
class GrailsOAuthApi extends DefaultApi20 {
	@Override
	String getAccessTokenEndpoint() {
		"http://localhost:9000/oauth/token"
	}

	@Override
	String getAuthorizationUrl(OAuthConfig config) {
		"http://localhost:9000/oauth/authorize?" +
				"response_type=code&" +
				"client_id=${config.apiKey}&" +
				"client_secret=$config.apiSecret&" +
				"scope=read"
	}

	@Override
	Verb getAccessTokenVerb() {
		Verb.POST
	}

	@Override
	AccessTokenExtractor getAccessTokenExtractor() {
		new JsonAccessTokenExtractor()
	}
}
```

### GrailsOAuthConfig

In ```oauth-client\src\main\groovy\oauth\client\GrailsOAuthConfig.groovy``` create:

```groovy
package oauth.client

import org.scribe.model.OAuthConfig
import org.scribe.model.SignatureType

class GrailsOAuthConfig extends OAuthConfig {
	GrailsOAuthConfig(String key, String secret) {
		super(key, secret, "http://localhost:8080/oauth/callback/grailsoauth", null, "read", null)
	}
}
```


### GrailsOAuthProfile

In ```oauth-client\src\main\groovy\oauth\client\GrailsOAuthProfile.groovy``` create:

```groovy
package oauth.client

import groovy.json.JsonSlurper
import groovy.transform.CompileStatic
import org.pac4j.oauth.profile.OAuth20Profile

@CompileStatic
class GrailsOAuthProfile extends OAuth20Profile {
	@SuppressWarnings("GroovyUnusedDeclaration")
	private static final long serialVersionUID = 1L

	GrailsOAuthProfile() {
	}

	GrailsOAuthProfile(String requestBody) {
		final json = new JsonSlurper().parseText(requestBody)
		setId(json['id'] as String)
		addAttribute("username", json['username'])
		addAttribute("email", json['username'])
		addAttribute("enabled", json['enabled'])
		addAttribute("accountExpired", Boolean.getBoolean(json['accountExpired'] as String))
		addAttribute("accountLocked", Boolean.getBoolean(json['accountLocked'] as String))
		addAttribute("passwordExpired", Boolean.getBoolean(json['passwordExpired'] as String))
		final roles = json['roles'] as Collection<String>
		roles.each { role -> addRole(role) }
	}
}
```

### JsonAccessTokenExtractor

In ```oauth-client\src\main\groovy\oauth\client\JsonAccessTokenExtractor.groovy``` create:

```groovy
package oauth.client
   
import groovy.json.JsonSlurper
import groovy.transform.CompileStatic
import org.scribe.exceptions.OAuthException
import org.scribe.extractors.AccessTokenExtractor
import org.scribe.model.Token

@CompileStatic
class JsonAccessTokenExtractor implements AccessTokenExtractor {
	@Override
	Token extract(String response) throws OAuthException {
		final json = new JsonSlurper().parseText(response)
		final accessToken = json["access_token"] as String
		if (accessToken) {
			return new Token(accessToken, response)
		} else {
			throw new OAuthException("Cannot extract an access token. Response was: " + response)
		}
	}
}
```