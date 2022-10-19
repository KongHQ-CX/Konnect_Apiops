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
KONG_PROXY_EU_URL: Kong Proxy EU Endpoint
KONG_PROXY_US_URL: Kong Proxy US Endpoint
```
**Also replace the runtime group name in the kong_CI.yml github workflow or add a Konnect_ApiOps runtime group in Konnect!**

```Get Dev Portal EU ID``` job retrieves the Developer Portal Id for the European Instance.

```Dev Portal EU change``` job pushs the change to the Developer Portal Id for the European Instance using the spec format json file located in the portal directory.  

```Validate Specification``` job uses inso CLI to do automated linting. If these is a problem with the api specification, it will stop the build process.  

```Generate declarative config``` job generates the Konnect yaml config file. It creates first a Kong yaml file and then convertes into a 3.0 spec format.

```Sync to Konnect``` job synchronises the Konnect yaml with Konnect. 

```Prepare URL 4 Test file``` job replaces the  ```base_url``` in test spec file with the KONG_PROXY_EU_URL.

At the end we use inso CLI to run the test cases we built in Insomnia and use the KONG_PROXY_EU_URL to check everythig has been correctly deployed.
