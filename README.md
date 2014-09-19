---
  tags: scraping, nokogiri, tutorial, kids
  languages: ruby, html, css
  resources: 2
  level: intermediate
---

# Learn to Scrape!

## Intro

Welcome to Web Scraping! We're glad you're here! Web scraping is the amazing process of pulling data from a website to use in your own program. Whether it's pulling stocks from yahoo finance, news headlines from the new york times, items for sale from craigslist, tweets from your favorite celebrity, or hilarious subreddit titles, web scraping has got you covered.

In this short tutorial, we'll be learning the basics of using the [Nokogiri](http://nokogiri.org/) gem by scraping a small portion of a website about Financial District dining options (made by a Flatiron HS student!). Here's the site (open it now in a new tab):

<a href="https://s3-us-west-2.amazonaws.com/nokogiri-scrape/index.html" target="_blank">Fidi Dining</a>

First, we'll learn how to make an http request using Ruby's [Open-URI](http://ruby-doc.org/stdlib-2.1.0/libdoc/open-uri/rdoc/OpenURI.html) module. Then, we'll learn how to convert that response into a `Nokogiri::HTML::Document` object. We'll then use the `css` method to collect the data we're interested in, and store it into a data structure of our choosing.

##Video Tutorial

<iframe src="//player.vimeo.com/video/106618253" width="500" height="375" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>

## Getting Started

The first thing we need to do is set up our project. Normally, we'd want to test drive the development of this program, but for the purposes of this guide, we'll skip that. We're going to be coding our solutions in a file called fidi_scraper.rb that you can create in your development directory for practice.

To be able to use either Nokogiri or Open-URI, we're going to need to make sure we require them both at the top of our file. So on the first two lines of `fidi_scraper.rb`, add the following:

```ruby
require 'nokogiri'
require 'open-uri'
```

(If you haven't already done so, now would be the time to make sure you install Nokogiri by running `gem install nokogiri` from your command line.)

### What is Open-URI?

Open-URI is a module in Ruby that allows us to programatically make http requests. It gives us a bunch of useful methods to make different types of requests, but for this guide, we're interested in only one: `open`. This method takes one argument, a url, and will return to us the HTML content of that url.

In other words, running:

```ruby
fidi_html = open('https://s3-us-west-2.amazonaws.com/nokogiri-scrape/index.html')
```

stores the HTML of our site into a variable called `fidi_html` (You can check this by `puts`-ing the `fidi_html` variable and running the program running `ruby fidi_scraper.rb`) 

### And Nokogiri? What's that?

From the [Nokogiri](http://nokogiri.org/) website:

> Nokogiri (鋸) is an HTML, XML, SAX, and Reader parser. Among Nokogiri’s many features is the ability to search documents via XPath or CSS3 selectors.

Essentially, Nokogiri allows us to treat a huge string of HTML as if it were a bunch of nested data structures. This means that we can access any HTML element on a given page via handy dot-notation. We can do all of this without any ugly regular expressions, all via CSS selectors. It's amazing!

##Nokogiri to the rescue!

Don't worry about this syntax too much now, but the Nokogiri gem gives us this cool method, `Nokogiri::HTML` that takes an HTML string and converts it into this giant NodeSet (aka, a bunch of nested "nodes") that we can easily play around with.

Let's use that `html` variable again and pass it to the `Nokogiri::HTML` classes and see what happens:

```
fidi_nokogiri = Nokogiri::HTML(fidi_html)
```

We can test that a new Nokogiri object was made by putsing fidi_nokogiri. You should see a bunch of output, the top of which looks something like:

```
#<Nokogiri::HTML::Document:0x811468ac name="document" children=[#<Nokogiri::XML::DTD:0x8114635c name="html">, #<Nokogiri::XML::Element:0x811460f0 name="html" attributes=[#<Nokogiri::XML::Attr:0x8114608c name="itemscope">...
```

This returns to us a giant object that consists of nested "nodes" (nested arrays and hashes) that we can drill down into using CSS selectors. Let's see if we can do something useful with it.

In your browser, visit `https://s3-us-west-2.amazonaws.com/nokogiri-scrape/index.html`. Let's see if we can use this Nokogiri object to store the text from the title into a variable. Not terribly exciting, but useful to demonstrate how Nokogiri gets stuff done.

Hopefully you're using Chrome. If you are, right click on the 'Fidi Dining' title, and select 'Inspect Element'. You should see something like:

```html
<h1>Fidi Dining</h1>
```

If you don't see it immediately, try clicking on the triangle next to the body tag and then the triangle next to the div with the class "header." What this tells us is that the page header is in an `<h1>` element.

We want to pull out this data using Nokogiri. First thing we need to do is create a variable called `title`. We're going to call a method on the fidi_nokogiri object called `css` that accepts CSS selectors and returns a specific piece of the object based on those selectors.

```ruby
title = fidi_nokogiri.css(“h1”)
```

If we `puts title`, you'll notice that it returns the html for the title, including the h1 tags. If we only want the text, we can call the text method.

```ruby
title = fidi_nokogiri.css(“h1”).text
```

Great! Now let's find the restaurant name for Sophie's Cuban Cuisine by doing the same thing.

Right click on Sophie's Cuban Cuisine, and choose 'inspect element.' You can see that the information on Sophie's is in a `div` with a class of "restaurant" and another class "r3". If we were to write this in css, we'd say:

`div.restaurant.r3`

Chrome Web tools actually makes this very easy for you. If you select the node you want and look at the bottom of the web tools window, the css path is given to you:

<img src="https://s3-us-west-2.amazonaws.com/nokogiri-scrape/images/chrome-css-info.png">

Anyway, we can use the CSS selector given to us to drill through the Nokogiri object and grab just that text.

## Using the css method.

Now's the time for the oh-wow-mind-blown part of this. We want to programmatically get the Sophie's Cuban Cuisine name using our CSS selector and Nokogiri. We do this by calling the CSS method with our CSS selector as the argument:

```ruby
sophies = fidi_nokogiri.css("div.restaurant.r3")
```

But, that selector isn't quite specific enough. If you notice, the thing we actually want is within an `<h2>` tag inside of the `restaurant` div. Just like we would do in a CSS stylesheet, we just add that `h2` on to the end of our selector. There is a space between r3 and h2 because the h2 is a child of the r3 div.

```ruby
sophies = fidi_nokogiri.css("div.restaurant.r3 h2")
```

Hit return and you should get the following:

```
[#<Nokogiri::XML::Element:0x3ff2b902015c name="h2" children=[#<Nokogiri::XML::Text:0x3ff2b90216ec "Sophie's Cuban Cuisine">]>] 
```

Now, we're almost there, but this doesn't quite do it yet. If you notice, this is actually a part of the Nokogiri object. To convert it into text we use the text method:

```ruby
sophies = fidi_nokogiri.css("div.restaurant.r3 h2").text
```

Hit return, and you get:

```
"Sophie's Cuban Cuisine"
```

KABOOM! Test this out by adding the line `puts sophies` to your script. Just think about it, you can pull out the phone number by changing the h2 part of the CSS selector to h3 because that's how the phone number is tagged. You can grab any information you want by adjusting the CSS selector.

## Navigating through a list
If we use 'inspect element' on the Fidi Dining page, we can see that there are a bunch of divs that have the same 'restaurant' class. If we run

```ruby
fidi_nokogiri.css("div.restaurant")
```
You'll get a long list bounded by [ and ] and separated by commas. If this looks familiar, it's because it is! You've been given an array! Let's say we want to get the names of ALL the restaurants. We'll create a variable called `restaurants_names` and set it equal to the line above. If we `puts restaurants_names`, we get way too much information. We just want the names. How do we do that? Remember that names are inside the h2 tag, which is a child of each restaurant div. So let's add that h2 to our CSS selector. Let's also call the text method to just grab the text.

```ruby
restaurants_names = fidi_nokogiri.css("div.restaurant h2").text
```

Great. We're getting the names we want, but they're pretty ugly looking. Why don't we try iterating through the array using the collect method and converting each name individually to text so that we get a new array of restaurant names back. 

```ruby
restaurants_names = fidi_nokogiri.css("div.restaurant h2").collect do |name|
  name.text
end
```

That looks better. You're a Nokogiri scraping ninja now!

## Nokogiri Scraping Part 1

Now it's time for you to scrape a few pieces of the site on your own. Here are your challenges:

### Easy
* Scrape the title "Fidi Dining"
* Scrape the name "Open Kitchen"
* Scrape the telephone number for GoGo Grill.
* Scrape the review for TGI fridays.

### Medium

* Scrape the page title (that appears in the browser tab).
* Scrape the number of stars that Justino's Pizzeria received.
* Scrape the source link to the photo for Flavors.

### Hard (You'll need to use iteration)

* Scrape and print a string for each restaurant showing their name and star rating.
* Scrape and print the name and phone numbers of restaurants with three or more stars.
* Scrape the name and review for each restaurant that has a phone number starting in (212).

## Scraping Part 2: Choose your own site.

Choose your own site to scrape! Give yourself a challenge! (Talk to a teacher if you need some help brainstorming!)

## Resources
* [Codecademy](http://www.codecademy.com/dashboard) - [Ruby Track: Data Structures](http://www.codecademy.com/courses/ruby-beginner-en-F3loB/0/1)
* [RailsCasts](http://railscasts.com/) - [#190 Screen Scraping with Nokogiri](http://railscasts.com/episodes/190-screen-scraping-with-nokogiri)
