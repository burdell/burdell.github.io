---
title: $parse - Power to the Directive
author: nathan griffin
date: 2014-01-17
template: singlepage.html
---

I mentioned in a [previous post](http://nathaniscool.com/articles/angularjs/) about switching from using Backbone.js to Angular.js for front-end development at work. One of the goals of that switch was to basically allow a developer to create a page using just semantic markup and have 90% or more of the front-end functionality completed. 

Anybody familiar with Angular knows exactly how to accomplish this: *directives*. Angular gives you a ton of directives out of the box. Stuff like showing elements, hiding elements, disabling elements, looping, conditionals, .etc all come for free and can be dropped right into your HTML. Boom. 

Anybody who has attempted to write their own at some point wonders: How does Angular make them so powerful? Why can't I have *nice* things?

### A Solution ###
One of the coolest and most "magical" parts that people first notice with Angular is the ability to pass in actual JavaScript expressions straight from the DOM. I was really curious as to how exactly Angular handles this so well. Angular *feels* magical sometimes, but at the end of the day it's just JavaScript. So I went to the source code, and [AHA](https://github.com/angular/angular.js/blob/master/src/ng/parse.js)! They've written their own parsing engine. 

And it turns out *you too* can parse stringed expressions by injecting their [$parse](http://docs.angularjs.org/api/ng.$parse) service where you need it. This had a profound impact when trying to write directives that could be 1) re-used and 2) entirely defined through markup.

### Simple Examples ###
<iframe width="100%" height="300" src="http://jsfiddle.net/5SFj7/5/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Ok, so ... ?###
If you've written directives you're probably well aware that HTML attributes passed into directive are just strings. Boring strings. What about complex objects? What about functions?

The simplest solution to this (and the one I went to first) is *"SCOPE!"*. The directive and controller share this magical little object called scope. Anything I chunk in scope in the controller, I have access to in scope in the directive. So if I need a object / function / value in the directive, it's super easy to just pass the directive the *name* of it, and boom, access.

But remember, our goal for Angular was to have a developer come in and drop HTML declarations like they're hot and be done with it. If I have to go to the controller and set up a bunch of stuff every time I have to use that directive, the glimmer of semantic markup is dulled. I'm all for following MVC and separating out concerns between the view and controller, but this feels really restrictive and unnecessary.

### Rethinking the Approach ###
We had a need for a line-item directive for some upcoming functionality we were going to do, and had our own line-item jQuery plugin that worked pretty well, so the directive for this was just going to be a wrapper around that which took all the meta-data from markup and translated it into the options object for the plugin. Kinda like this:

<pre class="prettyprint">
	&lt;items ng-model='model.LineItemList' default-length='5'&gt;
		&lt;column field='Name'&gt;&lt;/column&gt;
		&lt;column field='TotalOrdered'&gt;&lt;/column&gt;
		&lt;column field='Cost'&gt;&lt;/column&gt;
		&lt;column field='TotalPrice'&gt;&lt;/column&gt;
	&lt;/items&gt;
</pre>

The plugin takes a list of column objects, and we could set up set up a multiplication calculation inside of there.

<pre class="prettyprint">
	columns = [
		{ field: 'Name' },
		{ field: 'TotalOrdered' }
		{ field: 'Cost' }
		{ 
			field: 'TotalPrice', 
			calculation: 
			{ 
				operator: '*', 
				baseField: 'TotalOrdered', 
				multiplier: 'Cost' 
			} 
		}
	];
</pre>

With $parse we could just pass in that same object into the directive as a string. Like this:

<pre class="prettyprint">
	&lt;column 
		field='TotalPrice' 
		calculation='{ operator="*", baseField="TotalOrdered" multiplier="Cost"}'&gt;
	&lt;/column&gt;
</pre>

But does this make life easier? Yes, it works, but it's weird, clunky, and just feels difficult. 

And we were definitely going to need addition and subtraction. What if we need 4 numbers? And what if we need something like "TotalOrdered \* TotalCost / (TotalCost \* 3.14)"? If you're sneaky, you're probably already suspecting that the plugin would hold an object of functions keyed by the 'operator' attribute, which means every different calculation requires a developer to go in and manually write the logic for that calculation.

This would be so much easier:

<pre class="prettyprint">
	&lt;column field='TotalPrice' calculation='TotalOrdered * TotalCost'&gt;&lt;/column&gt;
</pre>

And with $parse this is *incredibly* easy.

I wrote a simple calculator directive to show the idea behind doing this. The calculator is declared like this:

<pre class="prettyprint">
	&lt;calculator 
		values="{ pi: 3.14, r: 12, e:.66789 }" 
		calculations="pi * r * r, e * r" &gt;
	&lt;/calculator&gt;
</pre>

The idea is that any time you need to do a calculation, pass the data into a function and call $parse. In our line-item directive, we pass a function into the plugin so that the jQuery doesn't need to mix with Angular, and the plugin uses that function and passes it whichever row of data is needed.

This calculator is slightly simpler in that the data is just one object ($scope.fieldValues) instead of multiple rows of objects, and I loop through all the calculations ($scope.calculations) and calculate the values. 

<pre class="prettyprint">
	$scope.$watch("fieldValues", function(){
			var fieldValueNumbers = $scope.getFieldValueNumbers();
			_.each($scope.calculations, function(calc) {
				calc.value = $scope.calculate(calc.string, fieldValueNumbers);
			});
		}, true);

	$scope.calculate = function(calculationString, values){
			try {
				var value = $parse(calculationString)(values);
				return (_.isFinite(value) ? value : 0);
			} catch (e) {
				return "Invalid expression :("
			}
		};
</pre>

(If you're wondering, \_.each and \_.isFinite are from [Underscore](http://underscorejs.org/))

The full working example:
<iframe width="100%" height="300" src="http://jsfiddle.net/GRKDQ/2/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

The GitHub repo for it is [here](https://github.com/burdell/AngularCalculator) if you want it. 





