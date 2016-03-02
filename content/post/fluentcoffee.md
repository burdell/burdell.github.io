---
title: FluentCoffee - A Fluent Validator in CoffeeScript
author: nathan griffin
date: 2014-04-27
template: singlepage.html
---

I went to a talk on CoffeeScript a few weeks ago and it sorta sparked my interest in it. I'd looked at it a long time ago, but hadn't actually done anything with it, so I decided to write something small to try it out.

The result was [FluentCoffee](https://github.com/burdell/FluentCoffee), a fluent validator. Doing things fluently just means you chain together calls to functions which accomplishes what you're trying to do. 

### Basic Validation ###
Everything hinges off these basic validation functions:

- GreaterThan
- LessThan
- Between
- EqualTo
- Contains

There is also _Length()_ and _Not()_ that can be used (where it makes sense). 

So for primitive values (strings, numbers, arrays, booleans, etc) you validate them this way:
<pre class="prettyprint">
	primitiveValidation = new Validator()
		.For [1, 2, 3, 4, 6], "number array"
			.Contains 2
			.Not().Contains(5)
		.For "how now brown cow", "stupid saying"
			.Length().GreaterThan 10
		.For 3 * 10, "calculation"
			.LessThan 340
			.Between 250, 275
		.For 40 < 50
			.EqualTo true
	.Assert()
</pre>

The first parameter is the value, the second is the *name* of the value. The second parameter is optional, defaulting to "Value" if you don't include it.

This returns an object with a boolean _valid_ attribute, and list of object that contain the error message and the name of offending value. The above validation would return this:

<pre class="prettyprint">
	{
		valid: false,
		errors: [
			{
				value: "calculation",
				message: "calculation must be between 250 and 275"
			}
		]
	}
</pre>

### Object Validations ###
Object validation has all the same validations, but you just tell it which attribute to validate when you call _.For()_ on it. You have to qualify whether an attribute is required (_Require()_) or optional (_Optional()_). If an attribute is required, the validator will check its existance and add an error if it is not there.

<pre class="prettyprint">
	objMe = 
		name: "Nathan Griffin",
		age: 82,
		numberOfAppendages: 20,
		faveCereal: "Honey Bunches of Oats",
		job: "Code Ninja",
		phoneOpSystem: "Android"

	validation = new Validator()
		.For objMe
			.Require "age"
				.LessThan 30
				.Not().EqualTo 29
			.Require "numberOfAppendages"
				.Between 15, 25
			.Require "name"
				.EqualTo "Nathan Griffin"
			.Require "phoneOpSystem"
			.Require "job"
				.Contains "Ninja"
				.Length().GreaterThan 5
			.Optional "nickname"
				.Not().Contains "Dawg" 
		.Assert()
</pre>

### Function Validation ###
For function validation you just call _WithParameters()_ to evaluate the result as one of the previously mentioned validation types (or even one or more function validations if you're feeling sassy)

<pre class="prettyprint">
	objMe = 
		name: "Nathan Griffin",
		job: "Code Ninja",
		address: "1234 Sesame St Atlanta GA 30305",
		favoriteCereal: "Honey Bunches of Oats"

		objBurdell =
			name: "George P. Burdell",
			job: "trollol",
			age: 87,
			favoriteCereal: "Fruit Loops"

		identityFunction = (isMe) ->
			if isMe then objMe else objBurdell

		functionValidation = new Validator()
			.For identityFunction
				.WithParameters true
					.Require "name"
						.EqualTo "Nathan Griffin"
					.Require "job"
						.Contains "Ninja"
				.WithParameters()
					.Require "name"
						.EqualTo "George P. Burdell"
					.Require "job"
						.Contains "troll"
					.Require "age"
						.GreaterThan 50
						.LessThan 90
			.Assert()
</pre>
Not sure if it's obvious, but you can chain all these together. Look at [coffeeTester.coffee](https://github.com/burdell/FluentCoffee/blob/master/coffeeTester.coffee) to see that.

So that's basically it. I'm sure there's weird stuff / bugs if you try to be stupid with it, so be warned. 

One thing that'd be cool that it currently doesn't do would be to "cache" the validation so that I could validate all the steps at one point, do something else, then run the validation again without having to re-state the validations. This would also probably mean that the validations wouldn't actually be evaluated until you called _Assert()_, which I could see having some value. I almost started to refactor it to do that, but got kinda meh about it since there's something else I want to start.

Some quick thoughts on CoffeeScript...

### "hi h8rs" - CoffeeScript ###
_“There are three things I have learned never to discuss with people... Religion, Politics, and CoffeeScript.”_ 
― Charles M. Schulz, mostly

In the world of JavaScript, CoffeeScript gets a bunch of hate. It seems most people fall into either the camp of "_WOW I LOVE THIS_" or "_THIS IS VILE JIVE_". 

Most people probably dislike CoffeeScript for the same reason people dislike a bunch of things:

- It's unfamiliar
- It changes the way they've done stuff
- They misunderstand what it's trying to do

I think CoffeeScript really should be though of more like a *tool* rather than a *language*. Just like an IDE or a fancy text editor or a linter, CoffeeScript is simply a tool employed to write JavaScript. Yes, it's an abstraction and therefore an intrinsically more complicated way to write it, but at the end of the day you're generating JavaScript. Tools and support for CoffeeScript were (apparently) lacking in the past, but that's just not a valid argument against it anymore. I had zero issues with getting it up and running with Sublime Text on OS X within minutes. And I'm sure there are even more tools out there that I didn't take advantage of.

I used to have the commonly held position of *"It's better to just learn to write GOOD JavaScript"*, but trying to write anything non-trivial in CoffeeScript actually requires a pretty in-depth understanding of JavaScript. If you don't have a pretty firm grasp of JavaScript, your CoffeeScript is just going to either end up being terrible CoffeeScript instead of terrible JavaScript, or full of weird JavaScript-y CoffeeScript workarounds. 

I actually found CoffeeScript to be pretty enjoyable to use. It was simple, yet expressive and did a good job of getting out of your way. No clunky anonymous function declarations, built in null checking (really, this feature alone might be worth it), expressive 1-line conditionals, super easy handling of arbitrary parameter lengths, etc. The list goes on. 


### ...but would I want to switch to CoffeeScript permanently? ###
I'd probably be interested in doing more stuff in CoffeeScript, but I'm not sure I'm ready to slip into robes of a CoffeeScript evangelist quite yet. If I was part of team where everybody really bought into the idea, I don't think I would hate that. A big part of that is it would be much, much easier to have a syntatically homogenous codebase in CoffeeScript. 

Like I mentioned earlier, the endgame of CoffeeScript is still JavaScript. I didn't like having the extra step when debugging what I was writing. Luckily, FluentCoffee ended up only being ~120 lines of code, so I pretty much knew where errors were happening. On a much larger codebase I'm sure that quickly becomes an issue. Yes, I know sourcemaps exist for this. Which leads to my next point...

Syntactically the learning curve to CoffeeScript is really nice. But I think the point at which you could truly write it more efficiently than JavaScript takes some investment with the extra tooling needed and the built-in complexity that the Coffeescript -> JavaScript process brings. Maybe it doesn't, but there were times even writing something as relatively simple as FluentCoffee that I was scratching my head a little. Having team members that are invested in really understanding CoffeeScript and tools for it would also really help with this.



