---
title: "How to create SBOM for Ruby on Rails application during the build phase of Continuous Integration in Gitlab"
layout: post
subtitle: "Software bill of materials"
description: "Learn how to leverage Gitlab to do it"
image: /assets/img/uploads/receipts2.jpeg
date: '2022-05-17 06:28:05 +0900'
optimized_image: /assets/img/uploads/receipts2.jpeg
category: fundementals
tags:
  - SBOM
  - Gitlab
  - CI
author: Maverick Stoklosa
---

Start by installing the [CycloneDX Ruby Gem](https://github.com/CycloneDX/cyclonedx-ruby-gem) which will generate the Software Bill of Materials for you.

In your Gemfile add

```mysql
gem 'cyclonedx-ruby'
```

The install the new gem

```mysql
bundle install
```

Test the configuration by generating the SBOM locally

```mysql
cyclonedx-ruby -p .
```

You can now examine the resulting bom.xml file.

With the setup working we can integrate it into the build phase of Continuous Integration. In the .gitlab-ci.yml file add a section for SBOM generation

```mysql
variables:
    # SBOM
    GITLAB_TOKEN: "gitlab_token"
    GITLAB_USER: 'maverick'
    GITLAB_EMAIL: "consolemaverick@gmail.com"
    COMMIT_MESSAGE: 'Update SBOM'
    CI_PROJECT_PATH: "group_name/project_name"
    CI_SERVER_HOST: "gitlab.company.com"

  sbom:
    stage: build
    script:
      - git config --global user.email "${GIT_USER_EMAIL:-$GITLAB_EMAIL}"
      - git config --global user.name "${GIT_USERNAME:-$GITLAB_USER}"
      - git checkout "${CI_COMMIT_BRANCH}"
      - git remote set-url origin "https://${GITLAB_USER}:${GITLAB_TOKEN}@${CI_SERVER_HOST}/${CI_PROJECT_PATH}.g
      - git pull
      - bundle exec cyclonedx-ruby -p .
      - git add bom.xml

      - |-
        # Check if we have modifications to commit
        CHANGES=$(git status --porcelain | wc -l)

        if [ "$CHANGES" -gt "0" ]; then
          git status
          git commit -m "${COMMIT_MESSAGE}"
          git push origin "${CI_COMMIT_BRANCH}" -o ci.skip
        fi
```

You will need to generate a Gitlab personal access token with read/write repository permissions replacing the GITLAB\_TOKEN variable above.
