---
title: "A Simple Jekyll Post Manager"
layout: post
published: false
---

A few months ago, I stopped using Django + Mezzanine to manage my site. Now, I use Jekyll instead, and it's a terrific experience. Everything's simple, fast, and easy. I love it. That being said, I'm not happy with one thing in particular. Managing posts is a pain, and it becomes more difficult as the number of posts increase. And I think I know why: the posts are sorted by date. For humans -- or for me, anyway -- it's easier to remember posts by name. When we are trying to find a post, we hardly ever think "that post I wrote on November 22nd, 2017." We think, "that post called "How To Solve It.""

We can use the default commands found in the Rakefile, eg `rake post['Hey! New Post!']`. However, I find that syntax a bit strange. Plus, it's very simple; it can't do much other than open a new post. So, I spent an hour writing *another* **very** simple, quick-and-dirty tool to manage my posts for me. There are two parts: a database file, and the posts in `_posts`. The database file serves only as a bijective (one-to-one) mapping between human friendly names, and Jekyll compliant names. A sample entry would look like this:

    # .projekts

	A Simple Test
	2017-11-30-A-Simple-Test.markdown
	
This file is supposed to be human friendly as well, so the user can make manual changes it necessary. For every post the pattern is simply: human name, machine name, new line.

Right now, the tool doesn't do much. It basically does exactly what the rake task does. There's only one command, `post`. When `post` is called, it will treat everything after it as the title of a new post. If the post is already in the database, it will open that post in emacs. If the post doesn't exist, it will create the new file with some default front matter, add the file to the database, and open the new file in emacs. 

Many parameters are hard-coded. For example, Jekyll supports other formats than markdown. Additionally, I'm not sure how characters like ":", "?", "&" etc are treated by Jekyll when present in file names. The docs aren't clear, and I haven't tested it. Also, the user should have the ability to specify the front matter they want, which database file to use, where the `_posts` are located, etc. And of course, the editor should be easy to change. All this information could be passed as arguments, or defined in a configuration file. 

More importantly, this still doesn't solve my issue of looking up posts. To tackle that, I will write a command called `edit`, which lists all the posts in `_posts` by human friendly name and a number, like this:

    $ ./jpm list
	
	(1) These Are Some Posts          (Nov. 11, 2017)
	(2) Listed in Mapping Order       (Nov. 16, 2017)
	(3) Since the Map isn't Sorted    (Nov. 24, 2017)
	(4) Maybe In the Future           (Nov. 30, 2017)
	(5) I Will Write a Utility        (Dec.  2, 2017) 
	(6) To Periodically Sort Posts    (Dec. 30, 2017)
	(7) Based on A User-Defined       (Jan.  2, 2018)

The user enters the number of the post they want to edit, and the tool will open the post in your favourite editor. This may get tedious for more than 20 or so posts; at that point, I should consider implementing some sort of tab-completed query. Another command -- along the lines of `edit` -- is `list`. It is essentially the same command, but does not wait for any input from the user. It simply lists all the posts in the database.

Finally, we need command called `init`, to bootstrap a database for existing websites. This would simply look at every file in `_posts` with a valid filename, and add it to the database. In order for this to work properly, I would need to write a tool to parse the YAML front matter first, so that the proper title is used. This would allow a potentially more powerful tool: we could list other information with posts, like whether they are published or not. 

Anyway, the basic functionally works for me. I used it to write this post. In the coming weeks, I'll continue to make some tweaks to make it more workable for a greater number of users. My rough list of things to fix, in order of importance, is this:

1. Convert copied strings to formatted strings
2. Consider changing `post` syntax to require titles be enclosed in quotations
3. Write the `edit` command 
4. Derive `list` from `edit`
5. Parse YAML
6. Convert relative paths to user-defined paths (with defaults)
7. Add configuration file ability
8. Support arbitrary editors

The minimal tool is [available on github](https://github.com/avery-laird/jpm). It should compile with a simple: 

    $ gcc jpm.c -o jpm

Just place it in `_posts`, and try it out. Like I mentioned, it only does one thing right now. Hopefully I can slowly add functionality in the coming weeks. I have become increasingly busy lately, so if anyone would like to contribute, please do. Just make a pull request, and I'll take a look as soon as I can. Or, if you just have suggestions, leave a comment, [email me](mailto:laird.avery@gmail.com), or open an issue on github.


