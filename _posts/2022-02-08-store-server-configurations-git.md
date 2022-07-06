---
title: "How to store Server Configurations in source control"
layout: post
subtitle: "Put all the important files into Git"
description: "Learn how to store important server configuration files in Git Version control so you have a historical view of changes."
image: /assets/img/uploads/server_git.png
date: '2022-02-08 07:28:05 +0900'
optimized_image: /assets/img/uploads/server_git.png
category: fundementals
tags:
  - linux
  - git
  - server
  - configuration
author: Pawel Stoklosa
---

The same way you version control your code you can version control the configuration files on the server. There are 3 kinds of things you want to keep track of on a server

1. **configuration files for the server itself** and it's components like PostgreSQL, Redis, MongoDB etc.
2. **application startup scripts**
3. **application configuration files**

The **benefits** of tracking your configuration files in Git:

1. see what has changed on the server and who committed the changes
2. ability to rollback changes if there's undesired side effects
3. determine if someone malicious has gained access to the machine
4. historical view of the changes to any file
5. everything you need to recreate the infrastructure from scratch in case of a catastrophic failre
6. empowers the developer to examine how the application runs on the server itself

In addition to tracking the files in Git this configuration will also automatically commit any changes on a daily basis.

The important parts of the server reside in 3 locations:

1. `/etc`
2. `/var/www/{env}/{app}/shared/scripts`
3. `/var/www/{env}/{app}/shared/config`

Where `env` is the environment like `development`, `staging`, `production` and `app` is the application name. Each location is stored in a particular way.

# etc

`/etc`

This is where the **configurations are stored for the server itself** and all the components running on the server like PostgreSQL, Redis, Monit, Nginx.

This location is stored using [etckeeper](http://joeyh.name/code/etckeeper/)

In order to start storing this component in Git you will have to:

1. Make sure you’re forwarding your ssh key via `ssh -A` when connecting to the server.
2. Create a repository named after the server hostname i.e. host.env.domain.com
3. On the server install `etckeeper` via

```
apt install etckeeper
```

4. Set the remote version control location in the `/etc/etckpeer/etckeeper.conf` file

```
PUSH_REMOTE="git@gitlab.domain.com:group/project/app.env.domain.com.git”
```

5. Initialize etckeeper via

```
sudo -E etckeeper init
```

You will have to use `sudo -E` for all the commands to preserve your Git name and `ssh` key when pushing to version control.

6. Go to `/etc` to see what is to be committed via

```
sudo -E git status
```

7. Add your changes via

```
sudo -E git add .
```

8. Make your initial commit

```
sudo -E etckeeper commit
```

9. Push your changes to the remote branch


```
git push --set-upstream git@gitlab.domain.com:project/app.env.domain.com.git master
```

10. Configure the changes to be automatically committed daily

```
/etc/etckeeper/commit.d/60github-push
```

```
#!/bin/sh
set -e
if [ "$VCS" = git ] && [ -d .git ]; then
  cd /etc/
  git push
fi
```

```
sudo systemctl start etckeeper.timer
```

# Shared Scripts

`/var/www/{env}/{app}/shared/scripts`

These are the **scripts that make the application run**.

`env.sh`: defines environment variables needed to make the application run
`puma.sh`: starts the puma server which is connected to nginx to serve the application
`sidekiq.sh`: starts the sidekiq background jobs
`console.sh`: starts the Rails console with the correct user priveleges
...

These files go into the Git repository of the application `git@gitlab.domain.com:group/project/app.git`

They are modified locally in the repository and deployed to the server via Capistrano.

These files should be stored in `app/config/server/{env}/scripts`

The Capistrano recipe must be modified to deploy the scripts to the server.

```
before 'deploy:symlink:shared', 'deploy:configure_scripts'

namespace :deploy do
  task :configure_scripts do
    on roles(:web) do
      within release_path do
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/scripts/env.sh /var/www/#{fetch(:rails_env)}/#{fetch(:application)}/shared/scripts/env.sh")
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/scripts/puma.sh /var/www/#{fetch(:rails_env)}/#{fetch(:application)}/shared/scripts/puma.sh")
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/scripts/console.sh /var/www/#{fetch(:rails_env)}/#{fetch(:application)}/shared/scripts/console.sh")
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/scripts/sidekiq.sh /var/www/#{fetch(:rails_env)}/#{fetch(:application)}/shared/scripts/sidekiq.sh")
      end
    end
  end
end
```

# Shared Config

`/var/www/{env}/{app}/shared/config`

These are the **configuration files for the application**.

`database.yml`: defines how the application will connect to PostgreSQL, Redis, MongoDB etc.
`secrets.yml`: encrypted variables set by the application
`sidekiq_schedule.yml`: sidekiq background job schedule
...

These files go into the Git repository of the application `git@gitlab.domain.com:group/project/app.git`

They are modified locally in the repository and deployed to the server via Capistrano.

These files should be stored in `app/config/server/{env}`

The Capistrano recipe must be modified to deploy the scripts to the server.

```
before 'deploy:symlink:shared', 'deploy:configure_app'

namespace :deploy do
  task :configure_app do
    on roles(:web) do
      within release_path do
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/database.yml #{release_path}/config/database.yml")
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/secrets.yml #{release_path}/config/secrets.yml")
        execute("ln -sfn #{release_path}/config/server/#{fetch(:rails_env)}/sidekiq_schedule.yml #{release_path}/config/sidekiq_schedule.yml")
      end
    end
  end
end
```

Finally you will have to remove the linked files section of deploy.rb

```
  set :linked_files, fetch(:linked_files, []).push(
    'config/database.yml',
    'config/secrets.yml',
  )
```

Since the files are no linked in the configure_app part of the deploy process.

Finally for the automatic updates to work you will have to configure a dedicated external user in Gitlab i.e. hostname.

Give this user maintainer access to the server configuration git repository only and add the ssh key for this user generated on the server using `ssh-keygen`.

Confirm root user can push to the remote repository via

```
cd /etc
git add .
git commit -m 'root push'
git push
```
