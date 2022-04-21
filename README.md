# Hello World App Build Pipeline

This is a build pipeline for a container serving a simple Hello World page using Github + CircleCI.

This pipeline is translated from the manual deploy script (./local-deploy.sh) provided by the AllTrails interview team, which uses the following tools: Docker, AWS CLI, Kubectl, & Helm

More extensive info on the tool can be found in their repo located at https://github.com/alltrails/at-interviews-helloworld and specific description & requirements for this tool can be found at https://docs.google.com/document/d/1J3yn1rDOBdquOvLaYaEIAW7QLFDP9tdH/edit# . 

The build pipeline workflow is outlined as follows:

1.  A commit/change has been pushed to the Github repo and will trigger the pipeline-workflow to run
2.  Job build-and-deploy 
	- will checkout the code & spin up docker environment + remote environment
	- Install tools: AWS CLI, Kubectl, & Helm
	- Setup AWS Credentials
	- Update .kube config and login to Amazon ECR
	-  Build Docker container and push to Amazon ECR
	- and deploy container 

Failure reports are sent via email to all CircleCi team members.

#QUESTIONS

1.  How you would modify your pipe to accommodate for dev and prod environments?

	I made use of CircleCI’s contexts and created one for the DEV environment and the PROD environment.

	Each context contains the following environments variables that can be modified depending on the environment:

	AWS_ACCESS_KEY_ID
	AWS_ACCOUNT_NUMBER
	AWS_DEFAULT_REGION
	ASW_SECRET_ACCESS_KEY
	CLUSTER_NAME
	KUBE_PROFILE

	My workflow would then run the build-and-deploy job for the DEV and PROD contexts to deploy to each environment.


2.  What challenges you had, what you didn't have time for, and what you would change

- CHALLENGES

	--Debugging and Running ./local-deploy.sh

	The first major challenge for me was getting the local-deploy.sh to work on my MacBook. 
	One of the first things I like to do when automating manual processes is to get a very good understanding of how the manual process works. It makes automating the process easier if you understand each step. 
	I had initial issues with installing the required tooling, updating home-brew, running docker locally, and debugging issues with the script itself, specifically this error:

	Error: UPGRADE FAILED: unable to recognize "": no matches for kind "Ingress" in version "extensions/v1beta1”

	I tried my best, but I got to the point where I successfully deployed the container, but the kubectl commands specified would not generate an ATI application URL so I couldn’t actually see the hello world app live. 

	— Circle CI Onboarding/Setup
	
	The other major challenge was learning and using CircleCI at an accelerated pace w/ limited time. 
	I’m most familiar with the Jenkins CI but wanted to implement in CircleCI since that was what AllTrails uses currently. So some time cost was put into learning and setting up an instance for myself.

- DIDN’T HAVE TIME FOR

	- Getting deploy step to work and produce URL locally & in pipeline
	- Verify the URL works in browser
	- Translating the AWS Account verifications & Unique Namespace checks
	- Implementing manual hold/approval for deploy-and-build for PROD environment

- CHANGES

	—Use of Full SHA instead of shortened version
	I wasn’t able to figure out how to get the shortened version of the SHA outside of the git command used in local-deploy.sh but it made the code not as readable so opted to use CircleCI’s env var: CIRCLE_SHA1

	—Changes/upgrades to provided ingress.yaml
	I couldn’t get local-deploy.sh to work so I started debugging and figured out it was an issue with the ingress.yml found in  helm/helloworld/templates: 

	Error: UPGRADE FAILED: unable to recognize "": no matches for kind "Ingress" in version "extensions/v1beta1"

	I did some investigation and upgraded from extensions/v1beta1 to  networking.k8s.io/v1 as well as simplified   it down to bare bones. It doesn’t work perfectly since I can’t get the URL to get produced but the local-deploy.sh finishes successfully. I added the ingress.yml to this in case you’d like to check it out.
