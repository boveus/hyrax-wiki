# NOTE

This document was copied from the Sufia wiki. It is likely outdated and should be considered deprecated.

# Issues

A place to list "common issues" and links to their solutions

1. If you get error *HTTP 404 not found* when navigating to `/admin/stats`. The problem might be that you don't have the proper configuration for Solr. Make sure you run the `rails hyrax:jetty:config` task indicated in the README. This rake task copies the proper `solrconfig.xml` to your Solr installation which lists `/terms` as valid path.
