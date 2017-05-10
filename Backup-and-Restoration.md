# Hyrax Backup and Restoration/Migration

Migrating an existing Hyrax Hydra system to a new box or backing up data for disaster recovery purposes is fairly simple but has a few gotcha moments that can cause problems for restoration of data.

(This guide was copied from the Sufia wiki. Most of it should apply to Hyrax but it has not yet been tested.)

# Fedora:
Fedora 4 has multiple options for a few types of recovery.

## REST APIs:
A simple all in one option is to use the built in REST call to Fedora [see here for details](https://wiki.duraspace.org/display/FEDORA451/Backup+and+Restore).
You can see an example below which assumes a port of 8080 for fedora and a backup path /backup:

    curl -X POST -d "/backup" "localhost:8080/fedora/rest/fcr:backup"
This generates a directory of data and zip files that allow you to rebuild the repository. To ensure the stability of the data while this runs it pauses the Fedora respository.

The backup files can then be moved to a new location and restored to a machine with Fedora running by sending the restore REST command:

    curl -X POST -d "/backup" "localhost:8080/fedora/rest/fcr:restore"

Pros:

* Extremely stable, this is designed to handle in-use repositories and pause any transactions while it runs.

Cons:

* This method is extremely slow. A test backup of about 40 GB of data took 4-5 hours.

* This method does not support incremental or differential backup, so there's no real way to make this fast.

## Filesystem:
Another option for backing up Fedora is to use filesystem level tools. Fedora is a fairly robust database so as long as the data appears to be in the same rough configuration folders/locations it is fairly easy to use tools like rsync or other backup software to handle backups.

You will need to back up the two directories listed towards the end of this [Fedora guide](https://wiki.duraspace.org/display/FEDORA451/Backup+and+Restore):

* fcrepo.binary.directory

* fcrepo.ispn.repo.cache

Simply make sure those are both backed up and you can restore them to the new machine. This will transfer over all Fedora data. If you want to simply back up the entire Fedora directory that also works but may add some small amount of space to the backups.

Pros:

* Very fast.

* Allows whatever backup method you want to use.

Cons:

* It does not pause new submissions, so batch ingests that are running but incomplete may be missed. You may wish to pause/stop Fedora to avoid this.

* You may wish to use the LevelDB validation tools to validate the backup or test it regularly.

# Postgres:
The Postgres database handles user accounts and authentication. Any of the major Postgres options for backup and restoration work well [see here](https://www.postgresql.org/docs/current/static/backup.html). Generally speaking the Postgres data is fairly small and does not require a Point-in-Time recovery. SQL dumps, as they allow for version and architecture changes compared to other Postgres options, are a safer bet.

Postgres by default asks for a password whenever connecting. If your backup software does not have a way to automate that, or if you are using scripts to generate the dump file a .pgpass file is recommended to store a user name and password. See [this guide] (https://www.postgresql.org/docs/current/static/libpq-pgpass.html) for details as to formatting.

When you load a new postgres database in, you will likely need to assign permissions to the user account managing it as a freshly loaded database may overwrite the permissions depending on your options for the backup dump.

    GRANT Create,Connect,Temporary ON DATABASE sample TO user;

# Minter:
The minter should be saved to avoid future collisions with already existing objects.
The default location of minter-state is /tmp/minter-state.
Simply copy the minter to the new machine and adjust permissions to fit the account(s) you have running Hydra.

# Redis:
Finally, the Redis database mut be carried over to maintain the version history for objects. By default this can be found in

    /var/lib/redis/dump.rdb
To load it into a new machine you need to:

1. Stop redis
2. Replace the existing dump.rdb with your back up copy
3. You may need to adjust the permissions of the new dump.rdb to be redis:redis if those were not carried over by your backup method.
4. Restart redis

If you copy dump.rdb over to /var/lib/redis without stopping Redis, it will overwrite the dump.rdb with the current state in a short time.

# Indexing:
When all the data has been moved, you will need to re-index Solr in Hydra so that everything shows up in the user view on the new machine. Head to the current Hyrax folder

    cd /opt/hyrax-project/current
Enter the rails console

    bundle exec rails console production
Then reindex all the data

    ActiveFedora::Base.reindex_everything

For reasons not yet known, reindexing a single time may miss some objects, often ones with multiple versions, so run it twice to be safe.

# Common Problems
## Version history is missing
This means the Redis database did not load properly. Check the permissions and owner on the dump.rdb file.

## Fedora won't load
If you can't even access Fedora via the landing page (host:8080/fedora) you probably didn't set ownership recursively to the directory. Use

    sudo chmod -R user:user /directory
user should be based on your servlet container's account.
