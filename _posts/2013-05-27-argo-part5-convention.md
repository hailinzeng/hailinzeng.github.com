---
layout: post
tagline: "Supporting tagline"
tags : [Argo, github, opensource]
title : Argo part 5 - Convention
---
{% include JB/setup %}

From part 4, we known that all of the request is handled by ArgoDispatcher.

	//ArgoFilter.init()  
	dispatcher = ArgoDispatcherFactory.create(servletContext);   
	dispatcher.init();  

### Group & Project Module ###

In default, ArgoDispatcher only contains the ArgoModule. But if you defined group Module, or project Module, they are included in too.

	public ArgoDispatcher init(ServletContext servletContext, GroupConvention groupConvention) {  
    	…  
        List<Module> modules = Lists.newArrayList();  
        modules.add(new ArgoModule(this));  
  
        Module groupModule = groupConvention.group().module();  
        if (null != groupModule)  
            modules.add(groupModule);  
  
        Module projectModule = groupConvention.currentProject().module();  
        if (null != projectModule)  
            modules.add(projectModule);  
        …  
	}

The group here in groupConvention means group like com.bj58，and the project here means project like HelloWorld. 

### Group Convention ###

You can customize the convention class implementation in:

	//GroupConvention.java
	public final static String annotatedGroupConventionBinder = "com.bj58.argo.GroupConventionBinder";  

	//GroupConventionAnnotation.java
    String projectConventionClass() default "com.bj58.argo.ProjectConfig";

ArgoDipatcherFactory used *annotatedGroupConventionBinder* to create a GroupConvention instance. The default is a one that does not exist, and returns DefaultGroupConvention class instance.

	public static GroupConvention getGroupConvention() {  
       String className = GroupConvention.annotatedGroupConventionBinder;  
  
       Class<?> clazz = null;  
       GroupConvention groupConvention;  
       try {  
           clazz = GroupConventionFactory.class.getClassLoader().loadClass(className);  
           groupConvention = GroupConvention.class.cast(clazz);  
       } catch (Exception e) {  
           groupConvention = null;  
       }  
  
       if (groupConvention != null)  
           return groupConvention;  
  
       ...  
	  
	   return new DefaultGroupConvention(conventionAnnotation, folder);  
	}  

DefaultGroupConvention is annotated with @GroupConventionAnnotation. 

	//GroupConventionAnnotation.java  
	public @interface GroupConventionAnnotation { 
		String groupConfigFolder() default "/opt/argo";  
		String groupLogFolder() default "{groupConfigFolder}/log";
		...  

	    // Argo only scans class prefix by com.bj58.argo
	    String groupPackagesPrefix() default "com.bj58.argo";
	
		// only scan Controllers matched this pattern
	    String controllerPattern() default ".*\\.controllers\\..*Controller";
	}

This annotations defines the config folder, log folder, the convention that controllers should meet. 

The project Convention is similar, which returns a anonymous inner class:

	//DefaultGroupConvention.java
    ProjectConvention parseProjectConvention(GroupConventionAnnotation groupConventionAnnotation) {
        String className = groupConventionAnnotation.projectConventionClass();

        Class<?> clazz;
        try {
            clazz = Thread.currentThread().getContextClassLoader().loadClass(className);
        } catch (ClassNotFoundException e) {
            return new ProjectConvention() {
                @Override
                public String id() {
                    return "";
                }

                @Override
                public void configure(Binder binder) {
                }
            };
        }
		...
	}

As we expected, Group is in a higher level compared with Project. 

	public DefaultGroupConvention(GroupConventionAnnotation groupConventionAnnotation, File folder) {  
    	this.folder = folder;  
  
		// load projectConvention class
		// according to GroupConventionAnnotation.projectConventionClass()
    	ProjectConvention projectConvention = parseProjectConvention(groupConventionAnnotation);  
  
    	// load project configuration, 
		// in particular, scan all Controllers class here
    	projectConfig = parseProjectConfig(projectConvention, groupConventionAnnotation);  

    	Map<String, String> configInfoMap = parseGroupConventionPath(groupConventionAnnotation, projectConvention);  
  
		// load group configuration
    	groupConfig = parseGroupConfig(groupConventionAnnotation, configInfoMap);    
	}  


The DefaultGroupConvention implements interface GroupConvention, overrides group() which returns the member groupConfig，and overrides currentProject() which returns the member projectConfig. 


### Group & Project Config ###

Followed by parseProjectConfig()

	private ProjectConfig parseProjectConfig(ProjectConvention projectConvention,  
                                         GroupConventionAnnotation groupConventionAnnotation) {  
	    Set<Class<? extends ArgoController>> controllersClasses = parseControllers(groupConventionAnnotation);  
	  
	    return new DefaultProjectConfig(projectConvention.id(), controllersClasses, projectConvention);  
	}  

//parse controller that meets convention
  
	Set<Class<? extends ArgoController>> parseControllers(GroupConventionAnnotation groupConventionAnnotation) {  
		// get classes in package "com.bj58.argo"
	    Set<Class<?>> classSet = ClassUtils.getClasses(groupConventionAnnotation.groupPackagesPrefix());  
	  
	    Pattern controllerPattern = Pattern.compile(groupConventionAnnotation.controllerPattern());  
	  
	    ImmutableSet.Builder<Class<? extends ArgoController>> builder = ImmutableSet.builder();  
	  
		// class that matched with convention
	    for (Class<?> clazz : classSet)  
	        if (applyArgoController(clazz, controllerPattern))  
	            builder  
	                .add((Class<? extends ArgoController>) clazz)  
	                .build();  
	  
	    return builder.build();  
	}  

// parseGroupConfig, default Module of GroupConfig is EmptyModule.

	private GroupConfig parseGroupConfig(GroupConventionAnnotation groupConventionAnnotation  
	        , Map<String, String> configInfoMap) {  
	    String configPath = configInfoMap.get(GROUP_CONFIG_FOLDER);  
	    String logPath = configInfoMap.get(GROUP_LOG_FOLDER);  
	  
	    Module module =  EmptyModule.class.isAssignableFrom(groupConventionAnnotation.groupModule())  
	            ? EmptyModule.instance  
	            : newInstanceByClass(groupConventionAnnotation.groupModule(), "");  
	  
	    return new DefaultGroupConfig(getDir(configPath), getDir(logPath), module);  
	  
	}  

### Inject ###

The information obtained above is used in Argo.init(). 

	//Argo.init()
	this.groupConvention = groupConvention;  
	this.controllerClasses = groupConvention.currentProject().controllerClasses();  
	  
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

Guice build the Injector, and used the configuration in groupModule, projectModule, controller Module to Inject.

### Conclusion ###

GroupConventionFactory used the GroupConvention.annotatedGroupConventionBinder class, and the annotation @GroupConventionAnnotation on it to get the config folder, log folder, patterns that controllers should follow.

Of course, the default projectModule is a anonymous ProjectConvection, and the default groupModule is NullModule, they both do nothing in configure(), where you can call bind() here to do injection.