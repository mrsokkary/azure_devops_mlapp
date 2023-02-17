# Building a CI/CD Pipeline to Azure App Service
 
## Introduction

You will develop a Github repository from scratch as part of this project, as well as a scaffolding that will help you carry out Continuous Integration and Continuous Delivery. An initial lint, test, and install cycle will be carried out using Github Actions, a Makefile, requirements.txt, and application code. To allow continuous delivery to the Azure App Service, you must next integrate this project with Azure Pipelines.


## Project Plan
* Link to Trello's board for the project - https://trello.com/invite/b/IWvJcQ4K/ATTI457ef57f868643c02b1e21b2ea27e50e16A49E9B/udacityproject
* Link to a spreadsheet   https://docs.google.com/spreadsheets/d/1hMzmB58AuYoqAk0b0jgEOlYW_f8XmWiO/edit?usp=share_link&ouid=106091926904970901003&rtpof=true&sd=true

## Pre requisites

* An Azure Account (https://portal.azure.com/)
* A GitHub Account (http://github.com/)
 

## First: CI Pipeline with GitHub Actions

## Instructions

### Architectural Diagram - CI - GitHub Actions

![Architectural Diagram](./screenshots/ci-diagram.png)

### Instructions

* Enter to Azure Portal with your Account and open a Azure Cloud shell ( use Bash)

> If you are not created one , just follow the first creation and wait a seconds to get your Cloud Shell.

* First of all set up SSH Keys in your azure cloud shell, add the `id_rsa.pub` key to your GitHub repo ( ssh keys)  and then clone the project there.

```sh
ssh-keygen -t rsa -C "youremailaddress"
cat ~/.ssh/id_rsa.pub
```

![cloning repository](./screenshots/screen%201%20-%20cloning%20repo.png)

* Project Scaffolding ( files):
 
Create a virtual environment for your application.

```sh
python3 -m venv ~/.myrepo
source ~/.myrepo/bin/activate
```

* Run `make all` which will install, lint, and test code

```sh
make all
```

![test passed after make all](./screenshots/screen%202%20-%20test%20passed%20after%20make%20all.png)

* Next set up Github Actions in your repo doing this :

> just create new Github Actions new workflow (Python application template sample)

![workflow build](./screenshots/screen%203%20-%20github%20actions%20workflow%20build%20-%20OK.png)

![workflow summary](./screenshots/screen%203a%20-%20githubactions-workflow-created-summary.png)

Note: it was  created a simple Github Actions in the branch  which automates the lint  and test of our app.
Note: You can add the GitHub Actions badge to you README.md ( it's a good practice to do that)

## Second : CI/CD Pipeline with AZURE DEVOPS

### Azure DevOps Pipelines Architectural Diagram

![CD Diagram](./screenshots/cd-diagram.png)

* Go to Azure Devops page  and sign in it, create a new Project inside your organization ( if you don't have an organization create one first).

* In your new Project in Azure DevOps, go to Project Settings and create a new `Project settings --> Pipelines --> Service connections` as is explained on the YouTube video link  below ( Service Connection must be of Type Azure Resource Manager)

> Note 1 : Name your Service Connection `AzureServiceConnection`
> Note 2: Use a link of as this `https://dev.azure.com/<organization>/<project_name>/_apis/serviceendpoint/endpoints?api-version=5.0-preview.2`  to find your ServiceConnectionID ( take note of this number since you will needed in the yaml file to build the pipeline). Replace the values for the ones that you created for your organization and project name.
Note3: the ServiceConnection ID is the number before the name `AzureServiceConnection` of the above link.

* Create the webapp deploying the code from the local workspace to Azure app Service ( using Plan B1)  with this command:

```sh
az webapp up -n <your-appservice> 
```

![webapp created](./screenshots/screen%204a%20-%20webapp%20created.png)
![webapp created on Azure](./screenshots/screen%204b%20-%20webapp%20created.png)

>Note 1: Your service URL  will be something like this : `https://<name of webapp>.azurewebsites.net/`
>Note 2 : the name of your app must be unique!!!
>Note 3: This operation can take a while to complete ...

* In  your new Project in Azure DevOps, go to Pipelines --> New Pipeline --> GitHub --> Select Your Repo --> select `an Existing YAML file`

![creating azure pipeline](./screenshots/screen%205%20-%20creating%20azure%20pipeline.png)

> Choose the `main` branch and the file named `azure-pipelines.yml` as is showed on the figure below
> Update the `azure-pipelines.yml` with the name of your webapp and your Service connection point ( Check YouTube video for a detailed explanation)
> Modifications are at variables webAppName & environmentName too !!!

![existing yaml file](./screenshots/screen%205a%20-%20existing%20yaml%20file.png)

* Choose Run Pipeline and your Azure DevOps Pipeline is going to start to be deployed with all his stages ( in this case 2: Build & deploy)

![pipeline executing](./screenshots/screen%206%20-%20azure%20devops%20pipeline%20executing.png)

* Perform a cosmetic change to your app.py , so you can see your CI/CD pipelines in action on Azure DevOps ( CD) & GitHub Actions (CI)

> On file `app.py` change this line:

```sh
html = "<h3">Sklearn Prediction Home</h3>"
```

for this one:

```sh
html = "<h2>Sklearn Prediction Home APP - REST API</h2>"
```

and then perform a quick lint and push the changes to your repo:

```sh
make lint
git add .
git commit -m "app.py updated"
git push origin main
```

![Azure devops pipeline integrated](./screenshots/screen%206b%20-%20azure%20devops%20pipeline%20integrated%20to%20repo.png)

![Azure devops deploy pipeline successfully](./screenshots/screen%206c%20-%20azure%20devops%20deploy%20pipeline%20successfully.png)

* Check that the webapp is running opening this URL, example:

```sh
https://mywebapp216240.azurewebsites.net/
```

![webapp running](./screenshots/screen%207%20-%20webapp%20running.png)

* Update the file `make_predict_azure_app.sh` with the webapp service end point. Assign execute permission for this file

```sh
chmod +x make_predict_azure_app.sh
```

* When the Azure DevOps pipeline is successfully deployed, then its time to make a prediction on our webapp ( running in Azure App Service):

```sh
./make_predict_azure_app.sh
```

Answer:

```sh
Port: 443
{"prediction":[20.35373177134412]}
```

![make azure prediction](./screenshots/screen%208%20-%20make%20azure%20prediction.png)

* Logging: Check on webapp log tail `az webapp log tail --name <app name> --resource-group <resource group name>` , the answer like this:

![alt text](./screenshots/screen%209%20-%20webapp%20log%20tail.png)

* Execute a Load Testing , running Locust .

> Note: Azure cloud Shell is not enough good to perform locust there, so use a VM or your own local machine ( windows, macos, linux) to run Locust
> Note: copy both files (`loadtesting.sh` & `locustfile.py`) and run the `loadtesting.sh` file and open the browser on `http://localhost:8089/` :

```sh
./loadtesting.sh
```

>from there you can see how the load testing is performing and how you app established on a # of RPS
> This is good to know how good is your webapp and your plan to manage the requests. So you can decide to scale up the plan service of your webapp for example.
-> Locust could generate a stats report.

![load testing using locust](./screenshots/screen%2010%20-%20locust%20report.png)
![locust charts](./screenshots/screen%2010%20-%20locust%20charts.png)
 
 
## YouTube Demo
  
