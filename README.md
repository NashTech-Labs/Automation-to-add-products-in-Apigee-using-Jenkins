# Products in Apigee

In Apigee, API products in Apigee are a core concept of the Apigee API management platform. They represent bundles or collections of APIs that are grouped together and made available to developers for consumption. API products define the access, usage limits, and capabilities of the APIs included in the product.

## Create/Update Products in Apigee using Jenkins

This repository contains Jenkinsfile which has script for creating/updating products in Apigee environments.

For using this template we have to update our Apigee organisation name and credentials in jenkinsfile for parameter:

`def orgList = ['APIGEE_ORGANISATION_NAME']`

`credentialsId: '<gcp_service_account>'`


Create a Jenkins pipeline using this repository and build that pipeline by passing parameters to it. You can select multiple envs at the same time in parameter. Mention here the name of api product file.


![jenkins_pipeline](https://i.postimg.cc/kGwY3sdQ/Screenshot-from-2023-07-15-17-20-03.png)


Build the pipeline and it will create or update product in Apigee envs.

To verify this move to:

Apigee > Publish > API Products
