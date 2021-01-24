 
 # Philips Hue c8y Agent

Cumulocity micro service agent to connect and play with your Philips Hue bridge and lights.
Developed using the [Software AG webMethods Micro Service Runtime](https://hub.docker.com/_/softwareag-webmethods-microservicesruntime) and available as a docker image.

This agent uses the [Philips MeetHue API](https://developers.meethue.com)

**Installation**

This source code is a webMethods Micro Service Runtime package and you will need to first install the Software AG Micro Service Runtime or download a docker image.

**local installation**

If you have an Integration Server or Micro Service Runtime running locally for development purposes, first navigate to your packages directory;

*$ cd /${SAG_HOME}/IntegrationServer/packages*  
or  
*$ cd /${SAG_HOME}/IntegrationServer/instances/${INSTANCE}/packages*  

If your packages directory is already under version control

*$ git submodule add https://github.com/johnpcarter/c8yPhilipsHueAgent.git c8yPhilipsHueAgent*

or if you are not, then simply clone the repository

*$ git clone https://github.com/johnpcarter/c8yPhilipsHueAgent.git*  

Then download dependent packages

*$ git clone https://github.com/johnpcarter/c8yConnector.git*  
*$ git clone https://github.com/johnpcarter/JcPublicTools.git*  

Then restart your runtime server and refresh your package browser in Designer.

**Docker Installation**

A predefined Dockerfile template has been provided in the c8yConnector packages resources directory. It is recommended that you copy the directory
and then update Dockerfile and aclmap_sm.cnf file appropriately.

cd into your directory and download the latest source code

*$ cd ${WORKING_DIR}*  
*$ git clone https://github.com/johnpcarter/c8yPhilipsHueAgent.git*  
*$ git clone https://github.com/johnpcarter/c8yConnector.git*  
*$ git clone https://github.com/johnpcarter/JcPublicTools.git*  

copy the permissions file from the c8yPhilipsHueAgent into your directory

$ cp ./C8yPhilipsHueAgent/resources/aclmap_sm.cnf .  

Update the docker file to include the c8yPhilipsHueAgent package by copying the following line into the section 'add YOUR packages here'

**ADD --chown=sagadmin ./c8yPhilipsHueAgent /opt/softwareag/IntegrationServer/packages/c8yPhilipsHueAgent* 

You can now build your image

*$ docker build .*  

**Preparing the Micro Service zip file for your tenant**

You will need to export the newly build image as tar file and then zip it with the cumulocity.json file before uploading to your tenant

*$ docker save -o image.tar ${YOUR DOCKER IMAGE}*  
*$ zip ${c8y_app_name} cumulocity.json image.tar*

**Uploading the Micro Service agent to your Cumulocity tenant**

Login in to your Cumulocity tenant and select the "Administration" app, then from the left hand side menu "Applications", "Own Applications" and then click "Add application". Choose "Upload microservice"and select the zip that you created before.

Once uploaded, subscribe to the app in order to start it up and activate it.

For more information refer to the Cumulocity documentation on [Micro Service Runtime](https://cumulocity.com/guides/microservice-sdk/concept/#microservice-runtime)

**Starting up the agent remotely**

You can also use the agent from your own development environment or run the docker image outside of your tenant if you wish.
You will need to set the bootstrap credentials or set the c8y user in your local environment.

From your local environment, run the service '' and either provide your own credentials via the C8Y_TENANT, C8Y_USER and C8Y_PASSWORD inputs 
or you want to use the app credential provide the C8Y_BOOSTRAP_TENANT, C8Y_BOOTSTRAP_USER and C8Y_BOOTSTRAP_PASSWORD inputs.

If you want to run the docker image indepenently then set the following envioronment variables when starting up the container

*-e C8Y_BOOTSTRAP_TENANT='*'*
*-e C8Y_BOOTSTRAP_USER*
*-e C8Y_BOOTSTRAP_PASSWORD*

You can obtain your bootstrap credentials via the following api call

*$ curl "https://<TENTANT_NAME>.cumulocity.com/application/applications/<APP_ID>/bootstrapUser" \
 -u '<YOUR USER>:<YOUR PASSWORD>'*


**Remote authentication required by Philips Hue**
 
 You will need to use the following settings to obtain an oauth 2.0 token for your Hue bridge.
  
 *authorization: https://api.meethue.com/oauth2/auth?appid=cumulocity_device_manager&deviceid=c8y&device_name=c8y&response_type=code*
 
 *access: https://api.meethue.com/oauth2/token*
  
 You can register for a client id/secret via the developer site above and also find documentation on how to authenticate and obtain a token
 
 [Remote API Quick Stark Guide](https://developers.meethue.com/develop/hue-api/remote-api-quick-start-guide)
 
**Device Onboarding**

  Once you have your ouath 2.0 token, you can begin the onboarding process for the bridge and its attached lights and switches.
  
  1) [get /bridge/{name}/definition](./#/bridge/9627a8b7-1332-401b-878e-9fa641f418e4) to query the bridge and obtain a managed object
  that represents the bridge device that can be posted to c8y.
  2) [post /bridge/{c8yIdOfBridge}/provision](./#/bridge/6060f35b-3d59-402f-a3df-19df2edece0b) to create managed objects for
  all of the lights and switches attached to the bridge. The agent on successful completion will start sending metrics and listening for operations.
  
**Uploading to Cumulocity**
  Refer to [Micro Service SDK Guide](https://cumulocity.com/guides/microservice-sdk/introduction/) to see how to upload this agent to your
  Cumulocity tenant
  
**Device Models**

  This agent also defines a managed object of type '**c8y_Device_Agent**' in Cumulocity. This models allows the iOS Cumulocity Device Management app
  to provision Hue Bridges without having to hard-code this API. The OAuth 2.0 client id and secret are registered using the type '**philips-api**'. 
  You can add your own credentials to Cumulocity by posting an equivalent object to Cumulocity. The iOS app will allow the user to choose between them.
  
  Device models and images for Philips hue assets are also registered via the types '**device_Supplier**' and '**device_Model**' respectively.
