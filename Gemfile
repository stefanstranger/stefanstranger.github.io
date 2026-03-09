# frozen_string_literal: true

source "https://rubygems.org"

# Explicitly pin Jekyll to 4.4+ for jekyll-theme-chirpy compatibility
# (avoids conflicts with github-pages gem which pins to Jekyll 3.10)
gem "jekyll", "~> 4.4.0"
gem "jekyll-theme-chirpy", "~> 7.4", ">= 7.4.1"

group :jekyll_plugins do
  gem "jemoji"
end

gem "html-proofer", "~> 5.0", group: :test

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]

