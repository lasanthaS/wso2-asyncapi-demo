# use streaming-integrator-base-image
ARG BASE_IMAGE
FROM ${BASE_IMAGE}
MAINTAINER WSO2 Docker Maintainers "dev@wso2.org"

ARG HOST_APPS_DIR=./siddhi-files
ARG SAMPLES_DIR=${WSO2_SERVER_HOME}/samples/artifacts/
ARG APPS=${WSO2_SERVER_HOME}/deployment/siddhi-files
ARG CONFIG_FILE=./configurations.yaml
ARG CONFIG_FILE_PATH=${HOME}/configurations.yaml

# Copy jars
COPY libs/kafka_2.12_2.3.0_1.0.0.jar ${WSO2_SERVER_HOME}/lib/
COPY libs/kafka_clients_2.3.0_1.0.0.jar ${WSO2_SERVER_HOME}/lib/
COPY libs/metrics_core_2.2.0_1.0.0.jar ${WSO2_SERVER_HOME}/lib/
COPY libs/scala_library_2.12.8_1.0.0.jar ${WSO2_SERVER_HOME}/lib/
COPY libs/zkclient_0.11_1.0.0.jar ${WSO2_SERVER_HOME}/lib/
COPY libs/zookeeper_3.4.14_1.0.0.jar ${WSO2_SERVER_HOME}/lib/

# copy bundles & jars to the siddhi-runner distribution
COPY ${HOST_APPS_DIR}/ ${APPS}

# # copy deployment.yaml with service catalog configs
COPY deployment.yaml ${WSO2_SERVER_HOME}/conf/server/