# Install and run Custom Form Recognizer Feedback Loop containers

## Prerequisites

Before you use Custom Form Recognizer Feedback Loop containers, you must meet the following prerequisites:

|Required|Purpose|
|--|--|
|Docker Engine| You need the Docker Engine installed on a [host computer](#the-host-computer). Docker provides packages that configure the Docker environment on [macOS](https://docs.docker.com/docker-for-mac/), [Windows](https://docs.docker.com/docker-for-windows/), and [Linux](https://docs.docker.com/engine/installation/#supported-platforms). For a primer on Docker and container basics, see the [Docker overview](https://docs.docker.com/engine/docker-overview/).<br><br> Docker must be configured to allow the containers to connect with and send billing data to Azure. <br><br> On Windows, Docker must also be configured to support Linux containers.<br><br>|
|Familiarity with Docker | You should have a basic understanding of Docker concepts, such as registries, repositories, containers, and container images, and knowledge of basic `docker` commands.|
|The Azure CLI| Install the [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) on your host.|
|Form Recognizer resource |To use these containers, you must have:<br><br>An Azure **Form Recognizer** resource to get the associated API key and endpoint URI. Both values are available on the Azure portal **Form Recognizer** Overview and Keys pages, and both values are required to start the container.<br><br>**{FORM_RECOGNIZER_API_KEY}**: One of the two available resource keys on the Keys page<br><br>**{FORM_RECOGNIZER_ENDPOINT_URI}**: The endpoint as provided on the Overview page. 

# Create a Form Recognizer resource

When you're granted access to use Form Recognizer, you'll receive a Welcome email with several links and resources. Use the following ["Azure portal" link](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_formUnderstandingPreview#create/Microsoft.CognitiveServicesFormRecognizer) to open the Azure portal and create a Form Recognizer resource. In the **Create** pane, provide the following information:

|    |    |
|--|--|
| **Name** | A descriptive name for your resource. We recommend using a descriptive name, for example *MyNameFormRecognizer*. |
| **Subscription** | Select the Azure subscription which has been granted access. |
| **Location** | The location of your cognitive service instance. Different locations may introduce latency, but have no impact on the runtime availability of your resource. |
| **Pricing tier** | The cost of your resource depends on the pricing tier you choose and your usage. For more information, see the API [pricing details](https://azure.microsoft.com/pricing/details/cognitive-services/).
| **Resource group** | The [Azure resource group](https://docs.microsoft.com/azure/architecture/cloud-adoption/governance/resource-consistency/azure-resource-access#what-is-an-azure-resource-group) that will contain your resource. You can create a new group or add it to a pre-existing group. |

> Normally when you create a Cognitive Service resource in the Azure portal, you have the option to create a multi-service subscription key (used across multiple cognitive services) or a single-service subscription key (used only with a specific cognitive service). However, because Form Recognizer is a preview release, it is not included in the multi-service subscription, and you cannot create the single-service subscription unless you use the link provided in the Welcome email.

When your Form Recognizer resource finishes deploying, find and select it from the **All resources** list in the portal. Then select the **Keys** tab to view your subscription keys. Either key will give your app access to the resource. 

## Containers
Form Recognizer Feedback Loop includes the following three containers: 
* **Form Recognzier Feedback Loop container** - named custom-supervised container - trains models and analyzes forms to extract key value pairs - http://localhost:5005
* **Form Recognizer document OCR container** named readlayout container - an OCR container tailored for documents http://localhost:7005
* **Form Recognizer Feedback Loop Sample UX** - named custom-supervised-sampleux - Sample UX container for labeling, training and analyzing a form via a UX experience - http://localhost:3005

# Container requirements and recommendations

The minimum and recommended CPU cores and memory to allocate for the Form Recognizer containers host are described in the following table:

| Container | Minimum | Recommended |
|-----------|---------|-------------|
|cognitive-services-customforms-recognizer | 2 core, 8-GB memory | 8 core, 32-GB memory |

Each core must be at least 2.6 gigahertz (GHz) or faster.

In dockers --> Settings --> Advanced set your CPUs and Memeory to the CPU and Memeory settings above. For example: 

![alt text for image](media/docker.png)



## Use the Docker CLI to authenticate the private container registry

In order to login to docker repository please execute following command (please use the user ID and Password from the welcome e-mail):

```
docker login containerpreview.azurecr.io  -u <user ID>  -p <password>
```

### How to run the containers
### Run the containers by using the docker-compose up command

Use the [docker-compose up](https://docs.docker.com/compose/reference/up/) to pull and run the Form Recongizer Feedback Loop containers. The docker compose will run all the containers hosted locally on the same host, use the following example Docker Compose YAML file to run the containers. 

1. Replace `<FORMRECOGNIZER_APIKEY>`with the subscription key from the form Recognizer Resource that you created. This key is used to start the container. It's available on the Azure portal **Form Recognizer Keys** page.
2. Replace `<FORMRECOGNIZER_BILLING_ENDPOINT>` with the endpoint URL for the Form Recognizer resource in the Azure region where you obtained your subscription keys. It's available on the Azure portal **Form Recognizer Overview** page.
 
Replace these parameters with your own values in the following example `docker-compose` command.
  

```bash
version: '3'
services:

  # Form Recognizer Feedback Loop Sample UX container http://localhost:3005
  custom-supervised-sampleux:
    image: containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer-custom-supervised-sampleux:latest
    ports:
      - 3005:3000
    command: eula=accept
      
  # Form Recognizer Feedback Loop container http://localhost:5005
  custom-supervised:
    image: containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer-custom-supervised:latest
    environment:
      apikey: <FORMRECOGNIZER_APIKEY>
      billing: <FORMRECOGNIZER_BILLING_ENDPOINT>
      CORS__AllowedOrigins: "*"
      CustomFormRecognizer__ComputerVisionApiKey: <FORMRECOGNIZER_APIKEY>
      CustomFormRecognizer__ComputerVisionEndpointUri: http://readlayout:5000/formrecognizer/v2.0-preview/readLayout/asyncAnalyze
    links:
      - readlayout
    volumes:
      - ./Mounts/output:/output
      - ./Mounts/input:/input
    ports:
      - 5005:5000
    command: eula=accept
      
  # ReadLayout container http://localhost:7005
  readlayout:
    image: containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer-readlayout:latest
    environment:
      apikey: <FORMRECOGNIZER_APIKEY>
      billing: <FORMRECOGNIZER_BILLING_ENDPOINT>
      CORS__AllowedOrigins: "*"
    ports:
      - 7005:5000
    command: eula=accept
```

Save the code in a file with a .yaml extension e.g docker-compose.yaml. 
In order to run the docker compose please execute the following command form a command line window:

```
docker-compose up
```

The docker compose excutes the following: 
* Pulls Custom Form Recognizer Feebdack Loop containers images from the container registry.
* Exposes TCP port 5005, 7005, and 3005 and allocates a pseudo-TTY for the container.
* Automatically removes the container after it exits. The container image is still available on the host computer.
* Mounts an /input and an /output volume to the container Form Recognizer Feedback Loop container.

### Container's endpoints

Once deployed following are the containers endpoints: 

|Container|Endpoint|
|--|--|
|custom-supervised container|http://localhost:5005|
|readlayout conatainer|http://localhost:7005|
|Sample UX container|http://localhost:3005

## Billing

The Custom Form Recognizer containers send billing information to Azure by using the _Form Recognizer_ resource on your Azure account. 

## Summary

In this article, you learned how to run the Form Recognizer Feedback Loop Private Preview containers. In summary:

* Form Recognizer provides three container for Docker.
* Container images are downloaded from the private container registry in Azure.
* Container images run in Docker.
* You can use either the Sample UX or REST API to call operations in Form Recognizer containers by specifying the host URI of the container.
* You must specify the billing information when you instantiate a container.

### Next Steps

Once deployed start labeling, training and extracting key value pairs from forms using the Form Recgonizer Feedback Loop feature. Try it now at http://localhost:3005, follow the [Quickstart: Label forms, train a model and analyze a form using the Form Recognizer Feedback Loop Sample UX](sampleUX-label-train-analyze.md) to get started.
Explore the Form Recognizer custom-supervised container API [here](http://localhost:5005/swagger) and explore the Form Recognizer read layout container API [here](http://localhost:7005/swagger)

>  Cognitive Services containers are not licensed to run without being connected to Azure for metering. Customers need to enable the containers to communicate billing information with the metering service at all times. Cognitive Services containers do not send customer data (for example, the image or text that is being analyzed) to Microsoft.
