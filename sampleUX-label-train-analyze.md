# Quickstart: Label forms, train a model and analyze a form using the Form Recognizer Feedback Loop Sample UX 

In this quickstart, you'll use the Azure Form Recognizer Sample UX to label forms, train and analyze forms to extract key-value pairs.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites
To complete this quickstart, you must have:
- Access to the Form Recognizer limited-access preview. To get access to the preview, fill out and submit the [Form Recognizer access request](https://aka.ms/FormRecognizerRequestAccess) form.
- [Python](https://www.python.org/downloads/) installed (if you want to run the sample locally).
- A set of at least 11 forms of the same type (same form structure). You will use this data to train the model and test a form. Sample dataset (zip file) to try out can be downloaded from [here](sample-data.zip). Unzip the sample data and upload 10 forms from the sample data to your blob container in a folder.  
- Form Recognizer Feedback Loop containers installed and running ([how to install and run the Form Recognizer Feedback Loop container](Install-Container.md))

## Label your forms

First, you'll need to label your forms and define which values you would like to extract from these forms. 

### Input Data
Form Recognizer feedback loop private preview requires the input data to be stored in folders per form type on a blob container. For example if source data has 3 form types – invoice 1, invoice 2, invoice 3 the folder structure should be have 3 folders one for each type. Place 10 forms from the sample data downloaded from [here](sample-data.zip) on your blob in a folder or bring your own 10 forms from the same type and place them on a blob container in a folder. These forms will be used for labeling and training a model. 

**Configuring Cross Domain Resource Sharing (CORS)**

Enable CORS on your storage account. Select your storage account in the Azure portal, click on CORS, add line below and click Save:
a.	ALLOWED ORIGINS = *
b.	ALLOWED METHODS = SELECT ALL
c.	ALLOWED HEADERS = *
d.	EXPOSED HEADERS = *
e.	MAX AGE  = 200


![alt text for image](media/Cors.png)

### Create Connection
Form Recognizer Feedback Loop Sample UX is a 'Bring Your Own Data' (BYOD) application. In the Sample UX, connections are used to configure and manage source (the assets to label) and target (the location to which labels and output data should be exported).

Connections can be set up and shared across projects. They use an extensible provider model, so new source/target providers can easily be added.

To create a new connection, click the New Connections (plug) icon, in the left hand navigation bar. 

![alt text for image](media/SampleUX-Connection-Settings.png)

**Connection settings -** 
1. **Display Name** - the connection display name
2. **Description** - Optional - your project description
3. **Provide** - Select Azure Blob Storage as the provider. 
4. **Account name** - Your Azure Storage account name
5. **Container name** - Your Azure storage container name where the source forms will be located. 
6. **SAS** - The Azure Blob storage shared access signature (SAS) URL. To retrieve the SAS URL, open the Microsoft Azure Storage Explorer, right-click your storage blob, and select Get shared access signature. Set the expiry time to the time you plan to label and use the service. Make sure the Read, Write, Delete and List permissions are checked, and click Create. Then copy the query string value in the URL section. It should have the form: ?<SAS value>. Use the query string for the SAS* configuration. 
In your Azure Portal go to the Blobl Storage, Shared Access Signature to get the SAS query: 
  
![alt text for image](media/SAS-storage.png)
  
### Creating a New Project
Labeling forms in the Sample UX revolve around projects - a collection of configurations and settings that persist.

Projects define source connections, and project metadata - including tags to be used when labeling source assets.

![alt text for image](media/SampleUX-New-Project.png)

**Project settings -** 
1. **Display Name** - the project display name
2. **Security Token** - Some project settings can include sensitive values, such as API keys or other shared secrets. Each project will generate a security token that can be used to encrypt/decrypt sensitive project settings. Security tokens can be found in Application Settings by clicking the gear icon in the lower corner of the left hand navigation bar.
3. **Source Connection** - The Azure Blob Storage connection you created in the previous step which you would like to use for this project. 
4. **Folder Path** - Optional - If your source forms are located in a folder on the blob container please specify the folder name here
5. **Form Recognizer Backend** - The Form Recognizer backend container IP and port - 
http://localhost:5005
6. **OCR Service URI** - The Form Recognizer Read Layout OCR container IP and port and API path - http://localhost:7005/formrecognizer/v2.0-preview/readLayout/asyncAnalyze
7. **Description** - Optional - Project description
8. **Tags** - Optional - The key label names for your project, these can also be created within the project. 

### Labeling Forms
When a project is created or opened, the main tag editor window opens. The tag editor consists of three main parts:

1. A resizeable preview pane that contains a scrollable list of forms, from the source connection
2. The main editor pane that allows tags to be applied
3. The tags editor pane that allows users to modify, lock, reorder, and delete tags
Selecting a form on the left will load that from in the main tag editor. Values can then be selected and a tag can be applied.

As desired, repeat this process for any additional forms.

To start labeling create the tags (key labels) you would like to extract using the tags editor pane. 

![alt text for image](media/Tags.PNG)

To label files:  
a.	Use Click and Drag gesture to select one or multiple words from the highlited OCR results.
b.	Click on the tag by mouse or press corresponding tag number on keyboard.

![alt text for image](media/label-screen.png)

Tag Ordering: 
Hotkeys of 1 through 0 are assigned to the first ten tags. These can be reordered by using the up/down arrow icons in in the tag editor pane.

Once you labeled 10 forms you can train a model. Click on the Train icon in the left navigator. 

### Train a custom model
Click on the Train icon in the left navigator top open the Training page. Click the train button to train a model. Once train completes you will see the following: 
* Model ID - The model ID created for this train. Each train run creates a new model ID. You will need this model ID to call the Anazlye API. 
* Model Accuracy - The model average accuracy, if needed you can improve the model accuracy by labeling additional forms and running train again to create  new model. We recommend starting by labeling 10 forms and add additional forms as needed based on the model accuracy. 
* Tags and Estimated accuracy per tag

![alt text for image](media/Train.png)

> Note: You can train via the Sample UX or via the API calling the /formrecognizer/v2.0-preview/asyncTrain API. To train and Analyze via the API please refer to the [Quickstart: Train a Form Recognizer Feedback Loop model and extract form data by using the REST API with Python](fl-python-quickstart.md)

### Analyze a Form
Click on the predict icon to test your model analyze a form and extract key value pairs. 

![alt text for image](media/Analyze.png)

> Note: You can analyze forms and extract key value pairs via the Sample UX or via the API calling the /formrecognizer/v2.0-preview/asyncAnalyze API. To analyze a form via the API please refer to the [Quickstart: Train a Form Recognizer Feedback Loop model and extract form data by using the REST API with Python](fl-python-quickstart.md)

### Next steps
In this quickstart, you used the Form Recognizer Feedback Loop Sample UX to label forms, train a model and run it in a sample scenario. Next, train a model on your own forms or use the model you just created to extract key value pairs from your forms via the REST API. Try it out at http://localhost:5005/swagger API to explore the Form Recognizer Feebback Loop API in more depth.




