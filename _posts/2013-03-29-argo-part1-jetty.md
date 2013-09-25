---
layout: post
category : opensource
tagline: "Supporting tagline"
tags : [Argo, github, opensource]
title: Argo part1: Jetty
---
{% include JB/setup %}

Argo([https://github.com/58code/Argo](https://github.com/58code/Argo "https://github.com/58code/Argo")) is a lightweight web framework opensouced by 58.com(the craigslist in china). 

### Background ###

Before I read the source code of this project, I have only the Java language background, but no experience in web container, Spring, Servlet, tomcat, jetty. So the first question that hit me is how does Argo works?


There is a example in the folder samples/hello-world, but i could find the Main function. From the document file Readme, running Argo needs only one command:

    maven jetty:run

I known maven is a project management tool like ant, but what is **jetty**? I don't know the relations between this command with Argo.

### Jetty ###

After googling, i found the useful pages about jetty in the following URL.

[http://wiki.eclipse.org/Jetty/Feature/Jetty_Maven_Plugin](http://wiki.eclipse.org/Jetty/Feature/Jetty_Maven_Plugin)

[http://wiki.eclipse.org/Jetty/Reference/Jetty_Architecture](http://wiki.eclipse.org/Jetty/Reference/Jetty_Architecture)

1) jetty is a web container like *tomcat*.
T
At first, I thought it may existing some configurations of jetty that would relate to Argo. But in the file samples/hellow-world/pom.xml, I could only found the following statements:

	<plugin>  
	    <!-- http://wiki.eclipse.org/Jetty/Feature/Jetty_Maven_Plugin -->  
	    <groupId>org.mortbay.jetty</groupId>  
	    <artifactId>jetty-maven-plugin</artifactId>  
	  
	    <configuration>  
	        <stopPort>9966</stopPort>  
	        <stopKey>foo</stopKey>  
	        <scanIntervalSeconds>0</scanIntervalSeconds>  
	        <connectors>  
	            <connector implementation="org.eclipse.jetty.server.nio.SelectChannelConnector">  
	                <port>80</port>  
	                <maxIdleTime>60000</maxIdleTime>  
	            </connector>  
	        </connectors>  
	        <webAppConfig>  
	            <contextPath>/</contextPath>  
	        </webAppConfig>  
	    </configuration>  
	</plugin>  

The only thing that may have connections with hellow-world project is webAppConfig. But it only contains a path *contextPath*.

2) When running *jetty:run*, jetty will deploy the web application based on:

- resources in ${basedir}/src/main/webapp
- classes in ${project.build.outputDirectory}
- web.xml in ${basedir}/src/main/webapp/WEB-INF/

I can find three html files in the directory src/main/webapp，no configurations. Also, there is no web.xml. So the only possible configuration is inside the source code( *HelloController.java*，*HomeController.java* ).


3) jetty has the following architecture:

![http://wiki.eclipse.org/images/8/88/JettyUML1.png](http://wiki.eclipse.org/images/8/88/JettyUML1.png)

The Connector receives Http connections. The Handle handles request from the Connector，and gives a response. Also, a ThreadPool is used.

4) The Connector and the Handler can be configured in:

- In code. See the examples in the Jetty 7 Latest Source XRef.
- With Jetty XML - dependency injection style XML format.
- With your dependency injection framework of choice: Spring or XBean.
- Using Jetty WebApp and Context Deployers.

In 1), we can see the connector implementation is org.eclipse.jetty.server.nio.SelectChannelConnector. But what i want is the part that relate with hellow-world.

No Main function to call jetty in code, no XML configuration files. Not configured in webapp. 

So, I'm in certain that Argo used the dependency injection technique. That is Argo use **dependency injection** to configure jetty.


### Dependency Injection ###
[http://stackoverflow.com/a/140655/732267](http://stackoverflow.com/a/140655/732267)

[http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html](http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html)

[http://helloj2ee.googlecode.com/files/DependencyInjection.pdf](http://helloj2ee.googlecode.com/files/DependencyInjection.pdf) //strongly recommended 

An example in DependencyInjection.pdf:

    class MovieLister{
      private final MovieFinder finder;
      public MovieLister() {
        finder = new ColonDelimitedMovieFinder(“movies.txt”);
      }

      public Movie[] moviesDirectedBy(String arg) {  
        List allMovies = finder.findAll();  //<---

        for (Iterator it = allMovies.iterator(); it.hasNext();) {  
          Movie movie = (Movie) it.next();  
          if (!movie.getDirector().equals(arg)) it.remove();  
        }
        return (Movie[]) allMovies.toArray(new Movie[allMovies.size()]);  
    }
    

The MovieLister relies on MoiveFinder. 

By using dependency injection, you can eliminate this dependency, and inject a proper MovieFinder implementation when using MovieLister

    public class MovieLister {
     private final MovieFinder finder;
     public MovieLister(MovieFinder finder) {
       this.finder = finder;
     }
     public List<Movie> moviesDirectedBy(String arg) {
       List<Movie> movies = finder.findAll();
       for ( Iterator<Movie> it = movies.iterator(); it.hasNext(); ) {
         Movie movie = it.next();
         if ( !movie.getDirector().equals(arg) ) {
           it.remove();
         }
       }
       return movies;
     }
    }

You may have used this technique in OO, without knowing it also a implement of dependency injection. 

The dependency injection can be implemented by:

- Constructor Injection (like the MovieLister above)
- Setter Injection
- Interface Injection

Let's come back to Argo.

### Dependency Injection in Argo ###

The interface that HelloController implements is ArgoController, which is defined as:

	/**
	 * interface that a Controller class should implemented
	 */
	public interface ArgoController {
	    /**
	     * Called when instantiated by injector
	     */  
	    void init();
	}

We can speculate that Argo use interface injection, and ArgoController is strongly related to Dependency Injection.

### Web Container: jetty & tomcat ###

We known jetty gets the configuration by interface injection, and then can starts the Hello-world web application.

Another tool that could be used to run Hello-world web application is tomcat. In fact, jetty and tomcat are web contains that supports Servlets Filters.

[http://stackoverflow.com/questions/1893253/tomcat-web-server-or-web-container](http://stackoverflow.com/questions/1893253/tomcat-web-server-or-web-container)
[http://stackoverflow.com/questions/12689910/difference-between-web-server-web-container-and-application-server](http://stackoverflow.com/questions/12689910/difference-between-web-server-web-container-and-application-server)

tomcat configuration is similar to jetty: (samples/hellow-world/pom.xml)

	<plugin>  
	    <groupId>org.apache.tomcat.maven</groupId>
	    <artifactId>tomcat7-maven-plugin</artifactId>
	    <version>2.0</version>
	    <configuration>
	        <path>/</path> <!--/argo -->
	        <port>80</port>
	    </configuration>
	</plugin>

PS: It must clarified that only servlet 3.0 supports annotation based injection. So, if you used tomcat 6, you still needs to configure the web.xml:

    <filter>
      <filter-name>mvcfilter</filter-name>
      <filter-class>com.bj58.argo.servlet.ArgoFilter</filter-class>
    </filter>

### Conclusion ###

Argo use interface injection to achieve the Dependency Injection. The interface related is com.bj58.argo.ArgoController.

Argo is configured by Dependency Injection. And jetty/tomcat used these configurations to bootstrap a web application.