---
title: Building an Angular App as A Series of Mini-Apps
author: nathan griffin
date: 2016-03-03
---

I started a new job this year building out an Angular app from scratch.

For a number of reasons, we decided to make our app a series of single-page apps. So switching between different sections of the site would be a full page reload, but browsing inside the different sections of the site would not require full page reloads.

However, I still wanted to be able to share directives, services, providers, filters, HTML snippets, etc between all these apps easily without having to do something super annoying. So I decided to build the seperate apps inside of our Gulp build.

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

The basic idea is that each folder under /areas is a seperate app and /shared is, well, stuff that is shared. Each app also has a /config folder that contains a file to instantiate a specific module for the app, and a file to define its routing.

I set up a basic Gulp.js build process for creating one app just to get something going:
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

Pretty simple. I then used a Gulp tool called gulp-inject that would gulp.src() the moved files and inject script tags into index.html of the app. To handle dependencies, I basically just used a naming convention and used this array in that gulp.src() call, which preserves the order:

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
Once I had the project working with one area, it was time to add multiple apps to the build process. To do this I added a simple function that looped through all the targeted build areas.

This allowed me to re-use my existing build tasks with just having to account for the area name being passed to it.

So if this is my /areas folder:
<pre class="prettyprint">
 /areas
 	/directory
 	/userprofile
 	/forums
 </pre>

 This would be the function to loop through them, with modified build tasks from above:
<pre class="prettyprint">
		
		var areas = ['directory', 'userprofile', 'forums'];
		function areaBuilder(taskFn){
		    _.each(areas, function(areaName){
		        taskFn(areaName);
		    });
		}

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
I'm sure there has to be some npm tool to read folder names that would automate this even more, but adding a new section won't be a super common occurence and doing it this ways gives you a way to 'hide' sections for whatever reasion.

This approach allowed me to get the project rolling quickly, but there are of couple of things that are less than ideal about it:

1. **Apps include a bunch of extra files from the /shared folder that aren't needed**

	As we started to develop various sections of the site, the /shared folder became pretty big. All the files from this folder were basically just dumped into an app regardless of whether that app needed it. I thought through a bunch of different ways to handle this. Most of them involved either setting dependencies inside the gulp build or other really convaluted things.

2. **This makes concatenation and minification way harder than it needs to be**

	I played around with trying to concatenate and minify the scripts a bit, and every solution I came up with just seemed way harder than it needed to be. Tooling files like this quickly become bloated because of stuff like this and I really wanted to avoid that.

3. **Handling file dependencies based off of folder and files names is really terrible in the long run**

	Having consistent naming conventions is really important, but using it to build dependencies is super brittle. One derivation from this will cause a bunch of work (and workarounds most likely), causing more bloat.

	Additionally, your actual files and dependencies are completely seperate. The makes it super not apparent what those dependencies are.

### Part 3: A Solution ###
The solution to this was to start using Browserify, which I knew about but had never actually used. Browserify lets you specify depencies inside of your JS files using Node-style require() statements which makes dependencies very clear and modular, and outputs your code into a single file so concatenation and minification become trivial. It also allowed me to completely take Bower out of the project, which is just one less thing to have to worry about.

Browserify gulp tasks:
<pre class="prettyprint">

	gulp.task('scripts', function() {
	    browserifyHelper();
	});

	gulp.task('prod-scripts', function(){
	    browserifyHelper(true);
	});

	function browserifyHelper(prodBuild) {
	    areaBuilder(function(areaName){
	        var b = browserify({ 
	            paths: ['app/', 'node_modules', 'app/shared/', 'app/areas/'],
	            fullPaths: true,
	            cache: {},
	            packageCache: {}
	        });

	        if (!prodBuild) {
	            b = watchify(b);
	            b.on('update', function(changedFilename){
	                bundleHelper(false, b, areaName);
	            });        
	        }
	        b.add('app/areas/' + areaName + '/init.js');
	        bundleHelper(prodBuild, b, areaName);
	    });
	}

	function bundleHelper(prodBuild, b, areaName){
	    
	    if (areaName) {
	        return bundle(areaName);
	    } else {
	        areaBuilder(function(areaName) {
	            return bundle(areaName);
	        })
	    }

	    function bundle(areaName) {
	        var bundleBlob = b.bundle();

            if (!prodBuild) {
                bundleBlob =  bundleBlob.on('error', function(err) {
                    return $.notify().write(err);
                });
            }

            bundleBlob = bundleBlob.pipe(source(jsAppFileName))
                .pipe(buffer());

	        if (!prodBuild) { 
	            bundleBlob = bundleBlob
	                .pipe($.sourcemaps.init({ loadMaps: true }))
	                .pipe($.sourcemaps.write('./maps'))
	        } else {
	            bundleBlob = bundleBlob
	                .pipe($.uglify());
	        }
	        
	        bundleBlob
	            .pipe(gulp.dest(areaPath(areaName, prodBuild) + '/js/'))
	    }
	}

</pre>
Each app would have an entry point file of init.js which would require() all it's individual pages (who would then define any component/services/etc it needed).

So if an area had this file structure...
<pre class="prettyprint">
    /forums
        /config
            forums.routes.js
        /list
        	forums.list.js
        	forums.list.html
        /detail
        	forums.detail.js
        	forums.detail.html
        forumsContainer.html
        forumsContainer.js
        init.js
</pre>
...its init.js would looks like this:

<pre class="prettyprint">

	'use strict';

	var app = require('app.init.js');
	app('forums');

	require('forums/forumsContainer.js');
	require('forums/list/forums.list.js');
	require('forums/message/forums.detail.js');
	require('forums/config/forums.routes.js');

	... other stuff...
</pre>
Notice that {areaName}.module.js not longer exists. Another advantage Browserify gave me was the ability to write instantiating the app's module generically. This allowed sitewide options/behavior and require() statements (stuff like permissions, navigation bar, footer, other stuff that is on every single page) to exist in one place. Here are the basics of app.init.js:

<pre class="prettyprint">

	//vendor
	var angular = require('angular');

	function initializeApp(areaName) {
	    var moduleBuilder = require('modulebuilder');
	    angular.module('communityApp', [
	        'ui.router',
	        'ngSanitize',
	        'community.templates',
	        moduleBuilder('community.providers'),
	        moduleBuilder('community.services'),
	        moduleBuilder('community.directives'),
	        moduleBuilder('community.filters'),
	        moduleBuilder('community.' + areaName)
	    ])    
	    .config(['$httpProvider', function($httpProvider){
	        $httpProvider.interceptors.push(require('services/permissionsLoader.js'));
	    }]);

	    require('services/permissionsLoader.js');
	    
	    require('directives/mainnavbar/mainnavbar.js');
	    require('directives/breadcrumbs/breadcrumbs.js');
	    require('directives/discussionsnavbar/discussionsnavbar.js');
	    require('directives/megamenu/megamenu.js');
	    require('directives/pageheader/pageheader.js');
	    require('directives/announcement/announcement.js');
	    require('directives/pagescroll/pagescroll.js');
	    require('directives/permissions/permissions.js');
	    require('directives/error/error.js');
	    
	    require('basecontroller.js');

	    angular.element(document).ready(function(){
	        angular.bootstrap(document, ['communityApp']);
	    });
	}

	module.exports = initializeApp;

</pre>
So, that's the basic idea. 

It's not a perfect solution and I'm sure there are things that could be improved. One thing I've noticed is that looping through the area likes that with Browserify is relatively slow. The looping also breaks the promise'y nature of how I've seen most peoplep write gulp tasks, meaning that I guess it's possible that a task could start prematurely, but I haven't really run into that scenario.















