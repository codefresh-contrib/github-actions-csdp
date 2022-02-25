# github-actions-csdp
GitHub Action Jobs for CSDP 

Use Case: Send GitHub Action's workflow CI data to CSDP platform.

While Codefresh Software Delivery Platform supports both Continuous Integration and Continuous Deployment we do not require both to take advantage of the many features of the CSDP platform.

We can work with existing CI platforms and handle just the application delivery and still provide you with an amazing set of values to track, audit and facilitate your application deliveries.

I will go how to configure the jobs in GitHub Actions that will relay information about your applications CI process and GIT data to CSDP.

## Prereqs

Firstly, you will need to locate 2 API keys that will be required to post data to the CSDP API.

### Runtime API Key

1. If you haven’t already please install the Codefresh Runtime to your Kubernetes cluster. 
1. From your terminal window run the following command to get the 1st needed API key which will be referred to going forward as CF_RUNTIME_KEY.
 ``` sh 
 $ kubectl get secret -n <runtime-namespace> codefresh-token -o=jsonpath='{.data.token}' | base64 --decode
 ```
Save this key for later.

### Platform API Key

1. Open your browser to https://g.codefresh.io/user/settings to generate the required CF_PLATFORM_KEY.
1. Scroll down to the API Keys section.
![This is an image](https://raw.githubusercontent.com/codefresh-contrib/github-actions-csdp/main/images/platform_api_key_creation_1.png)
1. Click the GENERATE button.
![This is an image](https://raw.githubusercontent.com/codefresh-contrib/github-actions-csdp/main/images/platform_api_key_creation_2.png)
1. Save this key for later.

Now that we have the required keys we will need to add them into GitHub as Secrets for our Actions.

### GitHub Actions Secrets

1. Open your browser to your GitHub Repository.
1. Click on the Settings Tab [If you do not see this tab you may not have the required permissions, please check with your GitHub Administrator]
1. From the left menu, find the Security section.
1. Expand Secrets.
1. Select Actions.
1. Click the New repository secret button. ![This is an image](https://raw.githubusercontent.com/codefresh-contrib/github-actions-csdp/main/images/github_actions_secrets.png)
1. Enter in the CF_RUNTIME_KEY and the value from the output of the Kubernetes command.
1. Click the Add Secret button.
1. Click the New repository secret button.
1. Enter in the CF_PLATFORM_KEY and the value from the Codefresh browser window.
1. Click the Add Secret button.

You’re done adding the required keys and now we can create the jobs required in your GitHub Action’s workflow.

## CSDP Jobs

Below are 3 jobs.

Job 1: Report Docker image to CSDP (Creates the Docker image metadata identity. Reports GIT and Docker information to CSDP).

This job requires read access to your Docker registry and you’ll need a username/password for that later when we’re editing the workflow YAML.  You likely already have the password saved as a secret in the GitHub Action’s Secrets.

Jobs 2 and 3 require that job 1 completes successfully.  

They are responsible for further enriching the image metadata identity with data at the correct time.

Job 2: Report JIRA information to CSDP (Parses the GIT branch to extract the JIRA ticket information and forwards that information to CSDP).

This job requires a JIRA user’s email and an API key for the user.  In the examples you’ll find below this API key is stored in the GitHub Action’s Secret JIRA_API_TOKEN.

Instructions on generating this JIRA API token in Atlassian site: https://support.atlassian.com/atlassian-account/docs/manage-api-tokens-for-your-atlassian-account/

Job 3: Report GitHub Pull Request information to CSDP (On pull request events, extracts the pull request data dn forwards that information to CSDP).

Now that we understand what each Job is doing depending on our use case we may eliminate some of the Jobs that are necessary from our GitHub Action’s workflow.

Adding Jobs to your GitHub Action’s workflow.

1. Open your IDE to workflow’s YAML file. [Can also be done from GitHub’s site]
2. Any time after you’ve pushed your Docker image to the Docker registry add the CSDP Jobs. (As mentioned you could remove the JIRA or PR job as they are not required if you find they are not applicable to your use case.)

## Workflow YAML

A simple example workflow is available in the file ./.github/workflows/docker-ci.yaml

You will need to copy the Jobs from the YAML and look for replacements highlighted by `< VALUE >`.

#### [Report Docker image to CSDP](https://github.com/codefresh-contrib/github-actions-csdp/blob/main/.github/docker-ci.yaml#L25-L55)
#### [Report JIRA information to CSDP](https://github.com/codefresh-contrib/github-actions-csdp/blob/main/.github/docker-ci.yaml#L57-L82)
#### [Report GitHub Pull Request information to CSDP](https://github.com/codefresh-contrib/github-actions-csdp/blob/add-blog-files/.github/docker-ci.yaml#L84-L105)

When you are finished you will be able to run the pipeline which will then populate the image entity to CSDP.

https://g.codefresh.io/2.0/images

![CSDP Image Listing](https://raw.githubusercontent.com/codefresh-contrib/github-actions-csdp/main/images/images_listing.png)

![CSDP Image Details](https://raw.githubusercontent.com/codefresh-contrib/github-actions-csdp/main/images/image_details.png)

DISCLAIMER:

There are two links for images that are currently not functioning that are in our engineering back log.  

See the image below for the two links.

![This is an image](https://raw.githubusercontent.com/codefresh-contrib/github-actions-csdp/main/images/missing_details.png)

We will also be adding in the future an Annotations Job that you will be able to incorporate into your GitHub Actions workflow which will allow you to add custom metadata to the image.  As an example, URLs to third party systems for security scanning reports.