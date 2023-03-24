# Konnect APIOps demo

This repository contains Konnect_ApiOps_Example project created with Insomnia and a workflow to build and deploy services, routes and plugins in Konnect.


## ApiOPs

APISecOps is short for API design, security, and operations. The solution centers around the four core fundamentals:

- **Centralization:** Centralize API operations and inventory to a single control plane. Irrespective of cloud provider or platform, all APIs can be managed together.
- **Governance:** A governance team should be enabled to define custom policy-as-code to evaluate API specifications and make sure they meet security standards.
- **API Design-First:** Development teams should design the API upfront before any code is written in order to align with governance — and business stakeholders. This is key as it is the entry point to all the automation, and the documentation.
- **GitOps:** The API specification itself — which serves as documentation, governance, and API administration — should all be handled via GitOps best practices for speed, resiliency, and reliability in the process.


The diagram below provides the 10,000-foot view of an APISecOps solution.

![Alt text](/images/apiOps.png "Title")


Remember that there are a number of questions you need to ask yourself before adapting an ApiOps stratagy:

- What level of collaboration/autonomy do you want to enable in your organization?

    - Product API teams / LoB should not be involved in CP/DP management. They will interface with their own Workspace.
    - Product API teams / LoB should not be involved in CP/DP management. They will have dedicated DPs and will interface via their own Workspace.
    - Product API teams / LoB should not be involved in CP management. They will take care of managing their own DPs and will interface via their own Workspace.
    - Product API teams / LoB will manage their own CP/DP.

- How many environments are necessary?
    - DEV
    - QA
    - PROD

- How do you split responsibilities between Platform teams and LoB/API teams in the following areas? Centralised/Decentralised/Mixed?
    - Authentication
    - Traffic Control
    - Observability (monitoring, logging, tracing)


Other questions to ask:
- one repo or several repo
- should dev should be aware of kong
- one or different environment by RG or Global?
- What moddel: federation, global or ApiOps?
- Push process: PR, Review, Commit? 
- What is the API development tool of choice for design/test/mocking?
- What API Spec standards do you use?
- Is a developer portal required? Global or per team?
- What is your source control system?
- What is your continuous integration server?
- What is your artifact server?
- How do you run automated testing?
- How many services
- How many teams and Runtime Group


## Prerequisite

This repo is an example on how a customer could leverage Insomnia and Konnect to create a Federation model where team could deploy their own api in Konnect.

They LoB will have dedicated Runtime Groupe and will not be involved in CP/DP management. They will interfer with Konnect to manage their RG configuration

Structure for the RunTime Group Team A:
```
Repo Service A
	Github
		flow
			kong_main_CI.yml
			kong_pr_CI.yaml
	Config
		kongGlobal.yaml
		kongTeams.yaml
	Service
		ApiSpecServiceA.yaml

----

Repo Service B
	Github
		flow
			kong_main_CI.yml
			kong_pr_CI.yaml
	Config
		kongGlobal.yaml
		kongTeams.yaml
	Service
		ApiSpecServiceB.yaml
```

In this repo in the config folder you can find 2 files:
- Global-config.yaml: global configuration enforce for all RG
- Team-config.yaml: let developer handle their plugins using extensions

**Do notice that in future Composite Runtimes Group will allow to maintain some common settings and configuration for a group of RG**

The Governance team will provide the:
- The KongGlobal.yaml 
- The Authorization tokens with only access to the team RG


## Repository Demo
In this model, the approach begins with API Design-First where the developer clone this github template, import the template in Insomnia, builds the API spec (and test suite) in Insomnia, using OpenAPI Spec (OAS) best practices and security standards defined by the business.

A Pull Request on the main branch will action the Github Flow that will, if succesful, deploy the changes on the Dev Runtime Group and environment.
After testing, merging the Pull Request will deploy the changes on the Production Runtime group. A failure check is setup in the flow so any failure will generate a rollback on previous version. 

### GitHub Actions
I have created a simple workflow to build and deploy Kong Services, routes and plugins. This workflow starts with checking out the the code, creating a backup from customer api specification and installing nodejs, deck and inso CLI.


## Installation

### 1) Git Clone the repo
Create a new repository using the repo template:
https://github.com/KongHQ-CX/Konnect_Apiops

Then
```
git clone https://github.com/[repo]
```

In your repo, we recommand protecting the main branch so it could only accept Pull Request and not direct Push.

### 2) Add the Service Account Token
In your repo please add the Service Account token provided by the governance team that will be used by the 2 flows. The current variable name is ```KONNECT_TOKEN```, please change it if needed.

Go to Setting --> Secrets configuration in your repository:
```
KONNECT_TOKEN : Konnect Service Account Token
```

![Alt text](/images/secret.png "Title")

Follow steps in this document to create the Service Account Token:
https://docs.konghq.com/konnect/org-management/system-accounts

**Make sure the token has only accessed to the Team Runtime Group**

### 3) Import the repo and Konnect_ApiOps_Example project in Insomnia
![Alt text](/images/inso.png "Title")

You can use insomnia to directly import this project from git repository. After importing the project, You can find the API specification in the Design tab. This is a simple api which returns an echo message and rate.

If you want to test the API, you can use the debug tab and call the Konnect_ApiOps_Example. Please update the environment to point to the correct proxy endpoint. In localhost you can use Ngrok.

In order to create test cases for your api, you can use the Test tab in Insomnia. In this project, we have created a simple test case for HTTP 200 response code.

```
const response1 = await insomnia.send();

expect(response1.status).to.equal(200);
```

We will run this test case as part of our Git Actions workflow to make sure, api deployment was successful.


### 4) Open the repo with your IDE (ie: Visual Studio Code)
Start you new micro service, before pushing to your service control system (Github, Gitlab...) make sure to replace the following variable in the 2 workflows"

```SERVICE_NAME``` The service name that will also appear in the Dev Portal, recommend to use the same name that your Insomnia Project Name

```KONG_PROXY_EU_URL``` The Kong DP proxy url for EU instance. Make sure to use the Production one for the Main flow and the Test url for the PR flow 

```KONG_PROXY_US_URL``` The Kong DP proxy url for US instance. Make sure to use the Production one for the Main flow and the Test url for the PR flow

```RUNTIME_GROUP``` The Runtime group Name provide by the Governance team

```REGIONS``` The Konnect regions list to deploy the service (US, EU or both)

**Make sure you added the Konnect_ApiOps runtime group in Konnect!**

### 5) Prepare the Pull Request
Create your service, OpenApiSpecs and Unit testing.

### 6) Create a Pull Request
Push the change to your repo. The Pull Request will trigger the kong_pr_CI flow. If succesful try make a call to the Dev Proxy URL.

### 7) Merge the Pull Request
Merhe the change from the Pull Request to the Main branch. The Pull Request will trigger the kong_main_CI flow. If succesful try make a call to the Prod Proxy URL.


## Jobs description Actions

```Backup Insomnia Spec [region]``` job creates a backup file for the specific region, services and tags

```Validate Specification``` job uses inso CLI to do automated linting. If these is a problem with the api specification, it will stop the build process.  

```Generate declarative config``` job generates the Konnect yaml config file. It creates first a Kong yaml file and then convertes into a 3.0 spec format.

```Update Konnect [region]``` job synchronises the Konnect yaml with Konnect. 

```Prepare URL 4 Test file``` job replaces the  ```base_url``` in test spec file with the KONG_PROXY_EU_URL.

```Run test suites for [region]``` job use inso CLI to run the test cases we built in Insomnia and use the KONG_PROXY_URL to check everythig has been correctly deployed.

```Get spec json``` job retrieves information about the openApiSpec files

```Get spec information for [region]``` job retrieves the Developer Portal Id, Version ID and Service Package ID for the European Instance.

```Post specs for [region]``` job adds the spec format json file to the service version.  

```Publish [region] Service Package``` job pushs the change to the Developer Portal Id for the European Instance

```fallback for [region]``` job uses the backup file to restor the state in case a job failed during the workflow 

```success``` job confirmes that the workflow successfully run

