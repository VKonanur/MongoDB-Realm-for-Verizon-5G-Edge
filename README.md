# MongoDB Realm for Verizon 5G Edge
> Getting Started Guide for MongoDB users to deploy Realm on Verizon's 5G Edge zones

The power of the mobile edge compute (MEC) resides in part in its ability to provide local computing with low latency, by having mobile clients use the nearest MEC node to handle application traffic. Most notably, Verizon in partnership with AWS has deployed several of these MEC zones, called the Wavelength zones in the AWS console, that offer
all of the well-known AWS resources like EC2, EKS, and ECS to deploy next-generation applications.
In this tutorial, we will go over deploying the much-adored object storage service Mongo Realm, the local database instance onto the MEC to power apps that desire low-latency storage and response times.


## Pre-requisites
To complete this exercise, you will need:
- A MongoDB account. To get one you can go to [cloud.mongodb.com](https://cloud.mongodb.com) and register. This execrise can be completed in the free-tier without needing to input any payment information
- An AWS account that is opted-in to Wavelength zones. The opt-in can be done from the AWS console instantly and is free-of-cost.

## Step 1: Create an Atlas Cluster and Realm app in the Mongo Cloud Portal
From the Mongo cloud console, we will first start with creating an organization. When doing so, ensure that you're choosing Atlas instead of Cloud Manager ([follow this tutorial](https://docs.atlas.mongodb.com/tutorial/manage-organizations/)) option. After the organization is created, create a project in the organization and provide it with requisite perimissions (ex: Project Owner) for your users. Once the project is created, 
create an Atlas cluster for the cloud database. The cluster is fully managed by Mongo and is currently available in two zones in the AWS free tier: us-east-1 and us-west-2, which are the regions in which AWS Wavelength zones in partnership with Verizon are located.
The next step is to create the Realm app on the cloud console by navigating to the Realm tab in your projects - you can keep the default configuration for this exercise. 
[include Realm app snapshot]


Turn on Realm sync, which enables write of Realm DB onto the Atlas cluster automatically. Now, navigate to the Realm you have created / using for this execise and 
[include Realm Sync functionality snapshot]. under Data Access on the left hand pane, go to App Users and add in an API user. This will generate a key token that will be used in the next step, where the locally deployed Realm on the Wavelength will use it to communicate back to the cloud Realm/Atlas cluster. 
Under the same `Data Access` tab, go to Schema and input the Realm object schema. While the format is standard, the below schema posted is for this current exercise. 

```
{
  "title": "IOTDataPoint",
  "bsonType": "object",
  "required": [
    "_id",
    "+pk",
    "reading"
  ],
  "properties": {
    "_id": {
      "bsonType": "objectId"
    },
    "+pk": {
      "bsonType": "string"
    },
    "_pk": {
      "bsonType": "string"
    },
    "device": {
      "bsonType": "string"
    },
    "reading": {
      "bsonType": "long"
    }
  }
}
```
> Realm object schema for this tutorial

Create Organization (if not yet), create new project -- 
choose Atlas Mongo Clusters. 
Create Realm App. 
Create API Key. Enable Realm Sync.

## Step 2: Deploy Docker container for Realm DB on the Wavelength zone instance. 
Once the MongoDB cloud elements have been provisioned and the API key for the Realm app have been created, we are ready to deploy the Realm database via a docker container on the Wavelength zone. 
The instructions below mirror for a Linux system, however, using the Docker application, it can also be replicated on Windows as well. 

```
git clone https://github.com/graboskyc/MQTTtoRealm.git &&
cd MQTTtoRealm.git`
```
`cp sample.env .env`

Input the private API key and the Realm App ID that you provisioned in the cloud portal in the `.env` file. 

Run `./build.sh` which will start the docker container for you using the above variables. Right now, the a local Realm database is setup on the Wavelength zone, in addition to having a MQTT listener to get data from a producer on Verizon's 5G network. 

## Step 3: Setup a data producer over a device on the 5G network. 
This an MQTT producer that will act as the synthetic data generator to send data to the Realm DB deployed on the Wavelength zone, which inturn will sync information to in the cloud portal and onto the Atlas cluster of choice.
The instructions below are for a Linux system, however, similar to the above, however, can be executed over a Windows system on the Docker application. 

```
git clone https://github.com/graboskyc/MQTTProducer.git && 
cd MQTTProducer.git
```

Here, update the `.env` file to include the carrier IP address of the Wavelength instance, your Realm App ID, and the topic name.

```
BROKER= "Carrier-IP"
CLIENTID="Realm App ID"
TOPIC="IOTDataPoint"
```

Run `./build.sh` script to start the container. At this point, go to your cloud console, in your project, open up the requisite Realm app and the data points from this producer should've started populating.


## Resources
To learn more, visit the 5G Edge Documentation to learn more.

Any questions? Reach out to verizon5gedge[at]verizon.com or create an issue on the repository, type a title and description, and provide your error message for reproducibility.
