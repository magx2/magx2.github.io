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

Change starting port to *9000*, because we will be starting both, client and server on the same machine. 

```oauth-provider\grails-app\conf\application.yml```:

```yml
server:
  port: 9000
```
