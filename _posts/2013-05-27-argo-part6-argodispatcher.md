---
layout: post
tagline: "Supporting tagline"
tags : [Argo, github, opensource]
title : Argo part 6 - ArgoDispatcher
---
{% include JB/setup %}

### ArgoFilter ###

Let's back to ArgoFilter. In ArgoFilter.init(), it creates a ArgoDispatcher instance by calling Argo.init().

In ArgoFilter.doFilter(), It simply hands over the request to ArgoDispatcher.
{% highlight java %}
@Override  
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {  
    HttpServletRequest httpReq = (HttpServletRequest) request;  
    HttpServletResponse httpResp = (HttpServletResponse) response;  
    dispatcher.service(httpReq, httpResp);  
}  
{% endhighlight %}


### Request Dispatcher ###

From the annotation @ImplementedBy(DefaultArgoDispatcher.class), we know that this is inject by guice, and the implementation class is DefaultArgoDispatcher.
{% highlight java %}
@ImplementedBy(DefaultArgoDispatcher.class)
// @ImplementedBy equals to bind(ArgoDispatcher.class).to(DefaultArgoDispatcher.class)
public interface ArgoDispatcher {
	...
}
{% endhighlight %}

The StaticActionAnnotation, MultipartConfigElement in DefaultArgoDispatcher can be injected.

{% highlight java %}
@Inject  
public DefaultArgoDispatcher(Argo argo, Router router, StatusCodeActionResult statusCodeActionResult, MultipartConfigElement config) {  
    this.argo = argo;  

    this.router = router;  
    this.statusCodeActionResult = statusCodeActionResult;  
    this.config = config;  

    this.logger = argo.getLogger(this.getClass());  

    logger.info("constructed.", this.getClass());  
}  
{% endhighlight %}

StaticActionAnnotation, MultipartConfigElement are bind with an implementation in ArgoModule.

{% highlight java %}
@Override  
protected void configure() {  
	…
    bind(Action.class).annotatedWith(StaticActionAnnotation.class)
            .to(StaticFilesAction.class);

	bind(MultipartConfigElement.class)  
           .toProvider(DefaultMultipartConfigElementProvider.class)  
           .in(Singleton.class);  
	…  
}
{% endhighlight %}

DefaultArgoDispatcher.service() that ArgoFilter called:

{% highlight java %}
public void service(HttpServletRequest request, HttpServletResponse response) {  
    ArgoRequest argoRequest = new ArgoRequest(request, config);  
  
    try {  
        BeatContext beatContext = bindBeatContext(argoRequest, response);  
  
        route(beatContext);  
    } finally {  
        Closeables.closeQuietly(argoRequest);  
    }  
}  
{% endhighlight %}

### BeatContext ###

bindBeatContext:

{% highlight java %}
private BeatContext bindBeatContext(HttpServletRequest request, HttpServletResponse response) {  
    Context context = new Context(request, response);  
    localContext.set(context);  
  
    BeatContext beat = argo.injector().getInstance(defaultBeatContextKey);  

    // add default __beat to Model  
    beat.getModel().add("__beat", beat);  
    context.setBeat(beat);  
    return beat;  
}  
{% endhighlight %}

BeatContext is bind to DefaultBeatContext, as in ArgoModule.configure()

{% highlight java %}
@Override  
protected void configure() {  
	…
	bind(BeatContext.class)  
       .annotatedWith(ArgoSystem.class)  
       .to(DefaultBeatContext.class);  
	…  
    bind(Model.class).to(DefaultModel.class);
	// Model is bind to DefaultModel, which is implemented by a Map.
}
{% endhighlight %}


Thre are two class implements the interface BeatContext: BeatContextWrapper and DefaultBeatContext. BeatContextWrapper is much simple. DefaultBeatContext can be injected, and ClientContext now is bind to DefaultClientContext.

{% highlight java %}
@ArgoSystem  
public class DefaultBeatContext implements BeatContext {  
    @Inject  
    public DefaultBeatContext(HttpServletRequest request, HttpServletResponse response, Model model, ClientContext clientContext, ServletContext servletContext) {  
        this.request = request;  
        this.response = response;  
        this.model = model;  
        this.clientContext = clientContext;  
        this.servletContext = servletContext;  
    }  
}  
{% endhighlight %}

### Route request to Action ###

DefaultArgoDispatcher.Route():

{% highlight java %}
//DefaultArgoDispatcher.route()  
private void route(BeatContext beat) {  
    try {  
        ActionResult result = router.route(beat);  

        if (ActionResult.NULL == result)  
            result = statusCodeActionResult.getSc404();  

        result.render(beat);  

    } catch (Exception e) {  

        statusCodeActionResult.render405(beat);  

    } finally {  
        localContext.remove();  
    }  
}  
{% endhighlight %}

Router is annotated by @ImplementedBy. The implementation class is DefaultRouter.

{% highlight java %}
@ImplementedBy(DefaultRouter.class)  
public interface Router {  
    public ActionResult route(BeatContext beat);  
}  
{% endhighlight %}


DefaultRouter initialized the member *actions* in constructor.

{% highlight java %}
@Inject  
public DefaultRouter(Argo argo, @ArgoSystem Set<Class<? extends ArgoController>> controllerClasses, @StaticActionAnnotation Action staticAction) {  
  
    this.argo = argo;  
  
    argo.getLogger().info("initializing a %s(implements Router)", this.getClass());  
  
    this.actions = buildActions(argo, controllerClasses, staticAction);  
  
    argo.getLogger().info("%s(implements Router) constructed.", this.getClass());  
}  
{% endhighlight %}

The buildActions() above instantiated all controller class in getControllerInstances(), extract and saved meta info in ControllerInfo, and called ControllerInfo.analyze() on every Controller and the staticAction injected.

Later, an override buildActions is called.

{% highlight java %}
List<Action> buildActions(Set<ArgoController> controllers, Action staticAction) {  
    
    for (ArgoController controller : controllers) {  
        ControllerInfo controllerInfo = new ControllerInfo(controller);  
        List<ActionInfo> subActions = controllerInfo.analyze();  
        ...  
    }  
}  
{% endhighlight %}

analyze() finds all matched *Methods*, (public functions returns ActionResult, and returns a ActionInfo). ActionInfo will finds annotations, especially pre/post interceptors，url patterns，params in url patters，http request type GET/POST in constructor. And the are ordered by the attribute *order* in @Path(order=1000).


actions is used in DefaultRouter.route(), they applied matchAndInvoke() on beat.

MethodAction.matchAndInvoke checks the Http request type，checks if url matches the Action @Path url pattern by AntPathMatcher. If matches, then execute the interceptor, and invoke on the real actor.

{% highlight java %}
public RouteResult matchAndInvoke(RouteBag bag) {  
    if (!actionInfo.matchHttpMethod(bag))  
        return RouteResult.unMatch();  
  
    Map<String, String> uriTemplateVariables = Maps.newHashMap();  
  
    boolean match = actionInfo.match(bag, uriTemplateVariables);  
    if (!match)  
        return RouteResult.unMatch();  
  
    // PreIntercept  
    for(PreInterceptor preInterceptor : actionInfo.getPreInterceptors()) {  
        ActionResult actionResult = preInterceptor.preExecute(bag.getBeat());  
        if (ActionResult.NULL != actionResult)  
            return RouteResult.invoked(actionResult);  
    }  
  
    ActionResult actionResult = actionInfo.invoke(uriTemplateVariables);  
  
    // PostIntercept  
    for(PostInterceptor postInterceptor : actionInfo.getPostInterceptors()) {  
        actionResult = postInterceptor.postExecute(bag.getBeat(), actionResult);  
    }  
  
    return RouteResult.invoked(actionResult);  
}  
{% endhighlight %}

During the executing of interceptor, if any preInterceptor returns NULL, then the following interceptors and action invoke will not be executed.

Action.invoke is java reflection call that calls the method/Action of Controller with the params uriTemplateVariables.

### Rend the Page ###
result.render（）

The class implements interface ActionResult contains class implementations to handle 404，405，redirect，redirect301，VelocityViewResult，InnerPrintWriter，DefaultStaticResult.

Take VelocityViewResult for example. It take the Model/map in beatContext.getModel(), and call VelocityWrite to render the velocity template web page.

{% highlight java %}
// init context: take the Model
Context context = new VelocityContext(beatContext.getModel().getModel());

// render:
VelocityWriter vw = null;
try {
    vw = new VelocityWriter(response.getWriter());
    template.merge(context, vw);
    vw.flush();
} 
{% endhighlight %}

By now, we have finished the whole journey of Argo.

**EOF**
