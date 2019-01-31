# rancher2-upgrade

## About

This fork is for those of you who dislike using `:latest`. It will update your deployment with a tag you've pushed, instead of just just forcing an image pull.

Allows you to upgrade existing deployments in Rancher 2 in your CI process.
This image will use the Rancher CLI to change an environment variable, which 
will cause kubernetes to re-pull the docker image.
ai-rancher2-upgrade would not be necessary if the Rancher 2 CLI would allow
specifying the project ID. In this case it would be a 2-liner to upgrade a 
service. But since the Rancher 2 CLI currently has no non-interavtive mode, it
will ask the user to select a project if there is more than just the default project. This
image picks up the work-around suggested in https://github.com/rancher/rancher/issues/14448.
It writes a cli2.json file based on environment variables which contains the 
authentication data and project it.

## Usage

1. Specify the following environment variables (e.g. in Gitlab > CI/CD > Variables"):

- RANCHER2_ACCESS_KEY (From Rancher 2 "API & Keys")
- RANCHER2_SECRET_KEY (From Rancher 2 "API & Keys")
- RANCHER2_SERVER_URL (The URL of your Rancher 2 deployment)
- RANCHER2_PROJECT_ID ("View in API" next to project, then the property "id")
- REGISTRY_USERNAME (Your Gitlab username)
- REGISTRY_PASSWORD (Your Gitlab password)

2. Run the docker-entrypoint.sh (gitlab will not do this automatically). This will create the
cli2.json

3. Execute "upgrade.sh [namespace] [deployment] [image]" script for each deployment you want to upgrade.

Sample gitlab-ci.yml:

```yaml
variables:
  CONTAINER_IMAGE_NAME: registry.gitlab.com/myproject/repo_name:$CI_COMMIT_TAG # Inject the git tag you've just pushed

build:
  stage: build
  only:
    - tags
  script:
    - docker login -u $REGISTRY_USERNAME -p $REGISTRY_PASSWORD registry.gitlab.com
    - docker build --pull -t $CONTAINER_IMAGE_NAME .
    - docker push $CONTAINER_IMAGE_NAME

deploy to test:
  stage: deploy
  image: kaivi/rancher2-upgrade:v2.0.0
  variables:
    GIT_STRATEGY: none
  only:
    - develop
    - tags
  script:
    - /usr/local/bin/docker-entrypoint.sh # Creates the cli2.json with authentication data
    - /usr/local/bin/upgrade.sh default website $CONTAINER_IMAGE_NAME # Triggers the actual upgrade
```

## Development

This project is hosted at https://github.com/kaivi/rancher2-upgrade

The docker image is hosted on dockerhub at https://hub.docker.com/r/kaivi/rancher2-upgrade