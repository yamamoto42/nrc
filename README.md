# nrc - Node Red Container -
[![Docker Automated build](https://img.shields.io/docker/automated/yamamoto42/node-red-container.svg)](https://hub.docker.com/r/yamamoto42/node-red-container/)

This is static json data responser.

![sketch](https://github.com/yamamoto42/nrc/blob/master/img.png "sketch")

## Getting Started
1. Build container yourself

```
git clone https://github.com/yamamoto42/nrc.git
cd nrc
docker build -t nrc .
docker run -it -p 80:1880 --name mynrc nrc
```

2. Use DockerHub container

```
docker run -it -p 80:1880 --name mynrc yamamoto42/node-red-container
```

## Run nrc
1. Run

```
curl http://localhost/rapidservlet/rapid
```
2. Execution result

```
{"result":[{"category":"t-C01cat","confidence":0.52550542354584,"position":[-50,-50,50,50]},{"category":"t-C01cat","confidence":0.52564257383347,"position":[-25,-50,75,50]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[0,-50,100,50]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[25,-50,125,50]},{"category":"t-C01cat","confidence":0.5256906747818,"position":[-50,-25,50,75]},{"category":"t-C01cat","confidence":0.52552902698517,"position":[-25,-25,75,75]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[0,-25,100,75]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[25,-25,125,75]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[-50,0,50,100]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[-25,0,75,100]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[0,0,100,100]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[25,0,125,100]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[-50,25,50,125]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[-25,25,75,125]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[0,25,100,125]},{"category":"t-C01cat","confidence":0.52543026208878,"position":[25,25,125,125]}],"IMA_status":"0","all_result":{"category":["t-C01cat","t-C02doc"],"result":[{"position":[-50,-50,50,50],"confidence":[0.52550542354584,0.47449463605881]},{"position":[-25,-50,75,50],"confidence":[0.52564257383347,0.47435745596886]},{"position":[0,-50,100,50],"confidence":[0.52543026208878,0.4745697081089]},{"position":[25,-50,125,50],"confidence":[0.52543026208878,0.4745697081089]},{"position":[-50,-25,50,75],"confidence":[0.5256906747818,0.4743093252182]},{"position":[-25,-25,75,75],"confidence":[0.52552902698517,0.47447091341019]},{"position":[0,-25,100,75],"confidence":[0.52543026208878,0.4745697081089]},{"position":[25,-25,125,75],"confidence":[0.52543026208878,0.4745697081089]},{"position":[-50,0,50,100],"confidence":[0.52543026208878,0.4745697081089]},{"position":[-25,0,75,100],"confidence":[0.52543026208878,0.4745697081089]},{"position":[0,0,100,100],"confidence":[0.52543026208878,0.4745697081089]},{"position":[25,0,125,100],"confidence":[0.52543026208878,0.4745697081089]},{"position":[-50,25,50,125],"confidence":[0.52543026208878,0.4745697081089]},{"position":[-25,25,75,125],"confidence":[0.52543026208878,0.4745697081089]},{"position":[0,25,100,125],"confidence":[0.52543026208878,0.4745697081089]},{"position":[25,25,125,125],"confidence":[0.52543026208878,0.4745697081089]}]}}
```

## Modify application / Deploy Azure ACI
Chek [Azure Document](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-tutorial-prepare-app).
 This case use port 1880. if you change service port, use yaml file.

1. Modify Code

```
git clone https://github.com/yamamoto42/nrc.git
cd nrc
vi flows_test.json
```
Can edit with nodered and export JSON.

2. Build container instance

```
docker build -t nrc .
```

3. Create container registry

```
az login
az account set --subscription <subscription_id>
nrcrg="nrcrg"
nrcacr="nrcacr"
nrcloc="eastus"
nrcname="mynrc"
az group create --name $nrcrg --location $nrcloc
az acr create -g $nrcrg -n $nrcacr --sku Basic --admin-enabled true
```

4. Push container registry

Check AcrLoginServer name
```
az acr list -g $nrcrg --query "[].{acrLoginServer:loginServer}" --output table
```
Tagging AcrLoginServer (usualy $nrcacr.azurecr.io)
```
docker tag nrc:latest $nrcacr.azurecr.io/nrc:latest
```
Push Azure container registry
```
az acr login -n $nrcacr
docker push $nrcacr.azurecr.io/nrc:latest
```

5. Deploy to Azure ACI

Check registry-password
```
az acr credential show -n $nrcacr
```
Create and Deploy nrc
```
az container create \
-g $nrcrg \
-n $nrcname \
--image $nrcacr.azurecr.io/nrc:latest \
--cpu 1 \
--memory 1 \
--registry-username $nrcacr \
--registry-password <xxxxxxxxxxxxxxxxxxxxxx> \
--ports 1880 \
--dns-name-label $nrcname
```
6. Run

```
curl http://$nrcname.$nrcloc.azurecontainer.io:1880/rapidservlet/rapid
```
