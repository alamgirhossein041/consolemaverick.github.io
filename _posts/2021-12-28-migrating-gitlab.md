---
title: "How to move a project between Gitlab Instances preserving issue assignments"
layout: post
subtitle: "Why is everything owned by one user?"
description: "Learn how to make sure everything is migrated between instances"
image: /assets/img/uploads/gitlab.png
date: '2021-12-28 07:28:05 +0900'
optimized_image: /assets/img/uploads/gitlab.png
category: fundementals
tags:
  - linux
  - gitlab
author: Maverick Stoklosa
---

Out of the box the export feature will result in all issues being owned by the importing user so if you import like so

```
gitlab-rake "gitlab:import_export:import[root, group/subgroup, testingprojectimport, /path/to/file.tar.gz]"
```

then all the issues will be assigned to `root` user aka. Administrator. You don't want the admin to get all the issues, you want the original authors and assignees to be preserved during the migration.

This occurs despite the usernames and email addresses of the users matching exactly between the instances.

The [official documentation](https://docs.gitlab.com/ee/user/project/settings/import_export.html) on this states the following requirements:

> Imported users can be mapped by their public email on self-managed instances, if an administrative user (not an owner) does the import. Additionally, the user must be an existing member of the namespace, or the user can be added as a member of the project for contributions to be mapped. Otherwise, a supplementary comment is left to mention that the original author and the MRs, notes, or issues are owned by the importer.

The key point being `public email`. When exporting the project from the original Gitlab instance examine the tar.gz file. After you extract it you will see a project_members.json file.

```
cat tree/project/project_members.ndjson

{"id":1421,"access_level":30,"source_type":"Project","user_id":659,"notification_level":3,"created_at":"2021-10-18T07:42:57.084Z","updated_at":"2021-10-18T07:42:57.084Z","created_by_id":24,"invite_email":null,"invite_token":null,"invite_accepted_at":null,"requested_at":null,"expires_at":null,"ldap":false,"override":false,"user":{"id":659,"username":"maverick,"public_email":""}}
```

The `public_email` field is blank 😲

This is because the `public_email` is actually blank in the database.

```
sudo gitlab-rails console
u = User.find_by_username("maverick")
u.public_email
""
```

You can fix `public_email` field:

```
u.public_email = u.email
u.save
```

Now when you export through the Gitlab interface the newly generated `json` will contain populated `public_email` fields and all the users will be mapped between the instances correctly.
