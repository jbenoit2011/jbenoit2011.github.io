---
layout: post
title:  "How to automate Ripple refreshing on changes saved?"
date:   2014-09-23
tags: ripple cordova workflow
---

Well, you are developing a Cordova-based hybrid app, [Gruntjs](http://gruntjs.com/) is a part of your workflow, and you are visualizing the result in your favorite browser through Ripple Emulator. Your source code is in a folder and your build in another.

But you faced a problem: Each time you modify your code source, you must trigger manually the building of your code (via a `cordova prepare`) and you have to refresh Ripple tab to visualize the new render.

Let me told you this time is over. How ? By automating the workflow. We will end up with something like this:

![cordova-development-workflow]
(http://i.imgur.com/CaXYYFE.png)

##Pre-requisites

First of all, you will need some packages:

- [RVM](http://rvm.io/): Install it with `curl -sSL https://get.rvm.io | bash -s stable --ruby`
- Ruby: Once you've installed RVM, run `rvm list` to list the current Ruby environnements installed. Enable the last one with `rvm use <ruby_version>`. e.g: `ruby use 2.1.3`.
- Rubygems: Install it with `rvm rubygems latest`
- Chrome browser (Not tested but should also work with Firefox or Safari)

##Installation

Let's continue! There will be 3 steps:

1. Install and configure grunt-watch
2. Install and configure guard-livereload
3. Install and configure livereload Chrome Extension

###Grunt-watch

__What is Grunt-watch?__

Grunt-watch is a plugin of Grunt which allow to watch modifications on specified files and trigger a task on these events.

1. Install it: `npm install grunt-contrib-watch --save-dev`
2. Add `grunt.loadNpmTasks('grunt-contrib-watch');` to your `Gruntfile`
3. Configure the tasks, by specifying the files to watch.

	e.g:

	```javascript
	watch: {
	  all: {
	    files: ['src/index.html', 'src/**/*.css', 'src/**/*.js'],
	    tasks: ['build'],
	    options: {
	      spawn: false,
	      interrupt: true,
	      event: ['added', 'changed']
	    }
	  }
	}
	```

	This tasks will watch continuously changes on files every CSS, JS files and index.html.

	[Spawn option](https://github.com/gruntjs/grunt-contrib-watch#optionsspawn) set to false will speed up tasks execution time but can also make tasks suceptible to failed.

	[Interrupt option](https://github.com/gruntjs/grunt-contrib-watch#optionsinterrupt) set to true will allow grunt-watch to stop the previous tasks if a new event happen. Which can be the case if you do a lot of `ctrl+s` like me :)

	[Event option](https://github.com/gruntjs/grunt-contrib-watch#optionsevent) will trigger tasks only if a file is added or modified. This way task triggering is limited, and it should not eat all of your CPU and RAM.

4. Run Grunt-watch task: `grunt watch`

###Guard-livereload
__What is Guard-livereload?__

Guard-livereload is a plugin of [Guard](https://github.com/guard/guard) a tool which allow to trigger tasks on file system modifications.
This plugin add the ability to communicate with livereload browser extension

1. Install it with `gem install guard-livereload`
2. Add this to your Gemfile (maybe you need to create it):

	```ruby
	group :development do
	  gem 'guard-livereload', require: false
	end
	```

3. Add guard definition for this plugin to your Guardfile : `guard init livereload`
4. Configure the tasks into Guardfile:

```
guard 'livereload' do
  watch(%r{www/.+})
end
```

Here we just need to watch the entire content of the www folder.
5. Run `guard start` to launch the guard watching process


###Livereload Chrome extension

1. [Install the extension](https://chrome.google.com/webstore/detail/livereload/jnihajbhpnppcggbcgedagnkighmdlei). A new icon should be added next to your address bar.
If you use Firefox or Safari, please refer to this [page](http://feedback.livereload.com/knowledgebase/articles/86242-how-do-i-install-and-use-the-browser-extensions-)
2. Switch to your Ripple emulator tab and click on the livereload icon to enable it.

##Test

Finally test your new workflow:

1. Open a file watched by Grunt-watch, modify it and save it.
2. The Ripple tab should be refreshed in around 3seconds (depending on your Grunt-watch triggered tasks total execution time).
