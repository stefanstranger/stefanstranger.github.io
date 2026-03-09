# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll", "~> 4.4.0"
gem "jekyll-theme-chirpy", "~> 7.4", ">= 7.4.1"

group :jekyll_plugins do
  gem "github-pages"
  gem "jemoji"
end

gem "html-proofer", "~> 5.0", group: :test

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

