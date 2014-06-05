1. Ensure submitter of pull request is on the [Hydra Project's Contributor License Agreement list](https://wiki.duraspace.org/x/UofvAQ) for Sufia.  If not, ping [Richard Green](mailto:legal@projecthydra.org) and request that this be done.
1. Merge the pull request via [GitHub](https://github.com/projecthydra/sufia/pulls). (Click the (?) icon for instructions.)
1. Pull that code into your locally cloned repo (and run the usual steps after the pull, e.g., `bundle install`, etc.)
1. Run specs via `bundle exec rake` (assuming they pass, continue), or verify specs are passing on the [CI server](http://travis-ci.org/projecthydra/sufia).
1. Bump version number in [SUFIA_VERSION](https://github.com/projecthydra/sufia/blob/master/SUFIA_VERSION)
1. Modify [the changelog](https://github.com/projecthydra/sufia/blob/master/History.md) to include changes in the version (which can usually be pulled from commit messages)
1. Build the gem, commit changes, tag the release, push it up to rubygems.org: `rake all:release`
  * If you happen to get an error "rake aborted! There are files that need to be committed first," you may have stuff cluttering up AF's built-in jetty submodule.  If so, do the following: `cd jetty; git checkout .; git clean -df; cd ..; rake all:release`
  * If this is your first time pushing to rubygems.org, you will be prompted for your rubygem credentials, in which case do the following: `gem push; rake all:release`
1. Create [release notes in GitHub](https://github.com/projecthydra/sufia/releases). In the new release, include at least a block with upgrade notes and a block showing the changelog (copy from earlier step). (See [an example](https://github.com/projecthydra/sufia/releases/tag/v3.6.1).)
1. Send a release message to [hydra-tech](mailto:hydra-tech@googlegroups.com) and [hydra-releases](mailto:hydra-releases@googlegroups.com) describing the changes (which you can copy from the GitHub release).
