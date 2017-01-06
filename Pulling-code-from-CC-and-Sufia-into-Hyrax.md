These are the steps for merging commits from CurationConcerns and Sufia into Hyrax.

Step 1 only needs to be done once.

1. Add CurationConcerns (or Sufia) as a remote to your local Hyrax repo: `git remote add cc https://github.com/projecthydra/curation_concerns.git` or `git remote add sufia https://github.com/projecthydra/sufia.git`
1. Fetch from the new remote: `git fetch cc` or `git fetch sufia`
1. Start a new branch for the merge: `git checkout -b merge_cc` or `git checkout -b merge_sufia`
1. Merge the latest commits into your branch: `git merge cc/master` or `git merge sufia/master`
1. Resolve any conflicts, keeping an eye out for references to CC or Sufia in the code or file paths, which you should change to Hyrax
1. Run the specs: `rspec`
1. Create a PR