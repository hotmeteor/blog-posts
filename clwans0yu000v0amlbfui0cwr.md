---
title: "Bitbucket Pipelines for Laravel apps"
datePublished: Fri May 17 2024 12:31:21 GMT+0000 (Coordinated Universal Time)
cuid: clwans0yu000v0amlbfui0cwr
slug: bitbucket-pipelines-for-laravel-apps
tags: laravel, bitbucket, bitbucket-pipelines

---

I never thought I'd be in this situation, but here we are: **I'm having to use** [**Bitbucket**](https://bitbucket.org) for a project.

I have nothing against Bitbucket, per se. I've just been a Github user for ~14 years, and haven't had any reason to ever use Atlassian's product. But a team my company, [Solenoid](https://twitter.com/SolenoidStudios), is working with right now had adopted it early on, so that's what we've got.

One thing they hadn't been able to set up yet is some nice automated processes for things like code formatting (with [Pint](https://github.com/laravel/pint) or [Prettier](https://prettier.io/)) or running the test suite when a PR is created. With [Github Actions](https://github.com/features/actions) this is now a breeze, and commonplace in most Laravel apps I've worked on.

Bitbucket has a comparable solution called [Bitbucket Pipelines](https://support.atlassian.com/bitbucket-cloud/docs/get-started-with-bitbucket-pipelines/), designed to do the same thing. Unfortunately, I could find next to no examples online on how to create more elaborate Pipelines for Laravel apps. So, after crawling the docs and picking up pieces from here and there, I've got a nice Pipeline for our app that formats our code and runs our tests on every PR (and PR update).

Here it is:

```yaml
# bitbucket-pipelines.yml

definitions:
  caches:
    composer:
      path: vendor
    node:
      path: node_modules
  services:
    mariadb:
      image: mariadb:11.3
      environment:
        MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
        MYSQL_DATABASE: 'laravel'
        MYSQL_USER: 'laravel'
        MYSQL_PASSWORD: 'secret'
  steps:
    - step: &phpunit
        name: "Run PHPUnit tests"
        image: fideloper/fly-laravel:8.3
        caches:
          - composer
        services:
          - mariadb
        script:
          - ln -f -s .env.ci .env
          - composer install --no-interaction --prefer-dist --optimize-autoloader
          - php artisan migrate
          - php artisan test --parallel
    - step: &pint
        name: "Pint"
        image: fideloper/fly-laravel:8.3
        caches:
          - composer
        script:
          - composer install --no-interaction --prefer-dist --optimize-autoloader
          - ./vendor/bin/pint -v --preset laravel
        artifacts:
          - app/**
          - config/**
          - database/**
          - routes/**
          - tests/**
    - step: &prettier
        name: "Prettier"
        image: node:20
        caches:
          - node
        script:
          - npm install
          - npm run prettier:fix
        artifacts:
          - resources/**
    - step: &commit
        name: "Commit back to branch"
        image: atlassian/default-image:4
        script:
          - git diff --quiet && git diff --staged --quiet || git commit -a -m "[skip ci] Pint and Prettier" && git push

pipelines:
  pull-requests:
    '**':
      - parallel:
          - step: *pint
          - step: *prettier
      - step: *commit
      - step: *phpunit
```

As you can see, this Pipeline is capable of doing a few things:

1. It will run our **Pint** and **Prettier** steps in parallel, tidying up any changes made to files in the PR. The fixes made are pushed into artifacts for the following step.
    
2. Artifacted code is then "quietly" diffed, and if there's diffs it will be committed back into the PR. The commit message is prefixed with `[skip ci]` as to not trigger an infinite loop of running Pipelines.
    
3. Finally, our Laravel test suite is run.
    

If any of this fails it's visible on the PR itself. Bitbucket can also be configured to prevent merging unless all builds pass, so that's a nice way to ensure broken code doesn't accidentally get merged in.

Hopefully this is helpful for anyone else that is using Bitbucket and Laravel, willfully or not!