---
layout: post
tagline: "Supporting tagline"
tags : [java]
title : get method parameter names in java
---
{% include JB/setup %}

Recently, i was maintaing a long last web project that uses an old version of our web framework (open-source on github now as the name of [Argo](http://github.com/58code/Argo)).

I wrote the following Action, and annotated with @Path with a parameter `id`.

{% highlight java %}

@Path("storm/delete/{id}")
@POST
@Login
public ActionResult delete(Integer id){

	// logic of delete

	try {

	} catch (Exception e) {

	}

	return ActionResult.view("job/json");
}

{% endhighlight %}

When browsing url `storm/delete/1`, it should bind the parameter `Integer id` with the value `1` in url, but it was not happen. I debuged, and find the parameter `id` is still `null`.

I digged into the source code, and find the part to bind the url parameter with Action parameter.

{% highlight java %}

for(int i = 0; i < paramNames.size(); i++){
	String paramName = paramNames.get(i);
	Class<?> clazz = paramTypes.get(i);
	
	String v = urlParams.get(paramName);
	
	// bind directly for primitive types
	if (v != null) {
		if(converter.canConvert(clazz)){
			param[i] = converter.convert(clazz, v);
			continue;
		}
	}
	
	if(converter.canConvert(clazz)) continue;
	param[i] = BindAndValidate.bindAndValidate(clazz).getTarget();
}
{% endhighlight %}

It loop through Action parameters, and tries to find the parameter value in url.

The `paramNames` array is evalueate by `paramNames = getMethodParamNames()`

{% highlight java %}

/**
 * get method parameter names with javassist
 * @return
 */
private String[] getMethodParamNames() {
	Class<?> clazz = this.controllerInfo.controller.getClass();
	Method method = this.actionMethod;
	try {
		ClassPool pool = ClassPool.getDefault();
		
		pool.insertClassPath(new ClassClassPath(clazz));

		CtClass cc = pool.get(clazz.getName());
		
		// DEBUG, cannot resolve overloading method
		// CtMethod cm = cc.getDeclaredMethod(method.getName());
		
		String[] paramTypeNames = new String[method.getParameterTypes().length]; 
		for (int i = 0; i < paramTypes.length; i++)  
			paramTypeNames[i] = paramTypes[i].getName();  
		CtMethod cm = cc.getDeclaredMethod(method.getName(), pool.get(paramTypeNames));
		
		MethodInfo methodInfo = cm.getMethodInfo();
		
		CodeAttribute codeAttribute = methodInfo.getCodeAttribute();
		LocalVariableAttribute attr = (LocalVariableAttribute) codeAttribute
				.getAttribute(LocalVariableAttribute.tag);
		if (attr == null) {
			throw new RuntimeException("class:"+clazz.getName()
					+", have no LocalVariableTable, please use javac -g:{vars} to compile the source file");
		}
		
		String[] paramNames = new String[cm.getParameterTypes().length];
		int pos = Modifier.isStatic(cm.getModifiers()) ? 0 : 1;
		
		for (int i = 0; i < paramNames.length; i++)
			paramNames[i] = attr.variableName(i + pos);
		for (int i = 0; i < paramNames.length; i++) {
			System.out.println(paramNames[i]);
		}
		
		return paramNames;

	} catch (NotFoundException e) {
		e.printStackTrace();
		return new String[0];
	}
}

{% endhighlight %}

I set a debug point in `getMethodParamNames`, and finally found it fails because the `parameNames` is not correctly evaluated.

<img width="782px" alt="get-method-param-name-in-java-debug-stacktrace" src="{{ site.url }}/assets/images/get-method-param-name-in-java-debug-stacktrace.jpg">

(left-up corner: the Action, right-up corner: `getMethodParamNames()` function)

The `paramNames` array here is `{"this"}`, which should be `{"id"}`. I watches some expression, and found the `attr.variableNames` array is infact `{"e", "this", "id", ... }`

I did some search on google, and found a tool [jclasslib](https://github.com/ingokegel/jclasslib) to browse LocalVariableAttribute of a `.class` file.

<img width="782px" alt="get-method-param-name-in-java-localvariabletable" src="{{ site.url }}/assets/images/get-method-param-name-in-java-localvariabletable.jpg">

This explains why it failed, as there is another local variable `e` which appear before every other local variables.

It seems the `LocalVariableTable` is not orderred by appearence, and I verrified the guessing by javadoc.

`If LocalVariableTable attributes are present in the attributes table of a given Code attribute, then they may appear in any order.` (from [http://docs.oracle.com/javase/specs/jvms/se5.0/html/ClassFile.doc.html#1546](http://docs.oracle.com/javase/specs/jvms/se5.0/html/ClassFile.doc.html#1546) )

The `getMethodParamNames` implementation assumes the LocalVariableTable is orderred, which of course fails if the compiled `.class` is not orgnized in this way.

And finally, i found the correct way the get a orderred local parameter names here [https://github.com/jboss-javassist/javassist/issues/14](https://github.com/jboss-javassist/javassist/issues/14)

	chibash commented on Jul 25
	I see. To get the k-th local variable, do this: 

{% highlight java %}
	for (int i = 0; i < parameterNames.length; i++)
	parameterNames[attr.index(i)] = attr.variableName(i);
{% endhighlight %}

	The local variable table is not sorted by the local variable index.
	
