---
title: Azure
badge: devops
---

## Submiting Docker builds to Azure

### Pre-required steps

#### Create a resource group

First you need a resource group to contain all your stuff. If you don't know which regions are available to you, just use the `az account list-locations -o table` command. Then create the group as follows:

```shell
az group create --name {{ resource-group-name $RESOURCE_GROUP_NAME }} --location {{ location ukwest }}
```

#### Create a container registry

```shell
az acr create --resource-group {{ resource-group-name $RESOURCE_GROUP_NAME }} --name {{ acr-name $ACR_NAME }} --sku Standard --location {{ location ukwest }}
```

You can check  your created container registries with `az acr list`:

```shell
az acr list -o {{ output-format table }}
```

### Build in Azure with ACR Tasks

You can submit files to be built on azure. This way you don't require a Docker host [^1]:

[^1]: You can follow Azure's [official tutorial](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-tutorial-quick-task)

```shell
az acr build --registry {{ acr-name $ACR_NAME }} --resource-group {{ resource-group-name $RESOURCE_GROUP_NAME }} --image {{ repository $REPOSITORY_NAME }}:{{ tag latest }} .
```

You can list the uploaded images as follows:

```shell
az acr repository show-tags --name {{ acr-name $ACR_NAME }} --repository {{ repository $REPOSITORY_NAME }} --resource-group {{ resource-group-name $RESOURCE_GROUP_NAME }}
```

## Deploying a container as a Web Application


![01](azure/deploy-container/deploy_container_01.png)

![02](azure/deploy-container/deploy_container_02.png)

![03](azure/deploy-container/deploy_container_03.png)

![04](azure/deploy-container/deploy_container_04.png)

![05](azure/deploy-container/deploy_container_05.png)

![06](azure/deploy-container/deploy_container_06.png)

![07](azure/deploy-container/deploy_container_07.png)

![08](azure/deploy-container/deploy_container_08.png)

![09](azure/deploy-container/deploy_container_09.png)

![10](azure/deploy-container/deploy_container_10.png)

![11](azure/deploy-container/deploy_container_11.png)

![12](azure/deploy-container/deploy_container_12.png)

![13](azure/deploy-container/deploy_container_13.png)

Finally add the `WEBSITES_PORT` setting with a value of `8080`:

![14](azure/deploy-container/deploy_container_14.png)

![15](azure/deploy-container/deploy_container_15.png)

Remember to hit the **Save** button:

![16](azure/deploy-container/deploy_container_16.png)

![17](azure/deploy-container/deploy_container_17.png)

Finally you might want to restart the container to make sure everything was reseted properly and browse the application. Take in mind that It takes a while to start running for the first time:

![18](azure/deploy-container/deploy_container_18.png)


## Redeployment

Now when you want to update the application, the only command to use is the following:


```shell
az acr build --registry {{ acr-name $ACR_NAME }} --resource-group {{ resource-group-name $RESOURCE_GROUP_NAME }} --image {{ repository $REPOSITORY_NAME }}:{{ tag latest }} .
```