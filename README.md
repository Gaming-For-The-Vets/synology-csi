# synology-csi [![Board Status](https://dev.azure.com/jparklab/c00a1581-5124-490c-b494-fffb5a31ce33/0babeaa0-b476-4461-846f-80933d0f614f/_apis/work/boardbadge/67a30810-7bb9-40b5-8e5b-ee592c85fdca?columnOptions=1)](https://dev.azure.com/jparklab/c00a1581-5124-490c-b494-fffb5a31ce33/_boards/board/t/0babeaa0-b476-4461-846f-80933d0f614f/Microsoft.RequirementCategory)

A Container Storage Interface Driver for Synology NAS 

## Build

### Build package

    make

### Build docker image

    # e.g. docker build -t jparklab/synology-csi .
    docker build -t <repo>[:<tag>] .

## Test

  Here we use [gocsi](https://github.com/rexray/gocsi) to test the driver, 

### Create a config file for testing

  You need to create a config file that contains information to connect to the Synology NAS API. See [Create a config file](#config) below

### Start plugin driver

    # You can specify any name for nodeid
    $ go run cmd/syno-csi-plugin/main.go \
        --nodeid CSINode \
        --endpoint tcp://127.0.0.1:10000 \
        --synology-config syno-config.yml 

### Get plugin info

    $ csc identity plugin-info -e tcp://127.0.0.1:10000

### Create a volume

    $ csc controller create-volume \
        --req-bytes 2147483648 \
        -e tcp://127.0.0.1:10000 \
        test-volume 
    "8.1"	2147483648	"iqn"="iqn.2000-01.com.synology:kube-csi-test-volume"	"mappingIndex"="1"	"targetID"="8"	

### List volumes

    The first column in the output is the volume D

    $ csc controller list-volumes -e tcp://127.0.0.1:10000 
    "8.1"	2147483648	"iqn"="iqn.2000-01.com.synology:kube-csi-test-volume"	"mappingIndex"="1"	"targetID"="8"	

### Delete the volume

    # e.g.
    # csc controller delete-volume  -e tcp://127.0.0.1:10000 8.1
    $ csc controller delete-volume  -e tcp://127.0.0.1:10000 <volume id>

## Deploy

### Ensure kubernetes cluster is configured for CSI drivers

   You can follow instructions on https://kubernetes-csi.github.io/docs/Setup.html to set up your kubernetes cluster for CSI drivers.

### Create a config file <a name='config'></a>

    ---
    # syno-config.yml file
    host: <hostname>        # ip address or hostname of the Synology NAS
    port: 5000              # change this if you use a port other than the default one
    username: <login>       # username
    password: <password>    # password
    sessionName: Core       # You won't need to touch this value
    sslVerify: false        # set this true to use https

### Create a k8s secret from the config file

    kubectl create secret generic synology-config --from-file=syno-config.yml

### Deploy to kubernetes

    kubectl apply -f deploy/kubernetes/v1.12

    (v1.13 has not been tested yet)

