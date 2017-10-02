---
layout: post
title: "A Text Editor"
categories: programming
---

Lately I've been playing around with the idea of creating a text editor. Here's a very quick and basic overview of what I think it should contain. The text editor has extensions, managers, projects, and a core API.  

## The Editor has Extensions

For example, a user wants the editor (TE) to integrate with pandoc in some way. Or, they may want a spell checker. The user could write an extension, which behaves as additional behaviour of the API. It expands the types of calls that can be made to the API.  

## The Editor has Managers

The core API makes no assumptions about how the user would like to display their content. A manager must be written for every display method (the web, desktop, etc).  

## The Editor has Projects

Projects are introduced as an abstract type: a project may be anything. For example, in the case of a Jekyll project, there is a root directory (the website), some configuration information, certain commands that must be run to generate and manage the site, and so on. These can be defined in a Project, eg:  

  
    {% highlight python %}
    class Project:
        def __init__(self, ..., ...):
	...
		
    class Jekyll(Project):
	def __init__(self, root_dir, ruby_dir, ...):
	...
		
	def create_post(self, ...):
	    title = "%s-%s-%s-%s.%s".format(year, month, day, title, file_type)
	    ...
    {% endhighlight %}
	
	
A project can be less complicated. They can be an analogue to major modes in emacs; there may be a markdown project. In this case, the markdown extension might be loaded, and the pandoc extension could be available to convert from markdown to PDF.


## The Editor has an API

The API must supply certain basic functionality. A buffer can be created, edited, and saved. Furthermore, the buffer is available to extensions. Managers and Projects have no direct access to the buffer, they must query the API.

## How is The Editor different from Emacs?

I feel The Editor fills a role Emacs does not. Extensible with Python rather than Elisp, project management (multiple directories) with different major modes, more web-friendly API. Emacs of course has all of these things in theory, but not without a lot of customization that I wish to mitigate. 

## How is The Editor different from Atom?

I haven't used atom in some time. However, I ran into similar problems of extensibility and nodejs, and project management.

## You're Wrong; My Favourite Editor is Better

There is nothing wrong with Atom or Sublime, and I love Emacs. All of those editors are hugely customizable, expertly built, and loaded with features. My main motivation is not to disrupt the editor habitat. I wanted to work on a project, and this seemed like a cool thing to try out. We'll see how it goes. Wish me luck!
