---
layout: post
tagline: "Supporting tagline"
tags : [Argo, github, opensource]
title : Argo part 2 - Model View Controller
---
{% include JB/setup %}

### Hello World ###

Take the Hello world for example, the content of view/index.html is:

{% highlight html %}
<html>
<head>
    <title>Argo sample page</title>  
</head>  
<body>  
<h2>Samples</h2>  
<ul>  
    <li><a href="$__beat.servletContext.contextPath/hello/world">hello world for getting params from url</a></li>  
    <li><a href="$__beat.servletContext.contextPath/1.html">Static file demo</a></li>  
    <li><a href="$__beat.servletContext.contextPath/form.html">distinguish between queryString params and form params</a></li>  
    <li><a href="$__beat.servletContext.contextPath/upload-form.html">file uploading demo</a></li>  
</ul>  
</body>  
</html>  
{% endhighlight %}

When we click the first link, we are taken to the url http://localhost/hello/world, and the page says "hello world". 

What would Argo have done behind this? 

### Controller ###

Let's turn to HelloController.java :

{% highlight java %}
public class HelloController extends AbstractController {  
  
    @Path("hello/{name}")  
    public ActionResult hello(String name) {  
        return writer().write("hello %s", name);  
    }  
}  
{% endhighlight %}

**@Path** annotation relates the url with the function hello(). And writer().write() writes the greeting "hello world".

Is that how Argo works? Let's go to the next example.

### Post Example ###

Take the third url *form.html* for example, we filled the form, and post to the url post.html. 

According to the @Path annotation, the java function is:

{% highlight java %}
@Path("post.html")  
@POST  // only handles POST request
public ActionResult postForm() {
    BeatContext beat = beat();  
  
    ClientContext client = beat.getClient();  
  
    Preconditions.checkState(Strings.isNullOrEmpty(client.form("company")));  
    Preconditions.checkState(Strings.isNullOrEmpty(client.form("address")));  
  
    client.queryString("name");  
  
    Preconditions.checkState(Strings.isNullOrEmpty(client.queryString("name")));  
    Preconditions.checkState(Strings.isNullOrEmpty(client.queryString("phone")));  
    Preconditions.checkState(Strings.isNullOrEmpty(client.queryString("submit")));  
  
    beat.getModel()  
            .add("company", client.queryString("company"))  
            .add("address", client.queryString("address"))  
  
            .add("name", client.form("name"))  
            .add("phone", client.form("phone"))  
            .add("submit", client.form("submit"));  
  
    return view("post"); // resources/views/post.html, written by velocity  
}
{% endhighlight %}

It get params from url params and forms, and add the name, value pair of them to the Model of beat object. And then returns a view *post* for displaying.  

We followed to check post.html. The content is :

{% highlight html %}
<h3>queryString parameter</h3>  
<ul>  
    <li>company: $company</li>  
    <li>address: $address</li>  
</ul>  
<h3>form parameter</h3>  
<ul>  
    <li>name: $name</li>  
    <li>phone: $phone</li>  
    <li>submit: $submit</li>  
</ul>  
{% endhighlight %}

$company displays the value of the variable company. Exactly what is shown in the page  the url that jumps to.

### Velocity ###

From the document, we know that view is written by **Velocity**. 

A syntax manual can be found on [http://velocity.apache.org/engine/releases/velocity-1.5/user-guide.html](http://velocity.apache.org/engine/releases/velocity-1.5/user-guide.html)

### Conclusion ###

Now, we know that we can bind a url to a function, get parameters from url params or form, pass variables from back-end to the from-end by the member of Model in beat object, and display it in the front-end page by Velocity.

You may have heard of **Model View Controler** Pattern, which separate the business logic with front-end ui. Yes, this is exactly pattern that Argo follows. 
