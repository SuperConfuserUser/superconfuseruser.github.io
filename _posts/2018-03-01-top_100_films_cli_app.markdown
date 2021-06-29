---
layout: post
title:      "Top 100 Films CLI App"
date:       2018-03-01 14:52:43 -0500
permalink:  top_100_films_cli_app
---


The very first portfolio project for Flatiron students is an application with a Command Line Interface (CLI) that uses data from an external source.  It a great final project testing your understanding of all the previous lessons.

The objectives are to implement the following:
* Project setup
* GitHub
* Object Oriented Programming
* CLI
* Scraping


## The Hurdles
This will be the first time we're asked to build an entire project from scratch.  I've been spoiled by the lessons which are all nicely setup and ready for code. The Learn IDE automates and simplifies a lot of tasks, so I don't have to worry too much about connecting to GitHub or dependencies. 

Project setup is daunting when you're just staring at an empty temp directory. Luckily, it's [Bundler](http://bundler.io) to the rescue. Creating a project directory with structure and support files is as easy as `bundle gem <GEM_NAME>`. Dependencies will also be easier to load with `bundle install`.

GitHub was another big trouble spot. There were lessons about Git basics, but they were very early on and didn't stick since we didn't have to use the commands.  Re-reading those helped. Reading other [sources](https://product.hubspot.com/blog/git-and-github-tutorial-for-beginners) helped. Using it constantly helped a lot. Now, I can clone remote repos and push local repos. Success! 

NOW, we can code!


## The Project

![](https://lh3.googleusercontent.com/E8J2W1jcNmEIs2jRO2o-wjDtZdqDV_g6kWyDjiGhk5uPMTtwk1qFwkkAaT9bY9g8cdrSJVX8LfFZwq5VblaWA9KvLaJ2UCmeLQZPlp0ac6MxjBs0EgF-i0fozJVI_4_ZU0pgsfV2Qg=w2400)

My pop culture knowledge is pretty weak pre-1999. Friends are horrified to find out that I haven't seen classics such as Pulp Fiction (it's on my Netflix queue).  So this project will help with adding more films to the queue and practicing coding skills.

Empire online has a nice [list](https://www.empireonline.com/movies/features/best-movies/) that was crowdsourced. Most are American. Some are very recent and popular and may not stand the test of time. The genre leans more towards action. There's a sprinkling of classics, and all seem like very solid films. 

The list is a good start, but it doesn't provide all the data I want for a film. For that, [IMDB](http://www.imdb.com) is king.  Wikipedia doesn't have as much multimedia, and Rotten Tomatoes doesn't have anything for more obscure films. 


## The Bits
Object Oriented programming requires thinking about specialization and responsibilities.  The film class holds information about the films. The CLI class interacts with the user. The scraper class retrieves the data.  Each method for each class is in charge of a specific type of action.

I embraced the retro goodness of CLI by adding an ASCII art title screen generated at [Patorjk.com](http://patorjk.com/software/taag/#p=display&f=Graffiti&t=Type%20Something%20). The [Colorize gem](https://github.com/fazibear/colorize) was also used to change text color for special outputs.

Gathering the data requires [Nokogiri](http://www.nokogiri.org) and [OpenURI](http://ruby-doc.org/stdlib-2.1.3/libdoc/open-uri/rdoc/OpenURI.html).  It can take a lot of tries to scrape down to the specific piece of information you need. The film title and year from the Empire list were used to generate a search link in IMDB that leads to the final page with film details.  


## The Conclusion
This gem is still a work in process and will go into review soon. You can take a peek at it on [GitHub](https://github.com/unenlightened/top-100-films-cli-app).

It was fun building something from the ground up and figuring out how to get all the bits and pieces working together.  Comments and critiques would be appreciated. They help the improvement process. Any movie recommendations would be welcome too.




(Confession: I haven't seen #1 - The Godfather.)




