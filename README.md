# Getting Started Accessing Twitter Data

What you'll build
-----------------

This guide will take you through creating a simple web application that accesses user data from Twitter, including the user's full name and a list of other Twitter users that they who they follow.

What you'll need
----------------

1. About 15 minutes
1. An application ID and secret obtained from [registering an application with Twitter][register-twitter-app].
 - A favorite text editor or IDE
 - [JDK 6][jdk] or later
 - [Maven 3.0][mvn] or later

[jdk]: http://www.oracle.com/technetwork/java/javase/downloads/index.html
[mvn]: http://maven.apache.org/download.cgi

How to complete this guide
--------------------------

Like all Spring's [Getting Started guides](/getting-started), you can start from scratch and complete each step, or you can bypass basic setup steps that are already familiar to you. Either way, you end up with working code.

To **start from scratch**, move on to [Set up the project](#scratch).

To **skip the basics**, do the following:

 - [Download][zip] and unzip the source repository for this guide, or clone it using [git](/understanding/git):
`git clone https://github.com/springframework-meta/{@project-name}.git`
 - cd into `{@project-name}/initial`
 - Jump ahead to [Create a resource representation class](#initial).

**When you're finished**, you can check your results against the code in `{@project-name}/complete`.

<a name="scratch"></a>
Set up the project
------------------
First you set up a basic build script. You can use any build system you like when building apps with Spring, but the code you need to work with [Maven](https://maven.apache.org) and [Gradle](http://gradle.org) is included here. If you're not familiar with either, refer to [Getting Started with Maven](../gs-maven/README.md) or [Getting Started with Gradle](../gs-gradle/README.md).

### Create the directory structure

In a project directory of your choosing, create the following subdirectory structure; for example, with `mkdir -p src/main/java/hello` on *nix systems:

    └── src
        └── main
            └── java
                └── hello

### Create a Maven POM

`pom.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>org.springframework</groupId>
	<artifactId>gs-accessing-twitter-initial</artifactId>
	<version>0.1.0</version>

	<parent>
		<groupId>org.springframework.bootstrap</groupId>
		<artifactId>spring-bootstrap-starters</artifactId>
		<version>0.5.0.BUILD-SNAPSHOT</version>
	</parent>

	<dependencies>
		<dependency>
			<groupId>org.springframework.bootstrap</groupId>
			<artifactId>spring-bootstrap-web-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.social</groupId>
			<artifactId>spring-social-twitter</artifactId>
			<version>1.1.0.BUILD-SNAPSHOT</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-crypto</artifactId>
			<version>3.1.4.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.thymeleaf</groupId>
			<artifactId>thymeleaf-spring3</artifactId>
			<version>2.0.16</version>
		</dependency>		
	</dependencies>

	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>http://repo.springsource.org/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>http://repo.springsource.org/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```

TODO: mention that we're using Spring Bootstrap's [_starter POMs_](../gs-bootstrap-starter) here.

> Note to experienced Maven users who don't use an external parent project: You can take it out later, it's just there to reduce the amount of code you have to write to get started.


<a name="initial"></a>
Enable Twitter
----------------
Before you can fetch a user's data from Twitter, there are a few things that you'll need to set up in the Spring configuration. Here's a configuration class that contains everything you'll need to enable Twitter in your application:

`src/main/java/hello/TwitterConfig.java`
```java
package hello;

import org.springframework.context.annotation.Bean;
import org.springframework.social.UserIdSource;
import org.springframework.social.config.annotation.EnableInMemoryConnectionRepository;
import org.springframework.social.connect.ConnectionFactoryLocator;
import org.springframework.social.connect.ConnectionRepository;
import org.springframework.social.connect.web.ConnectController;
import org.springframework.social.twitter.config.annotation.EnableTwitter;

@EnableTwitter(appId="someAppId", appSecret="shhhhhh!!!")
@EnableInMemoryConnectionRepository
public class TwitterConfig {

    @Bean
    public UserIdSource userIdSource() {
        return new UserIdSource() {         
            @Override
            public String getUserId() {
                return "testuser";
            }
        };
    }

    @Bean
    public ConnectController connectController(ConnectionFactoryLocator connectionFactoryLocator, ConnectionRepository connectionRepository) {
        return new ConnectController(connectionFactoryLocator, connectionRepository);
    }

}
```

Since the application will be accessing Twitter data, `TwitterConfig` is annotated with [`@EnableTwitter`][@EnableTwitter]. Notice that, as shown here, the `appId` and `appSecret` attributes have been given fake values. For the code to work, you'll need to [obtain a real application ID and secret][register-twitter-app] and substitute these fake values for the real values given to you by Twitter.

Notice that `TwitterConfig` is also annotated with [`@EnableInMemoryConnectionRepository`][@EnableInMemoryConnectionRepository]. After a user authorizes your application to access their Twitter data, Spring Social will create a connection. That connection will need to be saved in a connection repository for long-term use.

For the purposes of this guide's sample application, an in-memory connection repository is sufficient. Although an in-memory connection repository is fine for testing and small sample applications, you'll want to select a more persistent
option for real applications. You can use [`@EnableJdbcConnectionRepository`][@EnableJdbcConnectionRepository] to persist connections to a relational database.

Within the `TwitterConfig`'s body, there are two beans declared: a `ConnectController` bean and a `UserIdSource` bean.

Obtaining user authorization from Twitter involves a "dance" of redirects between the application and Twitter. This "dance" is formally known as [OAuth][oauth]'s _Resource Owner Authorization_. Don't worry if you don't know much about OAuth. Spring Social's [`ConnectController`][ConnectController] will take care of the OAuth dance for you.

Notice that `ConnectController` is created by injecting a [`ConnectionFactoryLocator`][ConnectionFactoryLocator] and a [`ConnectionRepository`][ConnectionRepository] via the constructor. You won't need to explicitly declare these beans, however. The `@EnableTwitter` annotation will make sure that a `ConnectionFactoryLocator` bean is created and the `@EnableInMemoryConnectionRepository` annotation will create an in-memory implementation of `ConnectionRepository`.

Connections represent a 3-way agreement between a user, an application, and an API provider such as Twitter. Although Twitter and the application itself are readily identifiable, you'll need a way to identify the current user. That's what the `UserIdSource` bean is for. 

Here, the `userIdSource` bean is defined by an inner-class that always returns "testuser" as the user ID. Thus there is only one user of our sample application. In a real application, you'll probably want to create an implementation of `UserIdSource` that determines the user ID from the currently authenticated user (perhaps by consulting with an [`Authentication`][Authentication] obtained from Spring Security's [`SecurityContext`][SecurityContext]).

### Create Connection Status Views

Although much of what `ConnectController` does involves redirecting to Twitter and handling a redirect from Twitter, it also shows connection status when a GET request to /connect is made. It will defer to a view whose name is connect/{provider ID}Connect when no existing connection is available and to connect/{providerId}Connected when a connection exists for the provider. In our case, {provider ID} is "Twitter".

`ConnectController` does not define its own connection views, so you'll need to create them ourselves. First, here's a Thymeleaf view to be shown when no connection to Twitter exists:

`src/main/resources/templates/connect/twitterConnect.html`
```html
<html>
	<head>
		<title>Hello Twitter</title>
	</head>
	<body>
		<h3>Connect to Twitter</h3>
		
		<form action="/connect/twitter" method="POST">
			<div class="formInfo">
				<p>You aren't connected to Twitter yet. Click the button to connect Spring Social Showcase with your Twitter account.</p>
			</div>
			<p><button type="submit">Connect to Twitter</button></p>
		</form>
	</body>
</html>
```

The form on this view will POST to /connect/twitter, which is handled by `ConnectController` and will kick off the OAuth authorization code flow.

Here's the view to be displayed when a connection exists:

`src/main/resources/templates/connect/twitterConnected.html`
```html
<html>
	<head>
		<title>Hello Twitter</title>
	</head>
	<body>
		<h3>Connected to Twitter</h3>
		
		<p>
			You are now connected to your Twitter account.
			Click <a href="/">here</a> to see your Twitter friends.
		</p>		
	</body>
</html>
```


Fetch Twitter data
---------------------
With Twitter configured in your application, you now can write a Spring MVC controller that fetches data for the user who authorized the application and presents it in the browser. `HelloController` is just such a controller:

`src/main/java/hello/HelloController.java`
```java
package hello;

import javax.inject.Inject;

import org.springframework.social.twitter.api.CursoredList;
import org.springframework.social.twitter.api.Twitter;
import org.springframework.social.twitter.api.TwitterProfile;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
@RequestMapping("/")
public class HelloController {
	
    private Twitter twitter;
    
    @Inject
    public HelloController(Twitter twitter) {
        this.twitter = twitter;		
    }
    
    @RequestMapping(method=RequestMethod.GET)
    public String helloTwitter(Model model) {
        if (!twitter.isAuthorized()) {
            return "redirect:/connect/twitter";
        }
        
        model.addAttribute(twitter.userOperations().getUserProfile());
        CursoredList<TwitterProfile> friends = twitter.friendOperations().getFriends();
        model.addAttribute("friends", friends);
        return "hello";
    }
	
}
```

`HelloController` is created by injecting a `Twitter` object into its constructor. The `Twitter` object is a reference to Spring Social's Twitter API binding.

The `helloTwitter()` method is annotated with `@RequestMapping` to indicate that it should handle GET requests for the root path (/). The first thing it does is check to see if the user has authorized the application to access their Twitter data. If not, then the user is redirected to `ConnectController` where they may kick off the authorization process.

If the user has authorized the application to access their Twitter data, then the application will be able to fetch almost any data pertaining to the authorizing user. For the purposes of this guide's application, it will only fetch the user's profile as well as a list of profiles belonging to the user's friends (the Twitter users that the user follows, not those that follow the user). Both are placed into the model to be displayed by the view identified as "hello".

Speaking of the "hello" view, here it is as a Thymeleaf template:

`src/main/resources/templates/hello.html`
```html
<html>
	<head>
		<title>Hello Twitter</title>
	</head>
	<body>
		<h3>Hello, <span th:text="${twitterProfile.name}">Some User</span>!</h3>
		
		<h4>These are your friends:</h4>
		
		<ul>
			<li th:each="friend:${friends}" th:text="${friend.name}">Friend</li>
		</ul>
	</body>
</html>
```

This template simply displays a greeting to the user and a list of the user's friends.
Note that even though the full user profiles were fetched, only the names from those profiles are used in this template.

Make the application executable
-------------------------------

Although it is possible to package this service as a traditional _web application archive_ or [WAR][u-war] file for deployment to an external application server, the simpler approach demonstrated below creates a _standalone application_. You package everything in a single, executable JAR file, driven by a good old Java `main()` method. And along the way, you use Spring's support for embedding the [Tomcat][u-tomcat] servlet container as the HTTP runtime, instead of deploying to an external instance.

### Create a main class

`src/main/java/hello/Application.java`
```java
package hello;

import org.springframework.bootstrap.SpringApplication;
import org.springframework.bootstrap.context.annotation.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;

@Configuration
@EnableAutoConfiguration
@EnableWebMvc
@Import(TwitterConfig.class)
@ComponentScan
public class Application {

    /*
     * SPRING BOOTSTRAP MAIN
     */
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

} 
```

The `main()` method defers to the [`SpringApplication`][] helper class, providing `Application.class` as an argument to its `run()` method. This tells Spring to read the annotation metadata from `Application` and to manage it as a component in the _[Spring application context][u-application-context]_.

The `@ComponentScan` annotation tells Spring to search recursively through the `hello` package and its children for classes marked directly or indirectly with Spring's [`@Component`][] annotation. This directive ensures that Spring finds and registers the `GreetingController`, because it is marked with `@Controller`, which in turn is a kind of `@Component` annotation.

The `@Import` annotation tells Spring to import additional Java configuration. Here it is asking Spring to import the `TwitterConfig` class where you enabled Twitter in your application.

The [`@EnableAutoConfiguration`][] annotation switches on reasonable default behaviors based on the content of your classpath. For example, because the application depends on the embeddable version of Tomcat (tomcat-embed-core.jar), a Tomcat server is set up and configured with reasonable defaults on your behalf. And because the application also depends on Spring MVC (spring-webmvc.jar), a Spring MVC [`DispatcherServlet`][] is configured and registered for you — no `web.xml` necessary! Auto-configuration is a powerful, flexible mechanism. See the [API documentation][`@EnableAutoConfiguration`] for further details.

### Build an executable JAR

Now that your `Application` class is ready, you simply instruct the build system to create a single, executable jar containing everything. This makes it easy to ship, version, and deploy the service as an application throughout the development lifecycle, across different environments, and so forth.

Add the following configuration to your existing Maven POM:

`pom.xml`
```xml
    <properties>
        <start-class>hello.Application</start-class>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

The `start-class` property tells Maven to create a `META-INF/MANIFEST.MF` file with a `Main-Class: hello.Application` entry. This entry enables you to run the jar with `java -jar`.

The [Maven Shade plugin][maven-shade-plugin] extracts classes from all jars on the classpath and builds a single "über-jar", which makes it more convenient to execute and transport your service.

Now run the following to produce a single executable JAR file containing all necessary dependency classes and resources:

    mvn package

[maven-shade-plugin]: https://maven.apache.org/plugins/maven-shade-plugin


Running the Service
-------------------------------------

Now you can run it from the jar as well, and distribute that as an executable artifact:
```
$ java -jar target/gs-accessing-twitter-0.1.0.jar

... app starts up ...
```

Once the application starts up, you can point your web browser to http://localhost:8080. Since no connection has been established yet, you should see this screen prompting you to connect with Twitter:

![No connection to Twitter exists yet.](images/connect.png)
 
When you click the "Connect to Twitter" button, the browser will be redircted to Twitter for authorization:

![Twitter needs your permission to allow the application to access your data.](images/twauth.png)

At this point, Twitter is asking if you'd like to allow the sample application to read Tweets from your profile and see who you follow. If you agree, it will also be able to read your profile details. Click "Authorize app" to grant permission.

Once permission has been granted, Twitter will redirect the browser back to the application and a connection will be created and stored in the connection repository. You should see this page indicating that a connection was successful:

![A connection with Twitter has been created.](images/connected.png)

If you click on the link on the connection status page, you will be taken to the home page. This time, now that a connection has been created, you'll be shown your name on Twitter as well as a list of your friends:

![Guess noone told you life was gonna be this way.](images/friends.png)


Congratulations! You have just developed a simple web application that uses Spring Social to connect a user with Twitter and to retrieve some data from the user's Twitter profile.

Summary
-------
Congrats! You've just developed a simple web application that obtains user authorization to fetch data from Twitter. 

[zip]: https://github.com/springframework-meta/gs-accessing-twitter/archive/master.zip
[u-war]: /understanding/war
[u-tomcat]: /understanding/tomcat
[u-application-context]: /understanding/application-context
[`SpringApplication`]: http://static.springsource.org/spring-bootstrap/docs/0.5.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/bootstrap/SpringApplication.html
[`@Component`]: http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/stereotype/Component.html
[`@EnableAutoConfiguration`]: http://static.springsource.org/spring-bootstrap/docs/0.5.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/bootstrap/context/annotation/SpringApplication.html
[`DispatcherServlet`]: http://static.springsource.org/spring/docs/current/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html
[register-twitter-app]: /gs-register-twitter-app/README.md
[@EnableTwitter]: http://static.springsource.org/spring-social-twitter/docs/1.1.x/api/org/springframework/social/twitter/config/annotation/EnableTwitter.html
[@EnableInMemoryConnectionRepository]: TODO
[@EnableJdbcConnectionRepository]: http://static.springsource.org/spring-social/docs/1.1.x/api/org/springframework/social/config/annotation/EnableJdbcConnectionRepository.html
[oauth]: /understanding-oauth
[ConnectController]: http://static.springsource.org/spring-social/docs/1.1.x/api/org/springframework/social/connect/web/ConnectController.html
[ConnectionFactoryLocator]: http://static.springsource.org/spring-social/docs/1.1.x/api/org/springframework/social/connect/ConnectionFactoryLocator.html
[ConnectionRepository]: http://static.springsource.org/spring-social/docs/1.1.x/api/org/springframework/social/connect/ConnectionRepository.html
[Authentication]: http://static.springsource.org/spring-security/site/docs/3.2.x/apidocs/org/springframework/security/core/Authentication.html
[SecurityContext]: http://static.springsource.org/spring-security/site/docs/3.2.x/apidocs/org/springframework/security/core/context/SecurityContext.html

