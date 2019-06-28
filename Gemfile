source 'https://rubygems.org'

require 'json'
require 'open-uri'
puts 'Fetching supported version from github.com'
versions = open('https://pages.github.com/versions.json').read
versions = JSON.parse(versions)
puts versions

gem 'github-pages', versions['github-pages']
gem 'jekyll-last-modified-at'
#gem 'link-checker'
gem 'jekyll-sitemap'
gem 'pygments.rb'
gem 'nokogiri'
gem 'rmagick'
gem 'jekyll-assets', '>=2.0'
gem 'jekyll-pdf'
