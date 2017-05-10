These are the steps for merging commits from CurationConcerns and Hyrax into Hyrax.

Step 1 only needs to be done once.

1. Add CurationConcerns (or Hyrax) as a remote to your local Hyrax repo: `git remote add cc https://github.com/projecthydra/curation_concerns.git` or `git remote add hyrax https://github.com/projecthydra-labs/hyrax.git`
1. Fetch from the new remote: `git fetch cc` or `git fetch hyrax`
1. Start a new branch for the merge: `git checkout -b merge_cc` or `git checkout -b merge_hyrax`
1. Merge the latest commits into your branch: `git merge cc/master` or `git merge hyrax/master`
1. Resolve any conflicts, keeping an eye out for references to CC or Hyrax in the code or file paths, which you should change to Hyrax
1. Run the specs: `rspec`
1. Create a PR