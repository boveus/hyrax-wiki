1. Ensure submitter of pull request is on the [Hydra Project's Contributor License Agreement list](https://wiki.duraspace.org/x/UofvAQ) for Sufia.  If not, ping [Hydra admin](mailto:legal@projecthydra.org) and request that this be done. (We must be sure that all code contributed to Hydra components is written by CLA signatories.)
1. Merge the pull request via [GitHub](https://github.com/projecthydra/sufia/pulls). (Click the (?) icon for instructions.)
1. Verify that all specs are passing on the [CI server](http://travis-ci.org/projecthydra/sufia).
1. Bump version number in [SUFIA_VERSION](https://github.com/projecthydra/sufia/blob/master/SUFIA_VERSION), lib/sufia/version.rb, and sufia-models/lib/sufia/models/version.rb.
1. Modify [the changelog](https://github.com/projecthydra/sufia/blob/master/History.md) to include changes in the version (which can usually be pulled from commit messages).
1. Modify [the README](https://github.com/projecthydra/sufia/blob/master/README.md) to replace all instances of the past version with the new version.
1. Release the gem to rubygems.org: `rake all:release`
  * If this is your first time pushing to rubygems.org, you will be prompted for your rubygems.org credentials, in which case do the following: `gem push; rake all:release`
1. Create [release notes in GitHub](https://github.com/projecthydra/sufia/releases/new). In the new release, include at least a block with upgrade notes and a block showing the changelog (copy from earlier step). (See [an example](https://github.com/projecthydra/sufia/releases/tag/v6.4.0).)
1. Send a release message to [hydra-tech](mailto:hydra-tech@googlegroups.com), [hydra-community](mailto:hydra-community@googlegroups.com), and [hydra-releases](mailto:hydra-releases@googlegroups.com) describing the changes (which you can copy from the GitHub release).
