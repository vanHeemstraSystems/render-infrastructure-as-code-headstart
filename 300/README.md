# 300 - Building Our Application

Based on "Getting Started" at https://render.com/docs/infrastructure-as-code

## 100 - Getting Started
___

1) Create a Blueprint Spec (a file called ```render.yaml```) at the root of your repo. You can start with [our sample Blueprint](https://render.com/docs/blueprint-spec) and modify it to match your services.

```
services:
  # A Docker web service
  - type: web
    name: webdis
    env: docker
    repo: https://github.com/render-examples/webdis.git # optional
    region: oregon # optional (defaults to oregon)
    plan: standard # optional (defaults to starter)
    branch: master # optional (uses repo default)
    dockerCommand: ./webdis.sh # optional (defaults to Dockerfile command)
    numInstances: 3 # optional (defaults to 1)
    healthCheckPath: /
    envVars:
      - key: REDIS_HOST
        fromService:
          type: redis
          name: lightning
          property: host # available properties are listed below
      - key: REDIS_PORT
        fromService:
          type: redis
          name: lightning
          property: port
      - fromGroup: conc-settings
  # A private Minio instance
  - type: pserv
    name: minio
    env: docker
    repo: https://github.com/render-examples/minio.git # optional
    healthCheckPath: /minio/health/live
    envVars:
    - key: MINIO_ROOT_PASSWORD
      generateValue: true # will generate a base64-encoded 256-bit secret
    - key: MINIO_ROOT_USER
      sync: false # placeholder for a value to be added in the dashboard
    - key: PORT
      value: 10000
    disk:
      name: data
      mountPath: /data
      sizeGB: 10 # optional
  # A Ruby web service
  - type: web
    name: sinatra
    env: ruby
    repo: https://github.com/renderinc/sinatra-example.git
    scaling:
      minInstances: 1
      maxInstances: 3
      targetMemoryPercent: 60 # optional if targetCPUPercent is set
      targetCPUPercent: 60 # optional if targetMemory is set
    buildCommand: bundle install
    startCommand: bundle exec ruby main.rb
    domains:
      - test0.render.com
      - test1.render.com
    envVars:
      - key: STRIPE_API_KEY
        value: Z2V0IG91dHRhIGhlcmUhCg
      - key: DB_URL
        fromDatabase:
          name: elephant
          property: connectionString
      - key: MINIO_ROOT_PASSWORD
        fromService:
          type: pserv
          name: minio
          envVarKey: MINIO_ROOT_PASSWORD

    autoDeploy: false # optional
  # A Python cron job that runs every hour
  - type: cron
    name: date
    env: python
    schedule: "0 * * * *"
    buildCommand: "true" # ensure it's a string
    startCommand: date
    repo: https://github.com/render-examples/docker.git # optional
  # A background worker that consumes a queue
  - type: worker
    name: queue
    env: docker
    dockerfilePath: ./sub/Dockerfile # optional
    dockerContext: ./sub/src # optional
    branch: queue # optional
  # A static site
  - type: web
    name: my blog
    env: static
    buildCommand: yarn build
    staticPublishPath: ./build
    pullRequestPreviewsEnabled: true # optional
    headers:
      - path: /*
        name: X-Frame-Options
        value: sameorigin
    routes:
      - type: redirect
        source: /old
        destination: /new
      - type: rewrite
        source: /a/*
        destination: /a
  # A Redis instance
  - type: redis
    name: lightning
    ipAllowList: # required
      - source: 0.0.0.0
        description: everywhere
    plan: free # optional (defaults to starter)
    maxmemoryPolicy: noeviction # optional (defaults to allkeys-lru)

databases:
  - name: elephant
    databaseName: mydb # optional (Render may add a suffix)
    user: adrian # optional
    ipAllowList: # optional (defaults to allow all)
      - source: 203.0.113.4/30
        description: office
      - source: 198.51.100.1
        description: home

  - name: private database
    databaseName: private
    ipAllowList: [] # only allow internal connections


envVarGroups:
  - name: conc-settings
    envVars:
      - key: CONCURRENCY
        value: 2
      - key: SECRET
        generateValue: true
      - key: USER_PROVIDED_SECRET
        sync: false
  - name: stripe
    envVars:
      - key: STRIPE_API_URL
        value: https://api.stripe.com/v2
```
sample Blueprint render.yaml file
___

2) Click **Blueprints** in the navigation sidebar in the dashboard (https://dashboard.render.com/).

___

3) Click the **New Blueprint Instance** button.

___

4) Select the repo and branch containing the ```render.yaml``` file. Here we will be using this repo (https://github.com/vanHeemstraSystems/render-infrastructure-as-code-headstart).
___

5) Once selected, you???ll see a list of the changes that will be applied based on the contents of ```render.yaml```. If there???s an issue with the file you???ll see an error message. If everything looks good, click **Apply** to create the resources defined in your file.
___

You???re all set! Future updates to your ```render.yaml``` will be synced automatically and we???ll notify you if there are any issues with the syncs.
