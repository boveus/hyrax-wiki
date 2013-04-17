1. Ensure submitter of pull request is on the [Hydra Project's Contributor License Agreement list](https://wiki.duraspace.org/x/UofvAQ) for Sufia.  If not, ping [Richard Green](mailto:r.green@hull.ac.uk) and request that this be done.
1. Merge the pull request via [GitHub](https://github.com/curationexperts/sufia/pulls). (Click the (?) icon for instructions.)
1. Pull that code into your locally cloned repo (and run the usual steps after the pull, e.g., `source .rvmrc`, `bundle install`, etc.)
1. Run specs via `bundle exec rake` (assuming they pass, continue), or verify specs are passing on the [CI server](http://travis-ci.org/curationexperts/sufia).
1. Bump version number in [lib/sufia/version.rb](https://github.com/curationexperts/sufia/blob/master/lib/sufia/version.rb)
1. Modify [release notes](https://github.com/curationexperts/sufia/blob/master/History.md) to include changes in the version (which can usually be culled from commit messages)
1. Commit and push your local changes (just textual, so another round of spec-checking should not be necessary)
1. Build the gem: `rake build`
1. Push the gem to rubygems.org, tag the release, push the finished tagged release to master: `rake release`
  * If you happen to get an error "rake aborted! There are files that need to be committed first," you may have stuff cluttering up AF's built-in jetty submodule.  If so, do the following: `cd jetty; git checkout .; git clean -df; cd ..; rake release`
  * If this is your first time pushing to rubygems.org, you will be prompted for your rubygem credentials, in which case do the following: `gem push; rake release`
1. Send a release message to [hydra-tech](mailto:hydra-tech@googlegroups.com) and [hydra-releases](mailto:hydra-releases@googlegroups.com) describing the changes (which you can copy from the release notes)
