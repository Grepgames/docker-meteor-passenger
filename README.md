[![Docker Repository on Quay.io](https://quay.io/repository/grepgames/meteor/status "Docker Repository on Quay.io")](https://quay.io/repository/grepgames/meteor) Meteor for Development and Production Using Passenger

## Features:

 * Meteor 1.x package/bundle support
 * Git-based repository + branch/tag via environment variables (`REPO` and `BRANCH`)
 * Bundle URL via environment variable (`BUNDLE_URL`)
 * Local bundle file via environment variable (`BUNDLE_FILE`)
 * Bind-mount, volume, Dockerfile `ADD` via environment variable (`APP_DIR`)
 * Uses docker-linked MongoDB (i.e. `MONGO_PORT`...) or explicit setting via environment variable (`MONGO_URL`)
 * NOTE: This does NOT set `MONGO_OPLOG_URL`.  There were too many potention complications.  As a result, unless you explicitly set `MONGO_OPLOG_URL`, Meteor will fall back to a polling-based approach to database synchronization.  Note that oplog tailing requires a working replica set on your MongoDB server as well as access to the `local` database.
 * Optionally specify the port on which the web server should run (`PORT`); defaults to 80
 * Deploy-key support (set `DEPLOY_KEY` to the location of your SSH key file)
   * old `GITHUB_DEPLOY_KEY` is also supported but deprecated
 * Non-root location of Meteor tree; the script will search for the first .meteor directory
 * PhantomJS pre-installed to support `spiderable` package
 * embed nginx (phusion-passenger)

## Versions:

Regardless of the version number of the tool included in this package, your application will run
on whatever version of Meteor it is configured to run under.

## Example nginx config

`MeteorAppName/private/nginx.conf` or `-v configDirName/nginx.conf:/etc/nginx/sites-enabled/nginx.conf`

```
server {
    listen 80;
    server_name http://testsite.com;
    root /var/www/public;  # do not modify root directory

    passenger_enabled on;
    passenger_user app;
    passenger_sticky_sessions on;
    passenger_env_var MONGO_URL mongodb://mymongoserver.com:27017/database;
    passenger_env_var MONGO_OPLOG_URL mongodb://mymongoserver.com:27017/database;
    passenger_env_var ROOT_URL http://testsite.com;

    # Set these ONLY if your app is a Meteor bundle!
    passenger_app_type node;
    passenger_startup_file bundle/main.js;
}

```

If you write params like MONGO_ULR in nginx.conf, docker environment variables can't overide it.

## Example run:

`docker run --rm -p 8000:80 -e ROOT_URL=http://testsite.com -e REPO=git@github.com:yourName/testsite.co.git" -e DEPLOY_KEY=/home/core/.ssh/id_rsa -e BRANCH=testing -e MONGO_URL=mongodb://mymongoserver.com:27017 -v ~/.ssh/deploy_key:/home/core/.ssh/id_rsa -v ~/conf/nginx.conf:/etc/nginx/sites-enabled/nginx.conf quay.io/grepgames/meteor`

or

`docker run --rm -p 8000:80 -e ROOT_URL=http://testsite.com -e BUNDLE_FILE=/media/deploy/bundle.tar.gz -e MONGO_URL=mongodb://mymongoserver.com:27017 -v ~/conf/nginx.conf:/etc/nginx/sites-enabled/nginx.conf quay.io/grepgames/meteor`

There is also a sample systemd unit file in the Github repository.
