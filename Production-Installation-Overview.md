# Steps
1. Install Java 8
1. Install Ruby 2.1 or greater.
1. Install Fcrepo 4.5 or greater.
1. Install Solr 5.4 or greater.
1. Install Redis 2.6 or greater (2.6+ is required for Redlock)
1. Install dependencies: FITS, imagemagick, openoffice, ffmpeg
1. Configure the statefile to be somewhere other than `/tmp`
1. Configure the derivatives and uploads directory to be something other than `tmp`
1. Add an ActiveJob driver such as `resque` or `sidekiq` to the Gemfile.


This was created with inspiration from https://github.com/projecthydra-labs/hydradam/wiki/Production-Installation%3A-Overview



