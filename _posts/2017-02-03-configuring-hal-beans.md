---
layout: post
title: How to register HAL renderer for all domain classes 
post_author: Martin Grześlowski
category: grails
tags: ['grails', 'HAL'] 
---
Thanks to this little code snippet below you can register HAL renderers for all your domain classes:

```groovy
grailsApplication.domainClasses*.clazz.each { Class domain ->
	"hal${domain.simpleName}Renderer"(HalJsonRenderer, domain)
	"hal${domain.simpleName}CollectionRenderer"(HalJsonCollectionRenderer, domain)
}
```

Read more: [http://docs.grails.org/latest/guide/webServices.html#hypermedia](http://docs.grails.org/latest/guide/webServices.html#hypermedia)

Run on Grails 3.2.4