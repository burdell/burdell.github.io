---
title: Building Angular Apps as A Series of Mini-Apps
author: nathan griffin
date: 2016-03-03
type: draft
---

_whoa it's been awhile_

I recently started another job doing more Angular stuff. The cool part about it is that I'm building something completely from scratch which is both awesome and frustrating at the same time.

For a number of reasons, we decided to make our app a series of single-page apps. So switching between different sections of the site would be a full page reload, but browsing inside the different sections of the site would not require full page reloads. However, I still wanted to be able to share directives, services, providers, filters, HTML snippets, etc between all these apps easily without having to do something super annoying.

The obvious solution to this was the use some kind of front-end build system. I had previous experience tooling with Gulp and its code-over-configuration approach seemed better suited to handle the automation of building out seperate Angular apps than its archnemesis Grunt.

### Initial Approach: One App ###
In order to get things going, I wasn't going to worry about building out seperate apps until I had at least one basic app working. 

Here's the basic folder structure I decided to go with:
<pre class="prettyprint">
app/
    /areas
        /userprofile
            /config
                userprofile.module.js
                userprofile.routes.js
            /edit
            	userprofile.edit.js
            	userprofile.edit.html
    /shared
        /directives
        /filters
        /providers
        /services
        ...etc...
    /design
    	.... sundry things to make the site pretty (.scss, images, etc)...
</pre>

The basic idea is that each folder under /areas is a seperate app and /shared is, well, stuff that is shared. Each app also has a /config folder that contains the file to instantiate a specific module for the app, and define its routing.

So then I set up a very basic gulp build. 

Here are the basic tasks for building out the JavaScript files:
<pre class="prettyprint">
	var sources = {
		js: ['app/**/*.js'],
		partials: [
		'app/areas/**/*.html',
        'app/shared/**/*.html',
        'app/layout/**/*.html',
        '!app/index.html'
		]
	};

	//move the app scripts to the dev output directory
	gulp.task('scripts', function() {
	  return gulp.src(sources.js)
	    	.pipe(gulp.dest(outputs.dev));
	});

	//ngHtml2js takes html files and puts them into Angular's $templateCache
	gulp.task('partials', function() {
	    var partialGlob = gulp.src(sources.partials)
	        .pipe($.ngHtml2js({ moduleName: "app.templates" }))
	        .pipe(gulp.dest(outputs.dev + '/templates'));
	});

	//using a gulp library that reads the Bower dependencies and moves them to a vendor folder
	gulp.task('bower', function(){
	    return gulp.src(bowerFiles)
	        .pipe(jsFilter)
	        .pipe(gulp.dest(outputs.dev + '/vendor'))
	        .pipe(jsFilter.restore());
	});
</pre>

Pretty simple. I then used a Gulp tool called gulp-inject that would gulp.src() the moved files and inject script tags into index.html of the app. To handle dependencies, I basically just used naming convention and used this array in that gulp.src() call, which preserves the order:

<pre class="prettyprint">
	var dependencyJs = [
	        outputs.dev + '/areas/**/config/*.js',
	        outputs.dev + '/areas/**/*.js',
	        outputs.dev + '/shared/**/*.module.js',
	        outputs.dev + '/shared/**/*.js',
	        outputs.dev + '/templates/**/*.js',
	        outputs.dev + '/app.init.js'
	    ]
</pre>

So after I added stuff like SASS compilation for our CSS, simple gulp.watch() functions, and stuff to spin up an Express server to serve index.html, that gave me a simple automated build process.

### Part 2: Multiple Apps! ###
Once I had a simple app working with one area, it was time to add multiple apps to the build process. To do this I added a simple function that looped through all the targeted build areas.

This allowed me to re-use my existing build tasks with just having to account for the area name being passed to it.

So if this is my /areas folder:
<pre class="prettyprint">
 /areas
 	/directory
 	/userprofile
 	/forums
 </pre>
 This would be the function to loop through them, with modified build tasks from above

<pre class="prettyprint">
	
		var sources: {
			js: function(areaName) {
				return [ 'app/' + areaName + '/**/*.js']
			},
			partials: function(areaName) {
				return [
					'app/areas/' + areaName + '/**/*.html',
			        'app/shared/**/*.html',
			        'app/layout/**/*.html',
			        '!app/index.html'
				]
			}
		}

		var areas = ['directory', 'userprofile', 'forums'];
		function areaBuilder(taskFn){
		    _.each(areas, function(areaName){
		        taskFn(areaName);
		    });
		}

		//move the app scripts to the dev output directory
		gulp.task('scripts', function() {
			areaBuilder(function(areaName){
				return gulp.src(sources.js(areaName))
		    		.pipe(gulp.dest(outputs.dev));
			});
		});

		//ngHtml2js takes html files and puts them into Angular's $templateCache
		gulp.task('partials', function() {
			areBuilder(function(areaName){
				var partialGlob = gulp.src(sources.partials(areaName))
			        .pipe($.ngHtml2js({ moduleName: "app.templates" }))
			        .pipe(gulp.dest(outputs.dev + '/templates'));
			});
		});	
</pre>
I'm sure there has to be some npm tool to read folder names that would automate this even more, but adding a new section won't be a super common occurence so ehhh. 

This approach allowed me to get the project rolling quickly, but there are of couple of things that are less than ideal about it:

1. **Apps include a bunch of extra files from the /shared folder that aren't needed**

	As we started to develop various sections of the site, the /shared folder became pretty big. All the files from this folder were basically just dumped into an app regardless of whether that app needed it. I thought through a bunch of different ways to handle this. Most of them involved either setting dependencies inside the gulp build or other really convaluted things. Read ahead to point 3 to find out why I didn't like any of these.

2. **This makes concatination and minification way harder than it needs to be**

	Though we didn't need this until much later in the process, it was something I kept in mind. I played around with it a bit, and every solution I came up with just seemed way harder than it needed to be. In my experience, tooling files like this quickly become really bloated because of stuff like this and I really wanted to avoid that.

3. **Handling file dependencies based off of folder and files names is really terrible in the long run**

	Having consistent naming conventions is really important, but using it to build dependencies is super brittle. One derivation from this will cause a bunch of work (and workarounds most likely). Like previously mentioned, stuff like this causes really bloated build files.

	Additionally, your actual files and dependencies are completely seperate, and it's super not apparent what those dependencies are. While previously mentioned inflexibilty of this approach is apparent, this problem is more subtle, but no less important. Anybody who has tried to work with code that was written by somebody else knows how important stuff like this is.

###Part 3: A Solution###
I knew the solution to this 



