To use Resque in Sufia 7 add the following to `sufia/.internal_test_app/config/application.rb`

```
config.active_job.queue_adapter = :resque
```

To enable the routes that allow you to view what jobs have been queued add the following to `sufia/app/config/routes.rb (in a production application you probably want to make sure only admin users have access to these routes) -- TODO: There routes are already there but only enabled if Sufia::ResqueAdmin has been defined (???). 

```
  namespace :admin do
    # Job monitoring
    mount Resque::Server, at: 'queues'
  end
```

Restart your Rails app and browse to `http://localhost:3000/admin/queues/overview` to see the queues.  

If the queues overview page shows a text that reads "0 of 0 Workers Working" it means that Resque has probably not been started. To start Resque run the following command from a separate terminal window and leave this terminal open/running. 

```
cd .internal_test_app
resque-pool start
```

Refresh the `http://localhost:3000/admin/queues/overview` page and you should see "0 of 1 Workers Working" to confirm that there is a worker ready to handle the jobs.

You can also start Resque as daemon via `resque-pool --daemon` so you don't have to leave the second terminal window open but I prefer to initially test without the daemon to make things work.



