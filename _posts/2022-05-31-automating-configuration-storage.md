---
title: "Automating server storage configuration using Git, Gitlab, etckeeper, Ansible and Ruby"
layout: post
subtitle: "Automate it"
description: "Instead of doing it manually"
image: /assets/img/uploads/automation.png
date: '2022-05-31 06:28:05 +0900'
optimized_image: /assets/img/uploads/automation.png
category: fundementals
tags:
  - Gitlab
  - Ruby
  - Ansible
author: Pawel Stoklosa
---

Storing server configuration inside of Git requires multiple steps. In order to optimize the process let’s automate as much of it as possible.

There are 3 components.

1. Create the SSH key for the root user on the server itself.
2. Use the SSH key from step 1 to create a user and project in Gitlab to store your server configuration.
3. Configure etckeeper on the server to use the project created in step 2 to upload the server configuration to the remote branch.

Let's go into each one in depth.

## Create root user SSH key

Let's start with the SSH key. For this we will use an Ansible play:

```
- hosts: all
  become: yes

  tasks:
  - name: Generate SSH key for root user
    shell: ssh-keygen -t ed25519 -f /root/.ssh/id_ed25519 -q -N ""
    ignore_rrors: true

  - name: Capture SSH key
    shell: cat /root/.ssh/id_ed25519.pub
    register: ssh_key

  - name: Display SSH key
    debug:
      var: ssh_key.stdout

```

This play generates an ssh-ed25519 SSH key for the root user. It then displays the SSH key so you can copy and paste it into the next component. The output will be something like this.

```
TASK [Generate SSH key for root user] *************************************************************************************************************************************************************************
Tuesday 01 June 2021  10:41:11 +0900 (0:00:08.656)       0:00:08.712 **********
changed: [server.com]

TASK [Capture SSH key] ****************************************************************************************************************************************************************************************
Tuesday 01 June 2021  10:41:13 +0900 (0:00:01.547)       0:00:10.260 **********
changed: [server.com]

TASK [Display SSH key] ****************************************************************************************************************************************************************************************
Tuesday 01 June 2021  10:41:14 +0900 (0:00:01.620)       0:00:11.880 **********
ok: [server.com] =>
  ssh_key.stdout: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKCUfYb2d/A3XCILyPbgLe4NJegkoRe9CpFUjfsMyUBl root@server.com

PLAY RECAP ****************************************************************************************************************************************************************************************************
server.com : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

Tuesday 01 June 2021  10:41:14 +0900 (0:00:00.072)       0:00:11.953 **********
===============================================================================
Gathering Facts ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 8.66s
Capture SSH key ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 1.62s
Generate SSH key for root user ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 1.55s
Display SSH key ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 0.07s
```

## Create Gitlab project/user to store the server configuration

For this we will use a Ruby shell script.

```
require 'httparty'
require "awesome_print"
require 'cli/ui'
require 'byebug'

def post(url, data)
  response = HTTParty.post(url, body: data, headers: { 'PRIVATE-TOKEN' => @access_token })
  JSON.parse(response.body)
end

def get(url)
  response = HTTParty.get(url, { headers: { 'PRIVATE-TOKEN': @access_token } })
  JSON.parse(response.body)
end

def setup_gitlab
  project_id = create_project
  user_id = create_user(project_id)
  add_user_to_project(project_id, user_id)
  add_ssh_key(user_id)
end

def create_user(project_id)
  data = { email: "maverick+#{@server_name}@server.com", name: @server_name, username: @server_name , external: true, force_random_password: true}
  response = post("#{@BASE_URL}/users", data)
  user_id = response['id']
  puts "Created user #{user_id}"
  user_id
end

def add_user_to_project(project_id, user_id)
  maintainer_access_level = 40
  data = { user_id: user_id, access_level: maintainer_access_level }
  post("#{@BASE_URL}/projects/#{project_id}/members", data)
  puts "Added user to project"
end

def add_ssh_key(user_id)
  data = { title: @server_name, key: @ssh_key}
  post("#{@BASE_URL}/users/#{user_id}/keys", data)
  puts "Added SSH key to user"
end

def create_project
  data = { namespace_id: @namespace_id, name: @server_name }
  response = post("#{@BASE_URL}/projects", data)
  puts "Created Project with repo URL: #{response['ssh_url_to_repo']}"
  response['id']
end

def get_groupname
  response = get("#{@BASE_URL}/groups/#{@namespace_id}")
  response['name']
end

def get_subprojects
  response = get("#{@BASE_URL}/groups/#{@namespace_id}")
  group_projects = response['projects']
  project_names = []
  group_projects.each do |project|
    project_names << project['name']
  end

  project_names
end

if ARGV.length != 4
  puts "Input access_token namespace_id server_name and ssh_key"
  exit
end

@access_token = ARGV[0]
@namespace_id = ARGV[1]
@server_name = ARGV[2]
@ssh_key = ARGV[3]

@BASE_URL = 'https://gitlab.server.com/api/v4/'

subproject_list = get_subprojects.join(', ')

CLI::UI::Prompt.ask("Use Namespace #{get_groupname} with projects: #{subproject_list}?") do |handler|
  handler.option('y') { |selection| setup_gitlab }
  handler.option('n') { |selection| puts 'Goodbye' }
end
```

In order to use it you have to specify

1. `API_TOKEN`
2. `GITLAB_GROUP_ID`
3. `SERVER_NAME`
4. `ROOT_SSH_KEY_ON_SERVER`

on the command line like so:

```
ruby project.rb API_TOKEN GITLAB_GROUP_ID SERVER_NAME "ROOT_SSH_KEY_ON_SERVER"
```

where the `ROOT_SSH_KEY_ON_SERVER` is what you generated in step 1 above. the `GITLAB_GROUP_ID` is the group that will house all the server configurations for this particular client. You can get this from the browser by going to the client group page in Gitlab. It will display the 'Project ID'. In Gitlab this is the namespace/group id.

The `SEVER_NAME` is the domain name or other identifier you want to use for the server. It will become the project name, name of the user that will be allowed to upload to the repository and the the Git repository name will be based on it as well.

The `API_TOKEN` is generated by you from Gitlab settings. You have to enable `api` access for this key.

The output of this script will be something like this:

```
? Use Namespace NAMESPACE with projects: project1? (You chose: y)
Created Project with repo URL: git@gitlab.server.com:group/client/client.server.com.git
Created user 556
Added user to project
Added SSH key to user
```

You can get the latest version on [Github](https://github.com/consolemaverick/gitlab-tools).

You will need the

```
Created Project with repo URL: git@gitlab.server.com:group/client/client.server.com.git
```

repository for the next step.

## Configure etckeeper to use Gitlab to store the server configuration

For this step let's leverage Ansible again.

```
- hosts: all
  become: true

  tasks:
  - name: Install etckeeper
    apt:
      pkg: etckeeper
      state: latest

  - name: Configure remote repository
    lineinfile:
      dest: /etc/etckeeper/etckeeper.conf
      regexp: 'PUSH_REMOTE=*'
      line: 'PUSH_REMOTE="{{server_vcs_repository}}"'

  - name: Create script to commit automatically
    template:
      src: "{{inventory_dir}}/../../files/etckeeper/60github-push"
      dest: /etc/etckeeper/commit.d/60github-push
      owner: "root"
      group: "root"
      mode: 0755

  - name: Start etckeeper timer
    ansible.builtin.systemd:
      name: etckeeper.timer
      state: started

  - name: Init etckeeper
    shell: cd /etc;etckeeper init

  - name: Add files to git
    shell: cd /etc;git add .

  - name: Initial commit
    shell: cd /etc;git commit -m 'Initial Commit'
    ignore_errors: true

  - name: Add gitlab.server.com SSH key
    shell: ssh-keyscan gitlab.server.com >> ~/.ssh/known_hosts

  - name: Push to remote branch
    shell: cd /etc;git push --set-upstream {{server_vcs_repository}} master
```

Where `{{inventory_dir}}/../../files/etckeeper/60github-push` contains

```
#!/bin/sh
set -e
if [ "$VCS" = git ] && [ -d .git ]; then
  cd /etc/
  git push
fi
```

This is the script that will be called by the systemd timer to push any changes made to the server everyday.

For this to work you have to set `server_vcs_repository` in your Ansible group to the output of the above script

```
server_vcs_repository: "git@gitlab.server.com:group/client/client.server.com.git"
```

When you run the above you will see that the server has no pushed the initial commit to the `client.server.com` project Git repository. You can verify this through the Gitlab web interface.e
