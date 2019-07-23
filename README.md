# Introduction
On this meetup, we will walk you through creating a Jenkins X cluster on GKE, creating an application using a Jenkins X quickstart, pushing the application into a staging environment.

* [Requirements](#requirements)
* [Further Reading](#further-reading)
* [Setup](#setup)
  * [Option 1: Use the devops playground sandbox account](#option-1-use-the-devops-playground-sandbox-account)
  * [Option 2: Register for a GCP Trial account](#option-2-register-for-a-gcp-trial-account)
    * [Launch Cloud Shell](#launch-cloud-shell)
    * [Download Dependencies](#download-dependencies)
  * [Create a Github Access Token](#create-a-github-access-token)
* [Create Cluster](#create-cluster)
  * [What Just Happened?](#what-just-happened)
* [Create an Application](#create-an-application)
  * [What Just Happened?](#what-just-happened-1)
  * [Watch the build logs](#watch-the-build-logs)
  * [View the application](#view-the-application)
* [Make a change to the application](#make-a-change-to-the-application)
  * [Watch the build logs](#watch-the-build-logs-1)
  * [View Preview Environments](#view-preview-environments)
  * [Other useful commands](#other-useful-commands)
* [Cleanup](#cleanup)

# Requirements

* Basic Linux cli knowledge
* Basic Git knowledge
* Basic VI knowledge

# Further Reading
* GKE Documentation
* [Jenkins X Documentation](https://jenkins-x.io/documentation/)
* [Jenkins X GitHub](https://github.com/jenkins-x)
* [Tekton](https://github.com/tektoncd/pipeline/tree/master/docs)
* [Prow](https://github.com/kubernetes/test-infra/tree/master/prow)
* [Next Generation Pipeline](https://www.cloudbees.com/blog/move-toward-next-generation-pipelines)
* [Serverless Jenkins with Jenkins X](https://medium.com/@jdrawlings/serverless-jenkins-with-jenkins-x-9134cbfe6870)


# Setup

For this workshop we are going to use a Devops Playground sandbox VM to access GKE, but feel free to use your own GCP account if you'd prefer.

## Option 1: Use the devops playground sandbox account

Use the DevOps playground sandbox account.

## Option 2: Register for a GCP Trial account

Google offer a trial account with $300 of free credit.  You will require a Credit / Debit card to sign up but this will not be charged.  To sign up follow [these instructions](https://cloud.google.com/free/).

### Launch Cloud Shell

Navigate to the [Google Cloud Console](https://console.cloud.google.com/). Once you have logged in, click on the Activate Cloud Shell button in the top right of the screen.

![Activate Cloud Shell](/images/startcloudshell1.png)

This may take a few minutes if this is the first time you've launched cloudshell. Once complete, you should see the following terminal:

![Cloud Shell Terminal](/images/startcloudshell2.png)

### Download Dependencies

All of the required dependencies for this workshop will be downloaded & installed directly into Google Cloud Shell.

If you are using the Devops Playground sandbox account, you can skip this step.

```
curl -L https://github.com/jenkins-x/jx/releases/download/v2.0.500/jx-linux-amd64.tar.gz | tar xzv
sudo mv jx /usr/local/bin
```

Check that the `jx` binary is correctly installed

```
jx --version
$ 2.0.500
```
## Create a Github Access Token

Navigate to (https://github.com/settings/tokens/new?scopes=repo,read:user,read:org,user:email,write:repo_hook,delete_repo), Give the token a name and save this somewhere safe.  We will rmremove the token at the end of the workshop.

# Create Cluster

Ensure you are logged in:

```
$ gcloud auth login
```

If you are using the shared project, if you don’t have enough credit:

```
$ gcloud config set project u8cel62va-gcp-sandpit-803790 
```

Create a new empty GKE cluster:

```
$ jx create cluster gke --skip-installation --skip-login -n <your name>
```

e.g.

```
$ jx create cluster gke --skip-installation --skip-login -n user01
```

* If you are prompted for the GKE project, select the `u8cel62va-gcp-sandpit-803790`
* Select the zone that is nearest - I'd suggest one of the `europe-west` zones

We will default a number of options, to override this, use the `--advanced` flag when creating the cluster.

```
Defaulting to machine type: n1-standard-2
Defaulting to minimum number of nodes: 3
Defaulting to maxiumum number of nodes: 5
Defaulting use of preemptible VMs: No
Defaulting access to Google Cloud Storage / Google Container Registry: Yes
Defaulting enabling Cloud Build, Container Registry & Container Analysis API's: Yes
Defaulting enabling Kaniko for building container images: No
```

It can take around 5 minutes to create the cluster, once this has completed we can launch the `jx boot` process:

```
jx boot
```

Configure your gitconfig file

```
? Please enter the name you wish to use with git:  Gareth Evans
? Please enter the email address you wish to use with git:  gareth@bryncynfelin.co.uk
```

When prompted for the domain, select the default.

When prompted for the github credetials, enter your github username:

```
Creating a local Git user for GitHub server
? GitHub username: <my gh user>
To be able to create a repository on GitHub we need an API Token
Please click this URL https://github.com/settings/tokens/new?scopes=repo,read:user,read:org,user:email,write:repo_hook,delete_repo

Then COPY the token and enter in into the form below:

? API Token: ****************************************
```

When prompted for the environment information, select the defaults.

Select the JX namespace

```
jx ns jx
```

View all deployments

```
$ kubectl get deployments
```

e.g.

```
NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
crier                          1         1         1            1           6m21s
deck                           2         2         2            2           6m21s
hook                           2         2         2            2           6m21s
horologium                     1         1         1            1           6m21s
jenkins-x-chartmuseum          1         1         1            1           6m26s
jenkins-x-controllerbuild      1         1         1            1           6m26s
jenkins-x-controllerrole       1         1         1            1           6m26s
jenkins-x-controllerteam       1         1         1            1           6m25s
jenkins-x-docker-registry      1         1         1            1           6m25s
jenkins-x-heapster             1         1         1            1           6m25s
jenkins-x-nexus                1         1         1            1           6m24s
pipeline                       1         1         1            1           6m20s
pipelinerunner                 1         1         1            1           6m20s
plank                          1         1         1            1           6m19s
sinker                         1         1         1            1           6m18s
tekton-pipelines-controller    1         1         1            1           6m17s
tekton-pipelines-webhook       1         1         1            1           6m17s
tide                           1         1         1            1           6m18s
```

View all pods

```
kubectl get pods
```

e.g. 

```
NAME                                            READY   STATUS    RESTARTS   AGE
crier-749f96fb4d-7kqbp                          1/1     Running   0          5m42s
deck-696f77d746-jgrt6                           1/1     Running   0          5m42s
deck-696f77d746-l222d                           1/1     Running   0          5m42s
hook-6d9859bb47-qgnjz                           1/1     Running   0          5m42s
hook-6d9859bb47-x4xd6                           1/1     Running   0          5m42s
horologium-6bc57b5f9-ffgq5                      1/1     Running   0          5m41s
jenkins-x-chartmuseum-75d45b6d7f-dxsq8          1/1     Running   0          5m48s
jenkins-x-controllerbuild-6ddb75f89d-ms6s6      1/1     Running   0          5m47s
jenkins-x-controllerrole-5b6d489775-crtp6       1/1     Running   0          5m46s
jenkins-x-controllerteam-6c67c985cd-dkc6p       1/1     Running   0          5m46s
jenkins-x-docker-registry-6d555974c7-kxx7l      1/1     Running   0          5m47s
jenkins-x-heapster-6586795784-cx8rl             2/2     Running   0          3m36s
jenkins-x-nexus-6ccd45c57c-k2gtl                1/1     Running   0          5m46s
pipeline-5f85b8df5b-ft56g                       1/1     Running   0          5m41s
pipelinerunner-5546ffd8b-56n4g                  1/1     Running   0          5m40s
plank-8849d9d67-4tbbh                           1/1     Running   0          5m40s
tekton-pipelines-controller-687cfbcc89-wkf24    1/1     Running   0          5m39s
tekton-pipelines-webhook-7fd7f8cdcc-5f9rn       1/1     Running   0          5m38s
tide-5f8fb5964c-mpx5f                           1/1     Running   0          5m39s
```

View the Environments

```
kubectl get env
```

e.g.
 
```
NAME         NAMESPACE       KIND          PROMOTION   ORDER   GIT URL                                                                    GIT BRANCH
dev          jx              Development   Never               https://github.com/garethjevans-test/environment-gevans01-dev.git          master
production   jx-production   Permanent     Manual      200     https://github.com/garethjevans-test/environment-gevans01-production.git   master
staging      jx-staging      Permanent     Auto        100     https://github.com/garethjevans-test/environment-gevans01-staging.git      master
```

## What Just Happened?

* Created a kubernetes cluster on GKE
* Configured & Installed Jenkins X using GitOps
* Create two environments (Staging & Production)
* Using tektoncd for the pipeline execution engine
* No Jenkins CI

# Create an Application

Next we're going to use a `quickstart` to an application. To do this run the following:

```
jx create quickstart
```

e.g.

```
Using Git provider GitHub at https://github.com
? Do you wish to use garethjevans-bot as the Git user name? Yes


About to create repository  on server https://github.com with user garethjevans-bot
? Which organisation do you want to use? garethjevans-test
? Enter the new repository name:  gevans01-test


Creating repository garethjevans-test/gevans01-test
? select the quickstart you wish to create golang-http
Generated quickstart at /home/gevans/gevans01-test
### NO charts folder /home/gevans/gevans01-test/charts/golang-http
Created project at /home/gevans/gevans01-test

The directory /home/gevans/gevans01-test is not yet using git
? Would you like to initialise git now? Yes
? Commit message:  Initial import

Git repository created
performing pack detection in folder /home/gevans/gevans01-test
--> Draft detected Go (65.746753%)
selected pack: /home/gevans/.jx/draft/packs/github.com/jenkins-x-buildpacks/jenkins-x-kubernetes/packs/go

replacing placeholders in directory /home/gevans/gevans01-test
app name: gevans01-test, git server: github.com, org: garethjevans-test, Docker registry org: jenkins-x-workshop
skipping directory "/home/gevans/gevans01-test/.git"
Pushed Git repository to https://github.com/garethjevans-test/gevans01-test
Creating GitHub webhook for garethjevans-test/gevans01-test for url http://hook.jx.35.187.10.136.nip.io/hook
Watch pipeline activity via:    jx get activity -f gevans01-test -w
Browse the pipeline log via:    jx get build logs garethjevans-test/gevans01-test/master
You can list the pipelines via: jx get pipelines
When the pipeline is complete:  jx get applications
For more help on available commands see: https://jenkins-x.io/developing/browsing/
Note that your first pipeline may take a few minutes to start while the necessary images get downloaded!
```

## What Just Happened?

* Created a new source repository on GitHub
* Used code generation to create a new project
* Used language detection to select an appropriate buildpack for the project
* Configured a PR & master pipeline for that project
* Kicked off an initial master build
* Automatically pushed the application into the staging environment

## Watch the build logs

```
jx get build logs
```

If nothing is listed, try manually invoking the build with:

```
jx start pipeline
```

## View the application

```
jx get applications
```

e.g.

```
APPLICATION   STAGING PODS URL
gevans01-test 0.0.2   1/1  http://gevans01-test.jx-staging.35.187.10.136.nip.io
```

# Make a change to the application

```
$ git checkout -b wip
$ vi main.go
$ git add main.go
$ git commit -m "feat: updated welcome message"
$ git push origin wip
$ jx create pr
```

## Watch the build logs

```
jx get build logs
```

## View Preview Environments
```
$ jx get preview
```

## Other useful commands

```
$ jx get environments
$ jx get pipelines
$ jx start pipeline
$ jx get build logs -w
$ jx get activity
$ jx create quickstart -l Go
$ jx get build logs -f <project-name>
$ jx get activity -f <project-name>
$ jx get applications
$ jx promote my-app --version 0.0.1 --env production
```

# Cleanup

Delete the GKE cluster:

```
$ gcloud container clusters delete <cluster-name> --region europe-west1-b
```
Remove storage:

```
$ gsutil ls && gsutil -m rm -r gs://<cluster-name>-<bucket-name>
```
Remove service accounts:

```
$ gcloud iam service-accounts list && gcloud iam service-accounts delete <name>@<project>.iam.gserviceaccount.com
```

Cleanup the local configuration:

```
$ rm -rvf ~/.jx
```

You can also run the `jx gc gke` command to generate a script to clean up unused resources.
