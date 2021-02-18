---
layout: post
title: "Time Travel"
description: "Getting time under control in Spring, one way"
category: [testing, java, time]
tags: [java, testing, time]
---

{::options parse_block_html="true" /}
## Overview
Time is often something worth raising to a domain-level concept is a recurring theme.

We've recently been doing some work where dates and times are important. We have cases where we want to verify
that the time of creation is reflected correctly.

In Java 8 there are several options. [Baeldung has a great article on those options](https://www.baeldung.com/java-override-system-time).
What follows is taking one of the options from that article all the way into a Spring-based project.

## Background

In a system where time is important to the business problem, and you want to write tests where the actual
time used is part of what you are checking, then getting time under control is a system requirement. It's
fair to say that system time is a domain-level concept.

Normally, Time is an external dependency because it is based on the clock in the computer
upon which the app is running. Tests attempt to get every dependency under the control of
the test. While not always possible, it is an ideal to strive for.

{% include aside/start id="focus-depend" title="Focus and Dependencies" %}
Generally, the more focused the test, the more likely controlling all dependencies is possible.
Getting all the dependencies controlled for a micro-test is generally much easier / more possible
than for an integration test, and an integration test tends to be easier than an acceptance test.
{% include aside/end %}

## Java 8
Java 8 added an entirely new date/time implementation based on [Joda Time](https://www.joda.org/joda-time/)
and written by its author. The new and old co-exist. But they are fairly disconnected.

What follows are the moving parts to time travel in a spring-based test. 

### How to get time
For a component that needs time, instead of using `Date`, use`Clock`.
For example, instead of:
```java
new Date()  // harder to control
```
We use this:
```java
Date.from(systemClock.instant())
// or
Date.from(Instant.now(systemClock))
```

### How to use the clock we define
Consider a Spring component that cares about time:
```java
@Component
public class HelloComponent {
    private final Clock systemClock;
    public HelloComponent(Clock systemClock) {
        this.systemClock = systemClock;
    }
    public String message() {
        return "Hello, it is now: " + Date.from(systemClock.instant());
    }
}
```

### Part of Application Configuration
For systemClock this to be injected, there needs to be a bean in the Spring configuration. The special sauce is a
combination of two annotations:
* `@Configuration` enhance the Spring context with some bean instances
* `@Bean` Define a Spring bean that can be autowired into a component

This can be on its own class or, in this simple example, placed on the`@SpringBootApplication` directly.
```java
@SpringBootApplication
@Configuration
public class TimeApplication {
	public static void main(String[] args) {
		SpringApplication.run(TimeApplication.class, args);
	}
	@Bean
	public Clock systemClock() {
		return Clock.systemDefaultZone();
	}
}
```

## Not this Now, That Now
What about testing for a date that is not "now" in the real world?

### Override the Clock
If you can get away with using a plain JUnit test, then you can construct
your own`Clock`instance, and manually call the constructor of the`HelloComponent`.
That's straightforward and a good option.

If you're trying to get it to work in a spring context, it requires another
moving part. We'll override the application-generated Clock with a test Clock.

### Create a Test Config
```java
public class TestConfig {
    @Bean
    @Primary
    public Clock systemClock() {
        return Clock.fixed(
                Instant.parse("2031-08-22T10:00:00Z"),
                ZoneOffset.UTC);
    }
}
```

{% include aside/start id="primary-override" title="Focused, Or Full Integration?" %}
Note, the use of`@Primay`on this`@Bean`method is to allow for this TestConfig to work on its own, or in a larger
initialization context. 

In the test that follows, the test limits what is in the Spring context for test execution by listing specific classes
that the test-writer knew were needed to get this test to run.

Alternatively, you can specify the application class and get a fully initialized spring context with all the 
project's components, controllers, services, etc. 

If you take this route, then in addition to using`@Primary`, such a test would also need the annotation:
```java
@TestPropertySource(properties="spring.main.allow-bean-definition-overriding=true")
```
{% include aside/end %}

### Oh yes, how about a test
```java
@SpringBootTest(classes = {HelloComponent.class, TestConfig.class})
class HelloComponentTest {
    @Autowired
    HelloComponent hello;
    @Test
    void messageControlled() {
        String message = hello.message();
        assertThat(message).contains("2031");
    }
}
```
If you look at the test config, the date is fixed to 2031-08-22, so while the assertion could be more specific, 
it demonstrates that we are overriding the real time with our desired time. 

{% include aside/start id="avoid_2031" title="Warning, do not run in 2031" %}
If you run this test in the year 2031, then the test passing proves nothing.
{% include aside/end %}

## Conclusion

* The most important thing to take away, in general, is that anything that we care about  
  in the application is worth getting under control.

* If your system cares about time, then getting time under control is a good idea because  
  it is part of the domain. That is, it is a business-level concept worthy of explicit  
  representation in the production code.

* This shows one path with Java 8 and Spring. While those details are of an immediate  
  nature typically, the previous two bullets are a bit longer-lived.
