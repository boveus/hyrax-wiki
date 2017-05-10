A way to run Fedora and Solr while doing development work in Sufia 7.

In a terminal window run the following command to launch Fedora:

    bundle exec fcrepo_wrapper -p 8984

In a separate terminal window run the following command to launch Solr:

    bundle exec solr_wrapper -p 8983 -d solr/config -n hydra-development

In a separate terminal window run the following command to run the Rails application:

    cd .internal_test_app
    bundle exec rails s

With this approach you can stop and restart the Rails application without having to kill Fedora and Solr. 