---
layout: post
tagline: "Supporting tagline"
tags : [Argo, github, opensource]
title : Argo part 4 - Servlet Filter
---
{% include JB/setup %}

The root of the call hierarchy of Argo is ArgoFilter. What is a Filter?

### Filter ###

Filter is usually used as a small segment of functionality that appeared before or after the main functions in web application. For example, a filter to check if user has access right to a certain URL.

A Filter guide：
[http://www.avajava.com/tutorials/lessons/what-is-a-filter-and-how-do-i-use-it.html](http://www.avajava.com/tutorials/lessons/what-is-a-filter-and-how-do-i-use-it.html)

Filter must implements the javax.servlet.Filter interface. And override the method doFilter to do the filtering.

An example from the guide: A filer to display request params in url.

{% highlight java %}
public class MyFilter implements Filter {  
    FilterConfig filterConfig = null;  

    public void init(FilterConfig filterConfig) throws ServletException {  
        this.filterConfig = filterConfig;  
    }  

    …  

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)  
            throws IOException, ServletException {  
        servletResponse.setContentType("text/html");  

        PrintWriter out = servletResponse.getWriter();  
        out.println("my-param (InitParameter): " + filterConfig.getInitParameter("my-param"));  
        out.println("<br/><br/>Parameters:<br/>");  
        Enumeration<String> parameterNames = servletRequest.getParameterNames();  
        if (parameterNames.hasMoreElements()) {  
            while (parameterNames.hasMoreElements()) {  
                String name = parameterNames.nextElement();  
                String value = servletRequest.getParameter(name);  
                out.println("name:" + name + ", value: " + value + "<br/>");  
            }  
        } else {  
            out.println("---None---<br/>");  
        }  

        out.println("<br/>Start Regular Content:<br/><hr/>");  
        filterChain.doFilter(servletRequest, servletResponse);  
        out.println("<br/><hr/>End Regular Content:<br/>");  
    }  

}  
{% endhighlight %}

It displays an extra params in the front of the regular content.

### Filter Configuration ###

A Filter can be configured

- in web.xml
- by @WebFilter annotation

For example, you can configure Argo in web.xml

{% highlight xml %}
<filter>
  <filter-name>mvcfilter</filter-name>
  <filter-class>com.bj58.argo.servlet.ArgoFilter</filter-class>
</filter>
{% endhighlight %}

### @WebFilter ###


*@WebFilter* examples

[http://www.codejava.net/java-ee/servlet/webfilter-annotation-examples](http://www.codejava.net/java-ee/servlet/webfilter-annotation-examples)

{% highlight java %}
@WebFilter(  
    urlPatterns = {"/*"},
    attribute2=value2,  
    ...  
)  
public class TheFilter implements javax.servlet.Filter {  
    // implements Filter's methods  
}  
{% endhighlight %}

The urlPatterns is a must attribute in @WebFilter. And the class annotated by @WebFilter must implements the javax.servlet.Filter interface.


### Filter in Argo ###

{% highlight java %}
/**
* Schedule by a Filter
*/  
@WebFilter(urlPatterns = {"/*"},  
        dispatcherTypes = {DispatcherType.REQUEST},  
        initParams = {@WebInitParam(name = "encoding", value = "UTF-8")}  
)  
public class ArgoFilter implements Filter {  
    private ArgoDispatcher dispatcher;  

    @Override  
    public void init(FilterConfig filterConfig) throws ServletException {  
        ServletContext servletContext = filterConfig.getServletContext();  

        try {  
            dispatcher = ArgoDispatcherFactory.create(servletContext);  
            dispatcher.init();  
        } catch (Exception e) {  
            servletContext.log("failed to argo initialize, system exit!!!", e);  
            System.exit(1);  
        }  
    }  

    @Override  
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
        HttpServletRequest httpReq = (HttpServletRequest) request;  
        HttpServletResponse httpResp = (HttpServletResponse) response;  

        dispatcher.service(httpReq, httpResp);  
    }  

    @Override  
    public void destroy() {  
        dispatcher.destroy();  
    }  
}  
{% endhighlight %}


- *@WebFilter(urlPatterns = {"/*"}* means ArgoFilter filtering on any the request URLS.

- In init(), a **ArgoDispatcher** instance is instantiated，and all the request is route to the dispatcher.
