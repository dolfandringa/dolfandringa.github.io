source 'https://rubygems.org'

require 'json'
require 'open-uri'
versions = '{"jekyll":"3.0.3","jekyll-sass-converter":"1.3.0","jekyll-textile-converter":"0.1.0","kramdown":"1.10.0","rdiscount":"2.1.8","redcarpet":"3.3.3","RedCloth":"4.2.9","liquid":"3.0.6","rouge":"1.10.1","github-pages-health-check":"1.1.0","jemoji":"0.6.2","jekyll-mentions":"1.1.2","jekyll-redirect-from":"0.10.0","jekyll-sitemap":"0.10.0","jekyll-feed":"0.4.0","jekyll-gist":"1.4.0","jekyll-paginate":"1.1.0","jekyll-coffeescript":"1.0.1","jekyll-seo-tag":"1.3.2","jekyll-github-metadata":"1.10.0","ruby":"2.1.7","github-pages":"66","html-pipeline":"2.3.0","sass":"3.4.21","safe_yaml":"1.0.4"}'
#puts 'Fetching supported version from github.com'
#versions = open('https://pages.github.com/versions.json').read
versions = JSON.parse(versions)

gem 'github-pages', versions['github-pages']
gem 'jekyll-last-modified-at'
gem 'link-checker'
gem 'jekyll-sitemap'
gem 'pygments.rb'
