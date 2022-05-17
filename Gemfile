source 'https://rubygems.org'

require 'json'
require 'open-uri'
puts 'Fetching supported version from github.com'
versions = open('https://pages.github.com/versions.json').read
#versions = open('versions.json').read
versions = JSON.parse(versions)
puts versions

gem 'github-pages', versions['github-pages']
gem "liquid-tag-parser", "~> 1.9"
gem 'jekyll-last-modified-at'
#gem 'link-checker'
gem 'jekyll-sitemap'
gem 'pygments.rb'
gem 'nokogiri'
gem 'rmagick'
gem "sprockets", "~> 3.7"
gem "jekyll-assets", '~> 2'
gem 'jekyll-pdf', git: 'https://github.com/jekyll-pdf/jekyll-pdf'
