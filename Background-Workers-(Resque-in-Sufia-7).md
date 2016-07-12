Sufia 7 no longer packages a default queuing back-end. Sufia 7 builds its jobs using Rails' ActiveJob framework, so you are free to use the queuing system of you choice (e.g. Resque, DelayedJob, Sidekiq) to manage long-running or slow processes. Flexibility and choice come with a cost, though, and there's some work involved in integrating whichever queueing back-end you select. This page offers guidance on installing and using Resque with Resque-pool to handle background jobs in your Sufia app.

## Pre-Requisites: Install and Run Redis

Resque relies on the [Redis](http://redis.io/) key-value store, so [Redis](http://redis.io/) must be installed *and running* on your system in order for background workers to pick up jobs.

## Code Changes: Install Resque

To use Resque -- [learn more about Resque](https://github.com/resque/resque) -- as your queueing back-end, you must modify the code created by the Sufia generator. Resque offers instructions for [installation](https://github.com/resque/resque/#in-a-rails-3x-or-4x-app-as-a-gem); Resque-pool also offers [instructions](https://github.com/nevans/resque-pool#how-to-use) for installation and use. In general, you need to add resque and/or resque-pool to your `Gemfile`, require resque tasks in your `Rakefile`, and configure Rails to use Resque as its ActiveJob adapter. 

## Managing Resque Workers

There are two ways you can manage your workers:

1. the `resque:work` rake task, which will run in the foreground in the terminal;
2. use [resque-pool](https://github.com/nevans/resque-pool) to manage a number of configurable workers in background processes

For the remainder of the background worker documentation, it is assumed that you're using resque-pool which is more robust, in which case you should make sure you've run through resque-pool's installation process (which involves tweaking your `Gemfile` and adding Rake tasks) first.


## Terminology

### Resque

Resque is a [message queue](https://en.wikipedia.org/wiki/Message_queue) that can be used by Sufia to manage long-running or slow processes. 

### Resque-Pool/pools

[Resque-Pool](https://github.com/nevans/resque-pool) is a tool for managing (starting, stopping) and configuring Resque worker processes. See [configuration](#configuration) below for more information.

### workers

Workers run the background jobs. Each worker has a copy of your Sufia app which has the jobs code in `app/jobs/`. The workers listen to queues (by polling Redis) and pull jobs waiting on the queue. Once a worker pulls a job, it will perform the task as expressed by the persisted job. A worker can be dedicated to a single queue or may listen to multiple queues -- this is [configurable](#configuration) in `config/resque-pool.yml`. Multiple workers can also listen to the same queue.

### queues

Jobs are sent to queues where they wait until a worker is available to pick them up. Sufia defines a number of queues for processing different background jobs (e.g. `batch_update`, `characterize`). Multiple queues are provided to give you the ability to control how many workers work on the different jobs. Why? Some jobs are fast-running, such as all of Sufia's event jobs, and some jobs are slow-running like the characterize job. Using dedicated queues allows you to say, "I only want one worker for characterization but I want five for events," thus making sure characterization jobs serially and event jobs run in parallel.

### jobs

A job is a task to be performed, and is encoded in JSON and persisted in Redis. The job includes the name of the method the worker should execute along with any parameters that need to be passed to the method.

### Redis

[Redis](http://redis.io/) is a key-value store. Resque uses Redis to persist jobs and queues, and Resque-Pool uses Redis to track and manage its worker processes.

## Configuration

Resque-pool's configuration file lives in your application at `config/resque-pool.yml` and you are free to tweak it to meet your needs. The default configuration is to create one worker for all queues:

```yaml
"*": 1
```

An example of a common configuration for production-like environments can be seen in [ScholarSphere](https://github.com/psu-stewardship/scholarsphere/blob/develop/config/resque-pool.yml):

```yaml
batch_update: 3
derivatives: 1
resolrize: 1
audit: 2
event: 5
import_url: 3
sufia: 1
"*": 1
```

Each line defines a queue name and how many workers should be created to process that queue.

## Starting the pool

**Prerequisite:**  [Redis](http://redis.io/) must be installed *and running* on your system in order for background workers to pick up jobs.  Unless Redis has already been started, you will want to start it up. You can do this either by calling the `redis-server` command, or if you're on certain Linuxes, you can do this via `sudo service redis-server start`.

OPTION 1: Start 1 worker (ignores the resque-pool [configuration file](#configuration)). The following command will run until you stop it, so you may want to do this in a dedicated terminal and would typically be used during development only.

**IMPORTANT:** Change directories to the root of your Sufia app before executing:

```
RUN_AT_EXIT_HOOKS=true TERM_CHILD=1 QUEUE=* rake environment resque:work
```

OPTION 2: Start a pool of configurable workers. This is typically used for production-like environments, but may also be used for development.  See [configuration](#configuration) examples above.

**IMPORTANT:** Change directories to the root of your Sufia app before executing:

```
RUN_AT_EXIT_HOOKS=true TERM_CHILD=1 resque-pool --daemon --environment development start
```

(Occasionally, Resque may not give background jobs a chance to clean up temporary files. The `RUN_AT_EXIT_HOOKS` variable allows Resque to do so. The `TERM_CHILD` variable allows workers to terminate gracefully rather than interrupting currently running workers.)

For more information on the signals that Resque-Pool responds to, see the [resque-pool documentation on signals](https://github.com/nevans/resque-pool#signals).

## Restarting the pool

**IMPORTANT:** Change directories to the root of your Sufia app before executing.

You may be interested in a script to help you manage starting/stopping Resque-Pool. Here's a [sample from ScholarSphere](https://github.com/psu-stewardship/scholarsphere/blob/develop/script/restart_resque.sh).

You may need to adjust it to meet your needs, e.g.: the default location for the pid file is the application's `tmp/` directory.  You will want to update `RESQUE_POOL_PIDFILE` to point to the location where your pid file is generated if you use a location other than the default.

## Monitoring

Edit `config/initializers/resque_admin.rb` so that `ResqueAdmin#matches?` returns a `true` value for the user(s) who should be able to access this page. One fast way to do this is to return `current_user.admin?` and add an `#admin?` method to your user model which checks for specific emails or the 'admin' role. See [Admin Users](#admin-users) for information on how to add users with the admin role.

Then you can view jobs at the `/admin/queues` route.

## Troubleshooting

The code executed by workers reflects the state of the code when the workers were started. If the workers are behaving in a way you can't explain, you may have more than one resque-pool master process running. You can also see unusual behaviors if more than one redis-server is running. The information below describes the processes that will be running when everything is operating correctly.

You should see one redis-server process:

```
$ ps -ef | grep redis-server
user1     7982  7882  0 01:26 pts/3    00:00:00 grep redis-server
root      8398     1  0 00:08 ?        00:00:04 /usr/local/bin/redis-server 0.0.0.0:6379
```

If you see multiple redis-server processes running, kill them all (assuming they're not serving other important applications on your system) and start redis-server again. You should only have one redis-server process running.

You should see one resque-pool-master process:

```
$ ps -ef | grep resque-pool
user1    8059  7882  0 01:27 pts/3    00:00:00 grep resque-pool
root     8416     1  0 00:08 ?        00:00:08 resque-pool-master[yourapphere]: managing [8653]
```

You should see one or more worker processes running:

```
$ ps -ef | grep resque | grep -v pool-master
root     8653  8416  0 00:08 ?        00:00:01 resque-1.25.2: Waiting for *
```

If you see multiple resque-pool-master processes running, kill all of them and all of their child processes as well. Start resque-pool again. You should only have one resque-pool-master process running.  But you may have multiple worker processes running.  

If resque or resque-pool fails to start with the error:
```
LoadError: No such file to load -- curation_concerns/actors/generic_work_actor
```
you have not correctly modified your Sufia app to use Resque as a back-end. See [Code Changes: Install Resque](#Code-Changes:-Install-Resque) for more information.
