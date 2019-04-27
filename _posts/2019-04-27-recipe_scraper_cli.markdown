---
layout: post
title:      "Recipe Scraper CLI"
date:       2019-04-27 16:41:45 -0400
permalink:  recipe_scraper_cli
---

## Intro
I enjoy doing a bit of cooking and really liked a few of the recipes from [this website](https://cooking.nytimes.com/). After learning how to scrape websites with Ruby, I was eager to try it out for myself to extract some data that I had an interest in.

This recipe site only allowed free access the recipes for 30 days and wanted you to subscribe in order to retain access. I figured it would be nice if I could store all the recipes somehow even if I didn't have an active subscription just because I like trying things for free. So I built a [recipe scraping command line interface](https://github.com/darrendaz/recipe-scrape-cli) as a first step in collecting the recipes for my own collection of favorite recipes.

The program is written in Ruby and uses Nokogiri and OpenURI to accomplish scraping the data. First, I set up the cli.rb file. Then  I set up my scraper.rb file to do the scrape whenever my cli tells it to do so. Throughout the process fo making this program was constantly going back and forth between all three of my files to make sure everything was running correctly, but using `binding.pry` definitely made my life easier here.

Ruby object to receive and store all the data in memory from the scrape

I thought the best place for me to start scraping would be the [Search](https://cooking.nytimes.com/search) page since it was essentially a grid of all the recipes published. BUT then I got hit with my first challenge.

> I have to figure out a way to scrape every page of the serach results because they only display 48 items at a time and there are 19,000+ recipes 😱

## My Solution:
```
    def self.scrape
        totalPages = 4
        pageNum = 1
        
        while pageNum <= totalPages do
            puts "scraping (#{pageNum} / #{totalPages})"
            target = SRC + "/search?q=&page=" + "#{pageNum}"
            html = open(target)
            doc = Nokogiri::HTML(html)
```

Through manipulation of the SRC url, I figured out a way to scrape 1 or all of the pages with a single change to the `totalPages` variable. I thought I was a badass when I figured this out.
