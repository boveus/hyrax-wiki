To use Resque in Sufia 7 add the following to `sufia/.internal_test_app/config/application.rb`

```
config.active_job.queue_adapter = :resque
```

To enable the routes that allow you to view what jobs have been queued add the following to `sufia/app/config/routes.rb (in a production application you probably want to make sure only admin users have access to these routes)

```
  namespace :admin do
    # Job monitoring
    mount Resque::Server, at: 'queues'
  end
```

Restart your Rails app and browse to `http://localhost:3000/admin/queues/overview` to see the queues.  

