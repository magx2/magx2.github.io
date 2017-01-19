---
layout: post
title: How to integrate Spring (Boot) @Autowired with Spocks tests
post_author: Martin Grześlowski
category: spring
tags: ['spring', 'spring-boot', 'spock'] 
---
## Overview

In this tutorial I will show you how to integrate Springs ```@Autowired``` with Spocks tests. With this approach you will be able to annotate mock objects with ```@MockBean``` and get them automagicly injected into tested class.


## Add Spock to ```build.gradle```

	apply plugin: 'groovy'
  
	dependencies {
		compile('org.springframework.boot:spring-boot-starter')
		
		testCompile('org.springframework.boot:spring-boot-starter-test')
		testCompile(
				'junit:junit:4.12',
				'org.codehaus.groovy:groovy-all:2.4.4',
				'org.spockframework:spock-core:1.0-groovy-2.4')
		testCompile group: 'org.kubek2k', name: 'springockito', version: '1.0.9'
	}


## Create empty ```Specification```

	import spock.lang.Specification

	class MockSpecification extends Specification {

		def test() {
			expect:
			1 == 0
		}
	}
  
## Add required annotations
	
	import org.junit.Test
	import org.junit.runner.RunWith
	import org.mockito.InjectMocks
	import org.springframework.beans.factory.annotation.Autowired
	import org.springframework.boot.test.context.SpringBootTest
	import org.springframework.boot.test.mock.mockito.MockBean
	import org.springframework.test.context.ContextConfiguration
	import org.springframework.test.context.junit4.SpringRunner
	import spock.lang.Specification
	
	@RunWith(SpringRunner.class)
	@ContextConfiguration
	@SpringBootTest
	class MockSpecification extends Specification {
	
		@InjectMocks
		@Autowired
		MyObjectThatHasDependencies obj
	
		@MockBean
		Service1 service1
	
		@MockBean
		Service2 service2
	
		@Test
		def test() {
			expect:
			obj.service1
			obj.service2
		}
	}

