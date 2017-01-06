These are the steps for merging commits from CurationConcerns into Hyrax reposotory.

Step 1 only needs to be done the first time.

1. add CurationConcerns as a remote to your hyrax repo:  `git remote add cc https://github.com/projecthydra/curation_concerns.git`
1. fetch from Curation Concerns: `git fetch cc`
1. checkout a branch `git checkout -b merge_cc` (edited)
1. merge the latest commits into your branch `git merge cc/master`
1. Resolve any conflicts
1. run the specs `rspec`
1. create a PR