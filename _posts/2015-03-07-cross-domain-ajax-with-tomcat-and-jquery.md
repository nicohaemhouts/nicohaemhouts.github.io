---
layout: post
title: Cross-Domain AJAX with Tomcat and jQuery
tags: "CORS, Tomcat, Javascript, IE8, IE9, jQuery, jQuery.XDomainRequest, Java, Web Development"
summary: How to setup your Tomcat for Cross-Domain AJAX and how to deal with the Internet Explorer gotchas in IE8 and IE9 using jQuery.
---
##Cross-Origin Resource Sharing (CORS)
Cross-Domain,  or cross-origin,  AJAX requests are requests from a web page hosted on domain X to a server hosted on domain Y. As a safety precaution the browser prohibits this, unless the server says it's okay by sending the appropriate Http-headers. 

Why does this safety precaution exist? Well, imagine if you accidentally visited a malicious website that loaded your Facebook page in a hidden frame. That malicious website could then steal your unique identification token and start making all sorts of destructive requests on your behalf. You wouldn't know about it until it's too late and Facebook would simply assume that all those requests were coming from you.

To prevent this from happening a web server that receives a request from another domain than the one it is hosted on, has to tell the web browser whether this is allowed or not. This is called cross-origin resource sharing, or [CORS](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) for short.
 
CORS works by exchanging specific http-headers between the browser and the server. It is up to the web browser to enforce CORS by not allowing cross-origin requests from domain X if the server on domain Y doesn't allow them. 

The browser uses the  [same-origin policy](https://en.wikipedia.org/wiki/Same-origin_policy) to determine if the request it is about to send is from the same origin or not. If it's not from the same origin, the browser first sends a preflight request to the server with an http *Origin* header  that contains the domain the requesting page is hosted on.  So, if a page on domain X wants to send a request to domain Y, the browser sends the header *Origin: X* to the server on domain Y.

The server answers with the  *Access-Control-Allow-Origin*  header. This header either contains a list of allowed origin domains or a wildcard indicating any and all domains are allowed to send cross-origin requests.  If the origin domain is cleared, the browser sends the actual request. If it is not allowed, the request is blocked by the browser and you should get a nasty error in the console of your developer tools.

Unless your server is providing information available to the public at large, e.g. weather forecasts or stock quotes, the wild card is usually considered a bad idea. If you are about to put a wildcard in the *Access-Control-Allow-Origin*, ask yourself if it would be okay for a potentially malicious website to make requests on behalf of one of your users.  

There's more to CORS then what I've described here, but in most cases this will do and if not, it's enough to get you started. To find out more about CORS, check out the book ["CORS In Action"](http://manning.com/hossain/) by Monsur Hossain. He also has a website up at [http://enable-cors.org/](http://enable-cors.org/).

##Configuring Tomcat For CORS

Configuring your web application in Tomcat to support CORS is pretty straight forward: all you have to do is add a CORS filter to your web.xml.

{% highlight xml%}
<filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>org.apache.catalina.filters.CorsFilter</filter-class>
    <init-param>
        <param-name>cors.allowed.origins</param-name>
        <param-value>http://blog.nicohaemhouts.com</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
{% endhighlight %}

More configuration options are available. Check the documentation of the [Tomcat CORS Filter](https://tomcat.apache.org/tomcat-8.0-doc/config/filter.html#CORS_Filter) for details.

If you are not on Tomcat, you can also get the [CORS Filter from The Transaction Company](https://bitbucket.org/thetransactioncompany/cors-filter). Simply add the [following maven dependency](http://mvnrepository.com/artifact/com.thetransactioncompany/cors-filter) to your POM.xml to get it

{% highlight xml %}
<dependency>
	<groupId>com.thetransactioncompany</groupId>
	<artifactId>cors-filter</artifactId>
	<version>2.3</version>
</dependency>
{% endhighlight %}


##Making Cross-Domain AJAX requests with jQuery

With your java web application properly configured to allow CORS, making the AJAX requests with jQuery is as easy as can be. You don't have to do anything special.  Simply use *$.ajax* or one of its shorthand versions like you normally would, except this time you make the request to a resource on another domain. 

There are however two things to keep in mind. The first is that your request *must* be asynchronous. The jQuery *$.ajax* method has an *async* option that defaults to *true*, but you can set it to *false* to force the request to be synchronous. For CORS this is not allowed.

The second thing you should know involves an enterprise UI classic and nail in your coffin, our old *"friend"* Internet Explorer and especially versions 8 and 9.


##CORS With IE8 and IE9

Internet Explorer partially supports CORS since version 8. However it does things slightly different from other browsers.  Instead of using the *XMLHttpRequest* object to make the request, IE8 and IE9 use the special [XDomainRequest Object](https://msdn.microsoft.com/en-us/library/ie/cc288060%28v=vs.85%29.aspx)

jQuery doesn't support *XDomainRequest* out of the box, so you will have to support it yourself. Luckily there's a perfectly good plugin that does it all for you: [jQuery.XDomainRequest.js](https://github.com/MoonScript/jQuery-ajaxTransport-XDomainRequest)

It's a [jQuery AjaxTransport](https://api.jquery.com/jquery.ajaxtransport/) extension. The *jQuery.ajaxTransport()* method creates an object to handle the actual transmission of an AJAX request.  You can use this to override the default transmission of *$.ajax()*.  The *jQuery.XDomainRequest* plugin uses this to force transmission with the *XDomainRequest* object if it's on IE8 or IE9.

This means you can simply use your *$.ajax* and its shorthand methods as you normally would, the plugin will force transmission with *XDomainRequest* when needed. All you have to do is include the plugin in your page and you're good to go.

Bare in mind that only *GET* and *POST* methods are supported and that
data will always be posted with the *Content-Type* set to *text/plain*.

One annoyance I've found has to do with getting proper error messages from your server when using *XDomainRequest*. When I post something to the server I always have it validated in Spring MVC using the *@Valid* annotation. When there are errors on the *BindingResult* object I return them as JSON with an HTTP status code *422,"Unprocessable Entity"*. This will trigger the *fail* handler of the *$.ajax()* promise. I can then retrieve the errors from the *jqXHR* object that's passed to the fail handler. 

{% highlight js %}
$.post(...)
  .done(function (data) {    
    //...
  }).fail(function (jqXHR, status, error) {    
    console.log(jqXHR.responseText);    
  });
{% endhighlight %}


However, when *XDomainRequest* is used, you get no data back on error:

{% highlight json %}
{"readyState":4,"responseText":"","status":500,"statusText":"error"}
{% endhighlight %}

So, for IE8 and IE9 you'll have to write some generic error message to display to the user. Either that, or you'll have to validate in the browser as well as on the server. This might get tedious as you'll be repeating yourself:  you will have to have the same validations on the server as on the client. And not having the validations on the server is obviously not done: you have to validate everything that's posted to your server, no matter what.

##Conclusion

It is really easy to setup CORS for your java web application. All you have to do is add the CORS filter to your web.xml and include the *jQuery.XDomainRequest.js* plugin to support IE8 and IE9. 

The only real downside I can see is that if you have to support IE8 and IE9 (you probably will), you might have to repeat your data validation code on the client. 





    
    


