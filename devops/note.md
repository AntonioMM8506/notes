## Deploy api with NestJS in Heroku.

Create app in heroku.
Add instance of Postgres

See the documentation https://devcenter.heroku.com/articles/heroku-cli#download-and-install

create an app with heroku.

heroku login
heroku create store-api
heroku addons:create heroku-postgresql:hobby-dev

Heroku add-ons are components that support your application, such as data storage, monitoring, analytics, data processing, and more.

now go to the app > settings > Reveal Config Vars. Heroku add a environment variable called DATABASE_URL, this variable contain the url connection to the data base.


GitLab configuration.

In the repository of the app go to the Settings > CI/CD > Variables. You should to configure two important variables HEROKU_API_KEY and HEROKU_APP_STAGING. HEROKU_API_KEY contain the key for the connection to the heroku instance and HEROKU_APP_STAGING the name of the app and a important configuration is change the flag of Protected to false.

Configuration of .gitlab-ci.yml

```
image: node:latest # version of the image

before_script: 
    - apt-get update -qy # update the packages from the repositories, q = Quiet. Produces output suitable for logging, omitting progress indicators. y = yes. Automatic yes to prompts.
    - apt-get install -y ruby-dev
    - gem install dpl # dlp is a deploy tool made for continuous deployment that’s developed and used by Travis CI, but can also be used with GitLab CI/CD. https://docs.gitlab.com/ee/ci/examples/deployment/

stages:
    - staging #Configuration of the stages of the deploy.

staging:
    type: deploy
    stage: staging
    image: ruby:latest
    script:
        - dpl --provider=heroku --app=$HEROKU_APP_STAGING --api-key=$HEROKU_API_KEY
        
    only: # specify the branch tha are listening for the deploy.
        - 27-deploy-in-staging-environment # name of the branch

```

configuration Procfile
```
web: npm run start:prod
```

scripts in package.json 
```
  "postinstall": "npm run prestart:prod",
  "prestart:prod": "rimraf dist && npm run build && npm run migration:run"
  "prebuild": "rimraf dist"
  "build": "nest build"
  "start:prod": "node dist/main"
  "migration:run": "ts-node node_modules/.bin/typeorm migration:run --config ./orm.config.js"
```

Configuration of file to connect data base.
orm.config.js

```
module.exports = {
  'type': 'postgres',
  'url': process.env.DATABASE_URL,
  'entities': [entitiesPath],
  'migrations': [migrationPath],
  'cli': {
    'migrationsDir': 'src/database/migrations'
  },
  'synchronize': true
}
```

.env file 

```
PORT=3000
PORT_DB=5444
JWT_SECRET=090909
ENVIRONMENT=PROD
DATABASE_URL=postgres://postgres:password@localhost:5444/persist
```