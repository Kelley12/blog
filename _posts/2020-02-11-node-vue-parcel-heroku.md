---
title: "Deploying my Node, Vue, Typescript, Parcel app to Heroku"
author: bkelley
layout: post
date: 2020-02-11 12:49:42 -0600
categories: posts
image: /assets/posts/heroku-deploy.png
description: >
  Trials and tribulations of deploying with new-to-me technology for the [Horsin-Around](https://www.github.com/kelley12/horsin-around) application.
---

- [The Setup](#the-setup)
  - [Backend](#backend)
  - [Frontend](#frontend)
  - [Testing, Logging, Development, CI/CD](#testing,-logging,-development,-ci/cd)
    - [Testing](#testing)
    - [Logging](#logging)
    - [Development](#development)
    - [CI/CD](#ci/cd)
- [The Dreaded Heroku Application Error](#the-dreaded-heroku-application-error)
- [Viewing Heroku Logs](#viewing-heroku-logs)
- [Cannot Find Module](#cannot-find-module)
- [Logging in to bash shell on Heroku](#logging-in-to-bash-shell-on-Heroku)
- [Manually Running The Build On Heroku](#manually-running-the-build-on-heroku)
- [No pg_hba.conf Entry](#no-pg_hba.conf-entry)
- [Changing Tactics](#changing-tactics)
- [Access Denied](#access-denied)
- [Not Found](#not-found)

## The setup

I created an app called [Horsin-Around](https://www.github.com/kelley12/horsin-around) for management of horse shows for a friend of mine's business. I created the app using the following technologies

### Backend

- [Node](https://nodejs.org/en/)
- [Express](https://expressjs.com/)
- [Typescript](https://www.typescriptlang.org/)
- [TypeORM](https://typeorm.io/)

### Frontend

- [Vue](https://vuejs.org/)
- [Parcel](https://parceljs.org/)

### Testing, Logging, Development, CI/CD

#### Testing

- [Mocha](https://mochajs.org/)
- [Chai](https://www.chaijs.com/)
- [Instanbul](https://istanbul.js.org/)

#### Logging

- [Winston](https://github.com/winstonjs/winston)
- [EventEmitter2](https://github.com/EventEmitter2/EventEmitter2)

#### Development

- [nodemon](https://github.com/remy/nodemon)

#### CI/CD

- [Travis](https://travis-ci.com/) (Linux)
- [Appveyor](https://ci.appveyor.com/) (Windows)
- [David](https://david-dm.org/)
- [Heroku](https://heroku.com/)

## The Dreaded Heroku Application Error

My application built and deployed without issue, but when visiting the appliction url, I experience the dreaded Heoku Application Error.

![Heroku Application Error](/assets/images/heroku-application-error.png "Heroku Application Error")

## Viewing Heroku Logs

First thing to do was to log into heroku command line

```bash
heroku login
```

Then display the tail end of the logs

```bash
heroku logs --tail -a APP-NAME
```

## Cannot Find Module

The tail end of the logs displayed `Error: Cannot find module '/app/dist/index.js'`. Obviously this meant that there was not an `index.js` file in `/app/dist/`.

Issue #1 was that I was trying to reference the wrong `index.js`, fixing this, I pushed my code to start the deploy again.

This time, I received the same error but for the `index.js` file I was trying to reference `Error: Cannot find module '/app/dist/server/index.js'`

## Logging in to bash shell on Heroku

To open up the bash terminal for my app, I used the `run bash` command with my app name:

```bash
heroku run bash -a APP-NAME
```

This allowed me to navigate the app folder structure and see what was there (or not there).

The bash terminal defaults to `/app` so I listed the files and folders in `/dist` to find that the `/dist/server` did not actually exist.

### Manually Running The Build On Heroku

I was able to run the build from the Heroku bash terminal.

```bash
export NODE_ENV=development
npm run build
```

After the build completed, `/dist/server` existed. Onto further investigation.

I realized that I had a `heroku-postbuild` script that was rebuilding post deploy but only running the parcel piece. Removing that script solved my problem. [Commit 75e1bae](https://github.com/Kelley12/horsin-around/commit/75e1baeb9554887d6ea5312c4c94c07243f678c4)

## No pg_hba.conf Entry

Once the [Cannot Find Module](#cannot-find-module) error was resolve, I ran into the next error:

`error: no pg_hba.conf entry for host "IP.ADDRESS", user "DB_USER", database "DATABASE_NAME", SSL off`

This is a PostgreSQL error stating that the Heroku Dyno that I was using did not have access. Having some customer websites hosted on Bluehost, I was attempting to use the Postgres databases there. However, BlueHost does not have many tools for handling access to the databases for remote hosts.

Even contacting support, they were unable to modify that access for me. This was a huge problem and required me to switch away from Postgres as my database provider.

Thank you [TypeORM](https://typeorm.io/)!

## Changing Tactics

Because Postgres is a problem when using BlueHost, I needed to switch to MySQL which has better support. Thanks to [TypeORM](https://typeorm.io/), this was relatively easy. It actually took longer to update the documentation than it was to update the code. Simply updating the Connection config, .env example, and some data types in the entities and it was switched over.

## Access Denied

After switching over to MySQL, I deployed and ran into another error:

`Error: ER_ACCESS_DENIED_ERROR: Access denied for user 'MySQL_User'@'ec2-111-222-333-444.compute-1.amazonaws.com' (using password: YES)`

This error is due to the host, `ec2-111-222-333-444.compute-1.amazonaws.com` in this case, does not have access to the MySQL database. Simply adding this host will not work as the dyno IP addresses change each time the app is started up.

The only way to solve this is one that I do not like but have no other option. I was even thinking about hosting the app and database on my own Raspberry Pi server to get away from this but that would be even less secure.

Another potential solution to this problem is to use a proxy so that all of the requests are routed through a static IP address. There are Heroku add ons such as [Fixie](https://elements.heroku.com/addons/fixie) or [QuotaGuard Static](https://elements.heroku.com/addons/quotaguardstatic). The problem is that they are limited to 250-500 requests/month in the free option. I anticipate more than this and am looking for free options. [@pierrickchabi](https://github.com/pierrickchabi) had a great solution for this in [mysql issue #725](https://github.com/mysqljs/mysql/issues/725).

For my solution, I needed to whitelist all `compute-1.amazonaws.com` EC2 services which is far from ideal but when the Heroku dyno could have an IP anywhere in the AWS IP range.

To do this, I had to add `%.compute-1.amazonaws.com` to the whitelist in Bluehost. After that, the application built and deployed properly.

## Not Found

With the app deploying and the DNS figured out, I was still running into an error with the app returning a blank page of `Not Found`. A 404 page really. This was not happening when I had the `heroku-postbuild` script in there, seemed to be rendering properly with that there.

When the `heroku-postbuild` script was in `package.json` the deployment would succeed, the html would render down to the most basic components, but the server wouldn't be able to start because the postbuild script was removing everything in the `/dist` folder and building parcel on top of it.

The problem came when I found that I didn't have an express route to the bundled frontend files. I added the following line to my express server

```javascript
this.app.use("/", express.static(`${__dirname}/../`));
```

This then properly routed to the frontend.
