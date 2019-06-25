:title: api

.. _api:

API
===

.. contents:: Table of Contents
    :local:

PiCli will POST a zipfile containing a set of files and a run_vars.yml configuration file.

This run_vars.yml file specifies the Pipe configuration for the remote function to use. Inside this file
there will be an ``options`` dictionary which can be consumed by the function to modify the default options
that the third-party tool that the function wraps is called with

Runvars Spec
************

.. jsonschema:: specs/runvars-schema.yaml

Runvars Example
***************

An example run_vars.yml file may look like this:

.. code-block:: yaml

  ---
  ci:
    ci_provider: gitlab-ci
    ci_provider_config:
      before_script:
        - echo `pwd`
        - echo "$CI_BUILD_NAME, $CI_BUILD_REF_NAME $CI_BUILD_STAGE"
        - echo "$CI_JOB_NAME"
        - echo "$CI_JOB_ID"
        - echo "$SERVICE_VERSION"
      build:
        artifacts:
          paths:
            - build/reports/tests/**
          reports:
            junit: build/reports/**/*.xml
        script:
          - git checkout ${GIT_BRANCH}
          - git pull origin master
          - python3 -m venv env
          - source ./env/bin/activate
          - pip install flask pytest-xdist pytest-cov nose coverage requests pylint
          - source ./env/bin/activate
          - pytest --junitxml=build/reports/tests/test_results.xml test
          - coverage run --source=source -m pytest test
          - coverage report
          - coverage xml
        stage: build
      cache:
        key: $CI_JOB_NAME-$CI_COMMIT_REF_SLUG
      generate_docker_image_push_to_nexus:
        script:
          - docker build -t ${DOCKER_REGISTRY_AT_NEXUS}/${APP_NAME}:${SERVICE_VERSION}
            --build-arg VERSION=${SERVICE_VERSION} --build-arg VERSION=${SERVICE_VERSION}
            -f docker/Dockerfile .
          - docker images
          - docker login -u ${DOCKER_REGISTRY_USER} -p ${DOCKER_REGISTRY_PASSWORD} ${DOCKER_REGISTRY_AT_NEXUS}
          - docker push ${DOCKER_REGISTRY_AT_NEXUS}/${APP_NAME}:${SERVICE_VERSION}
          - oc login --insecure-skip-tls-verify ${OCP_URL} --token=${OCP_TOKEN}
          - oc project ${OCP_PROJECT}
          - oc tag -n ${OCP_PROJECT} --insecure=true --reference-policy='local' ${DOCKER_REGISTRY_AT_NEXUS}/${APP_NAME}:${SERVICE_VERSION}
            ${OCP_IMAGE_STREAM}:release
        stage: generate_docker_image_push_to_nexus
      image: python:3.6
      include:
        - file: .gitlab-ci.yml
          project: piperci/validation
      stages:
        - validate
        - lint
        - build
        - generate_docker_image_push_to_nexus
      variables:
        APP_NAME: demo-os-python-app
        DOCKER_REGISTRY_AT_NEXUS: $DOCKER_REGISTRY_AT_NEXUS
        DOCKER_REGISTRY_PASSWORD: $DOCKER_REGISTRY_PASSWORD
        DOCKER_REGISTRY_USER: $DOCKER_REGISTRY_USER
        MAJOR_VERSION: '2'
        MINOR_VERSION: '0'
        OCP_IMAGE_STREAM: ${APP_NAME}-${OCP_PROJECT}
        OCP_PROJECT: $OCP_PROJECT
        OCP_TOKEN: $OCP_TOKEN
        OCP_URL: $OCP_URL
        SERVICE_VERSION: ${MAJOR_VERSION}.${CI_PIPELINE_ID}.${CI_JOB_ID}
        SONAR_LOGIN: $SONAR_LOGIN
        SONAR_URL: $SONAR_URL
  file_config:
    - file: test.sh
      sast: noop
      styler: noop
    - file: piperci.d/pi_global_vars.yml
      sast: noop
      styler: noop
    - file: piperci.d/default_vars.d/pipe_vars.d/pi_validate.yml
      sast: noop
      styler: noop
    - file: piperci.d/default_vars.d/pipe_vars.d/pi_sast.yml
      sast: noop
      styler: noop
    - file: piperci.d/default_vars.d/pipe_vars.d/pi_style.yml
      sast: noop
      styler: noop
    - file: piperci.d/default_vars.d/file_vars.d/src_config.yml
      sast: noop
      styler: noop
    - file: piperci.d/default_vars.d/group_vars.d/python_lint.yml
      sast: noop
      styler: noop
    - file: piperci.d/default_vars.d/group_vars.d/all.yml
      sast: noop
      styler: noop
    - file: charon/functional.py
      sast: noop
      styler: flake8
    - file: charon/scanner.py
      sast: noop
      styler: flake8
    - file: charon/worker.py
      sast: noop
      styler: flake8
    - file: charon/cloud.py
      sast: noop
      styler: flake8
    - file: charon/config.py
      sast: noop
      styler: flake8
    - file: charon/__init__.py
      sast: noop
      styler: flake8
    - file: charon/util.py
      sast: noop
      styler: flake8
    - file: charon/notification.py
      sast: noop
      styler: flake8
    - file: charon/charon_config.txt
      sast: noop
      styler: noop
    - file: charon/model/schema.py
      sast: noop
      styler: flake8
    - file: charon/model/base.py
      sast: noop
      styler: flake8
    - file: charon/model/__init__.py
      sast: noop
      styler: flake8
    - file: charon/cloudutils/base.py
      sast: noop
      styler: flake8
    - file: charon/cloudutils/__init__.py
      sast: noop
      styler: flake8
    - file: charon/cloudutils/openstack.py
      sast: noop
      styler: flake8
    - file: charon/notifications/base.py
      sast: noop
      styler: flake8
    - file: charon/notifications/__init__.py
      sast: noop
      styler: flake8
    - file: charon/notifications/mattermost.py
      sast: noop
      styler: flake8
    - file: charon/scanners/base.py
      sast: noop
      styler: flake8
    - file: charon/scanners/nessus.py
      sast: noop
      styler: flake8
    - file: charon/scanners/__init__.py
      sast: noop
      styler: flake8
    - file: charon/api/app.py
      sast: noop
      styler: flake8
    - file: charon/api/__init__.py
      sast: noop
      styler: flake8
  pi_global_vars:
    ci_provider: gitlab-ci
    project_name: python_project
    vars_dir: default_vars.d
    version: 0.0.0
  pi_sast_pipe_vars:
    run_pipe: true
    url: http://172.17.0.1:8080/function
    version: latest
  pi_style_pipe_vars:
    run_pipe: true
    url: http://172.17.0.1:8080/function
    version: latest
  pi_validate_pipe_vars:
    policy:
      enabled: true
      enforcing: true
      version: 0.0.0
    run_pipe: true
    url: http://172.17.0.1:8080/function
    version: latest
