---
layout: post
title: "JavaScript: Accessing Nested Object Properties Using a String."
tags: "JavaScript, nested properties, string, code"
summary: A simple JavaScript function to read the nested property values on an object using a string as the path to the property.
---

A question I see coming back every so often is how you can access a nested property on a JavaScript object using a string. There are a number of 
ways you can do this involving some looping or even recursion. Personally I prefer to use [Array.prototype.reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 
which is supported by every major browser and even Internet Explorer has had it since version 9.  
    
### The Use Case

We want to be able to access nested properties using a string representing the path to the property. We should be able to support the following cases:

{% highlight js %}
var snack = {
    id: "0001",
	name: "Cake",
	batters:
		{
			batter:
				[
					{ id: "1001", type: "Regular" },
					{ id: "1002", type: "Chocolate" },
					{ id: "1003", type: "Blueberry" },
				]
		}
};
 
getNested(snack, 'batters.batter').length;                 // --> 3 
getNested(snack, 'batters.batter.2.id');                   // --> "1003"
getNested(snack, 'batters.batter[1].id');                  // --> "1002"
getNested(snack, 'batters.batter[99].id') || 0;            // --> 0
getNested(snack, 'batters.batter.nutrition') || 'none';    // --> 'none'
getNested(snack, 'batters/batter/0/id', '/');              // --> "1001"
 
{% endhighlight %}

So, we should be able to support array indices using both square brackets (*[1]*) and no brackets at all. We
should return some *falsy* value if we come across a non-existing property. I prefer to return *undefined* as that is
what you would get if you tried to access a non-existing property the conventional way. 

Lastly the dot-syntax is default, but we should also support other path separators. Obviously, if any of the keys contain
the path separator it will fail and the function should return *undefined*

### The Solution

{% highlight js %}
function getNested (theObject, path, separator) {
    try {
        separator = separator || '.';
    
        return path.
                replace('[', separator).replace(']','').
                split(separator).
                reduce(
                    function (obj, property) { 
                        return obj[property];
                    }, theObject
                );
                    
    } catch (err) {
        return undefined;
    }   
}
{% endhighlight %}

A couple of things about this solution:

- We don't need to bother with square brackets for array indices or checking if the property is a number or not. This is because *batter["2"] === batter[2]*. 
- We pass in *theObject* as the starting value for the *reduce()* because otherwise it would take the first element in the array produced by splitting the path.
- Everything is wrapped in a *try-catch*-block because you can have a non-existing property in the middle of your path and then the next
call to the reduce function would try to do *undefined['someProperty']*. That would be a showstopper. 
- This doesn't work if the object is an array and the path starts with an array index wrapped in square brackets, like *'[2].foo.bar*. 
This is because the getNested-function replaces the opening square bracket by a dot. A property path can't start with a dot. It would however 
work if you omitted the square brackets and simply passed in *'2.foo.bar'*

You can take this solution one step further and add the getNested-function to the prototype of *Object*. This way every object 
would have a *getNested()* method (or whatever name you choose for it). However, I'm not a fan of altering prototypes of built-in 
objects.  

