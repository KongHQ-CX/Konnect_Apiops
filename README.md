# Konnect APIOps demo

This repository contains Konnect_ApiOps_Example project created with Insomnia and a workflow to build and deploy services, routes and plugins in Konnect.

## Konnect_ApiOps_Example
You can use insomnia to directly import this project from git repository. After importing the project, You can find the API specification in the Design tab. This is a simple api which returns an echo message and rate.

If you want to test the API, you can use the debug tab and call the Konnect_ApiOps_Example. Please update the environment to point to the correct proxy endpoint. In localhost you can use Ngrok.

In order to create test cases for your api, you can use the Test tab in Insomnia. In this project, we have created a simple test case for HTTP 200 response code.

```
const response1 = await insomnia.send();

expect(response1.status).to.equal(200);
```

We will run this test case as part of our Git Actions workflow to make sure, api deployment was successful.

## GitHub Actions
I have created a simple workflow to build and deploy Kong Services, routes and plugins. This workflow starts with checking out the the code, creating a backup from customer api specification and installing nodejs, deck and inso CLI.

Please add the following to Settingg-->Secrets configuration in your repository:
```
KONNECT_TOKEN : Konnect Token
```

Also replace in the Github action flow yaml  
```
SERVICE_NAME: name of the service, could be the Insomnia Project Name
KONG_PROXY_EU_URL: Kong Proxy EU Endpoint
KONG_PROXY_US_URL: Kong Proxy US Endpoint
RUNTIME_GROUP: the Konnect Runtime group Name
REGIONS: Konnect region
```

** Make sure you added the Konnect_ApiOps runtime group in Konnect!**

```Backup Insomnia Spec [region]``` job creates a backup file for the specific region, services and tags

```Validate Specification``` job uses inso CLI to do automated linting. If these is a problem with the api specification, it will stop the build process.  

```Generate declarative config``` job generates the Konnect yaml config file. It creates first a Kong yaml file and then convertes into a 3.0 spec format.

```Update Konnect [region]``` job synchronises the Konnect yaml with Konnect. 

```Prepare URL 4 Test file``` job replaces the  ```base_url``` in test spec file with the KONG_PROXY_EU_URL.

```Run test suites for [region]``` job use inso CLI to run the test cases we built in Insomnia and use the KONG_PROXY_URL to check everythig has been correctly deployed.

```Get spec json``` job retrieves information about the openApiSpec files

```Get spec information for [region]``` job retrieves the Developer Portal Id, Version ID and Service Package ID for the European Instance.

```Post specs for [region] service version``` job adds the spec format json file to the service version.  

```Publish [region] Service Package``` job pushs the change to the Developer Portal Id for the European Instance

```fallback for [region]``` job uses the backup file to restor the state in case a job failed during the workflow 

```success``` job confirmes that the workflow successfully run

