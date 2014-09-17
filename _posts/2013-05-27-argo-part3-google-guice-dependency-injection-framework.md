---
layout: post
tagline: "Supporting tagline"
tags : [Argo, github, opensource]
title : Argo part 3 - Google Guice Dependency Injection Framerwork
---
{% include JB/setup %}

In part1, we known that Argo use Dependency Injection to do the configuration. If you looked the maven dependency of Argo, you will find it has a dependency on *guice-3.0.jar*.

Argo used this Google-guice [http://code.google.com/p/google-guice/](http://code.google.com/p/google-guice/) Dependency Injection framework for  interface injection.

### Google-guice ###

The main components of Google-guice dependency injection framework are @Inject annotation, and the AbstractModule class.

Let's see an example on google code

The RealBillingService class relies on interface CreditCardProcessor and TransactionLog in constructor. 

{% highlight java %}
class RealBillingService implements BillingService {
  private finalCreditCardProcessor processor;
  private finalTransactionLog transactionLog;

  @Inject
  RealBillingService(CreditCardProcessor processor,
      TransactionLog transactionLog){
    this.processor= processor;
    this.transactionLog= transactionLog;
  }

  @Override
  public Receipt chargeOrder(PizzaOrder order,CreditCard creditCard){
    ...
  }
}
{% endhighlight %}

Notice that there is a @Inject annotation on the constructor. So something will be happen in Guice when the constructor of RealBillingService is called.
 
AbstractModule builds the object-graph of Guice, which binds the interface with the implementation by overriding the function Configure.

{% highlight java %}
public class BillingModule extends AbstractModule {
  @Override 
  protected void configure() {

     /*
      * This tells Guice that whenever it sees a dependency on a TransactionLog,
      * it should satisfy the dependency using a DatabaseTransactionLog.
      */
    bind(TransactionLog.class).to(DatabaseTransactionLog.class);

     /*
      * Similarly, this binding tells Guice that when CreditCardProcessor is used in
      * a dependency, that should be satisfied with a PaypalCreditCardProcessor.
      */
    bind(CreditCardProcessor.class).to(PaypalCreditCardProcessor.class);
  }
}
{% endhighlight %}

In this example, it bind the interface TransactionLog with DatabaseTransactionLog, and bind CreditCardProcessor with PaypalCreditCardProcessor.

We can also conclude that @Inject always comes with the bind.

After that, a Guice Injector instance can be created:

{% highlight java %}
public static void main(String[] args) {
  /*
   * Guice.createInjector() takes your Modules, and returns a new Injector
   * instance. Most applications will call this method exactly once, in their
   * main() method.
   */
  Injector injector = Guice.createInjector(new BillingModule());

  /*
   * Now that we've got the injector, we can build objects.
   */
  RealBillingService billingService = injector.getInstance(RealBillingService.class);
  ...
}
{% endhighlight %}

Let's check the usage in Argo .

### Guice in Argo ###

{% highlight java %}
//Argo.java

public class Argo {  
  
    public ArgoDispatcher init(ServletContext servletContext,  
                               GroupConvention groupConvention) {  
        …  
  
        List<Module> modules = Lists.newArrayList();  
        modules.add(new ArgoModule(this));  
  
        Module groupModule = groupConvention.group().module();  
        if (null != groupModule)  
            modules.add(groupModule);  
  
        Module projectModule = groupConvention.currentProject().module();  
        if (null != projectModule)  
            modules.add(projectModule);  
  
        servletContext.log("preparing an injector");  
        this.injector = buildInjector(modules);  
        servletContext.log("injector completed");  
  
        …  
    }  
  
  
    private Injector buildInjector(List<Module> modules) {  
        return Guice.createInjector(modules);  
    }  

}

//ArgoModule

public class ArgoModule extends AbstractModule {  
  
    @Override  
    protected void configure() {  
  
        bind(ServletRequest.class).to(HttpServletRequest.class);  
        bind(ServletResponse.class).to(HttpServletResponse.class);  
  
        bind(BeatContext.class)  
                .annotatedWith(ArgoSystem.class)  
                .to(DefaultBeatContext.class);  
  
        bind(ActionResult.class)  
                .annotatedWith(Names.named("HTTP_STATUS=404"))  
                .toInstance(StatusCodeActionResult.defaultSc404);  
  
        bind(ActionResult.class)  
                .annotatedWith(Names.named("HTTP_STATUS=405"))  
                .toInstance(StatusCodeActionResult.defaultSc405);  
  
        bind(Action.class).annotatedWith(StaticActionAnnotation.class)  
                .to(StaticFilesAction.class);  
  
        bind(ClientContext.class).to(DefaultClientContext.class);  
        bind(Model.class).to(DefaultModel.class);  
  
        bind(MultipartConfigElement.class)  
                .toProvider(DefaultMultipartConfigElementProvider.class)  
                .in(Singleton.class);  
  
        // bind all controllers.  
        for (Class<? extends ArgoController> clazz : argo.getControllerClasses())  
            bind(clazz).in(Singleton.class);  
  }  
}  
{% endhighlight %}

As you can see, in Argo.init(), a set of Module instance is created, and then the Guice.CreateInjector() function is called to bind the interface and implements.
 

From the name of argo.getControllerClasses(), we could inspect that these Module are the collection of Controllers Annotated by Argo.