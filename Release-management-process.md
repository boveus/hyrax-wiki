1. Does the release require QA, usability testing, or an accessibility audit? Schedule that to happen before cutting the final release if needed. Lean on the community to help with QA; on the UXIG to help with usability testing; and on [Colin Fulton](https://github.com/justcolin) to help with targeted accessibility audits. Ping [Mike Giarlo](https://github.com/mjgiarlo) (serving as product owner for Hyrax) for assistance with coordination.
1. Update all available translations ([needs documentation](https://github.com/samvera/hyrax/issues/1276) -- this is a new step. Ask [Mike Giarlo](https://github.com/mjgiarlo) if you need assistance.)
1. Bump the version number in [version.rb](https://github.com/samvera/hyrax/blob/master/lib/hyrax/version.rb) to a [SemVer](http://semver.org/)-appropriate version; feel free to consult others (via Slack, hydra-tech, or a Hydra Tech call) if you're not sure what version to choose.
1. Modify [the README](https://github.com/samvera/hyrax/blob/master/README.md) **and** [the install template](https://github.com/samvera/hyrax/blob/master/template.rb) to replace all instances of the past version with the new version.
1. Change the tag for the installation template in the README from, e.g. (if bumping from 1.0.0 to 1.1.0), `rails new my_app -m https://raw.githubusercontent.com/samvera/hyrax/v1.0.0/template.rb` to `rails new my_app -m https://raw.githubusercontent.com/samvera/hyrax/v1.1.0/template.rb`.
1. Add, commit, and push all your changes directly to the `master` branch of the **[upstream](https://github.com/samvera/hyrax)** repository. (If GitHub prevents you from pushing directly to `master`, submit your changes as a PR.)
1. Release the gem to rubygems.org via `rake release`. (See below if this doesn't work.)
1. Create [release notes in GitHub](https://github.com/samvera/hyrax/releases/new). In the new release, include at least a block with upgrade notes and a block showing the changelog -- see script [changelog.sh](https://github.com/samvera/hydra/blob/master/script/changelog.sh) for help generating changelog entries. (See [an example](https://github.com/samvera/sufia/releases/tag/v6.4.0).)
1. Update the [Hydra Stack Built-in Feature Checklist](https://wiki.duraspace.org/display/samvera/Built-in+Feature+Checklist) to indicate features that have been added, removed, or moved to a different layer of the stack.
1. Send a release message to [samvera-tech](mailto:samvera-tech@googlegroups.com), [samvera-community](mailto:samvera-community@googlegroups.com), and [samvera-releases](mailto:samvera-releases@googlegroups.com) describing the changes (which you can copy from the GitHub release). (*This assumes you've already joined those three lists.* Do that first!) It may be helpful to base your message on [a prior example](https://groups.google.com/forum/#!topic/samvera-releases/SvQAhtIgpqA), which also contains some new text explicitly thanking contributors, which you can get from the changelog.

## Problems running the release task

If this is your first time pushing to rubygems.org for a Hydra project:
  1. Create an account at rubygems.org if you haven't already, or make note of your rubygems.org email address.
  1. Ask on Slack or hydra-tech about making you an owner of the gem on rubygems.org. (Your rubygems.org email address and GitHub username need to be added to [this script](https://github.com/samvera/hydra/blob/master/script/grant_revoke_gem_authority.rb#L19), and then someone who already has owner access needs to run that script.)
  1. Ask on Slack or hydra-tech about adding you to the Owners team in the projecthydra GitHub organization.
  1. If you have not yet, set up `~/.gem/credentials` according to [these instructions](https://rubygems.org/profile/edit).

Then try again.