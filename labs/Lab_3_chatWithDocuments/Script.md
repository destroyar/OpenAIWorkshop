# Deployment Script

## Prerequisites

* Owner or Contributor permission is required in the Azure subscription.
* Microsoft.Search Resource provider needs to be registered in the Azure Subscription. 
* [PostMan Client Installed](https://www.postman.com/downloads/) for testing Azure Functions. Azure portal can also be used to test Azure Function App.  
* Azure Cloud Shell is recommended as it comes with preinstalled dependencies. 
* Azure Open AI already provisioned and text-davinci-003 model is deployed. The model deployment name is required in the Azure Deployment step below. 

* Install [.Net Core 3.1 or later](https://dotnet.microsoft.com/en-us/download/dotnet/3.1)

* Install [Node.js](https://nodejs.org/en/download)

* [Azure Bot Framework Composer](https://learn.microsoft.com/en-us/composer/install-composer?tabs=windows#install-and-run-composer) is installed in local computer.

* [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases/tag/v4.14.1) installed in local computer. 



Before deploying the Azure resources, you will need Azure OpenAI API endpoint, API key, and the model deployment name.

Follow the steps below to get the Azure OpenAI endpoint and API key. Save the endpoint and API key in a notepad for later use.

1. Navigate to [Azure Open AI Studio](https://oai.azure.com/portal)

2. Click on the the Gear icon on Top right corner.

    ![Alt text](Images/lab3_image18_gearicon.png)

3. Navigate to Resource Tab and copy the endpoint and key in a notepad.


    ![Alt text](Images/lab3_image19_endpointandkey.png)



To get the Azure OpenAI Model deployment name, click on the deployment under Management, and copy the model deploment name.

![Alt text](Images/lab3_image17_deploymentname.png)

<br />

# 1. Azure services deployment

Deploy Azure Resources:
    - Azure Function App to orchestrate calls to Azure OpenAI and Cognitive Search APIs
    - Azure Cognitive Search Service
    - Azure Form Recognizer

Here are the SKUs that are needed for the Azure Resources:

- Azure Function App - Consumption Plan
- Azure Cognitive Search - Standard (To support semantic search)
- Azure Forms Recognizer (AFR) - Standard (To support analyzing 500 page document)
- Azure Storage - general purpose V1 (Needed for Azure Function App and uploading sample documents)

(control+click) to launch in new tab.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft-USEduAzure%2FOpenAIWorkshop%2Fmain%2Flabs%2FLab_3_chatWithDocuments%2Fdeploy%2Fazure-deploy.json) 

<br />

# 2. Setup Azure Cognitive Search and prepare data

As part of the data preparation step, the documents are chunked into smaller sections (20 lines) and stored as individual documents in the search index. The chunking logic is achieved with a python script below. 

* Go to the Resource Group created from the previous step, and open the Cognitive Search resource. Navigate to Semantic Search blade and then select the Free plan.

    ![Alt text](Images/lab3_image2_semanticsearchplan.png)
    

*   Create Search Index, Semantic Configuration and Index a few documents using automated script. The script can be run multiple times without any side effects.
    
    Open Cloud Shell by clicking cloud shell icon on the upper right corner of the Azure portal and select PowerShell. Create a Fileshare if it prompts, to save all the files of this lab.

    ![Alt text](Images/lab3_image3_cloudshell.png)
    
    Run the below commands from cloud shell to configure python environment. 

        
        git clone https://github.com/Microsoft-USEduAzure/OpenAIWorkshop.git
        
        cd OpenAIWorkshop/labs/Lab_3_chatWithDocuments 
        
        pip install -r ./orchestrator/requirements.txt


*   Update Azure Search, Azure Open AI endpoints, Azure Form Recognizer Endpoint and API Keys in the secrets.env. 
    
    Create a secrets.env file in the ingest folder that will be referenced by the search indexer (search-indexer.py):
    **The endpoints below needs to have the trailing '/' at end for the search-indexer to run correctly.**

        cd ingest
        
        # create secrets.env using the built-in code editor 
        # When using code, you can type control+s to save the file and control+q to quit the editor
        
        code secrets.env


    Add the below entries with correct values to secrets.env. Please refer to [this doc](ShowKeysandSecrets.md) to retrieve API Keys and Urls.

        AZSEARCH_EP="https://<YOUR Search Service Name>.search.windows.net/"
        AZSEARCH_KEY="<YOUR Search Service API Key>"
        AFR_ENDPOINT="<YOUR Azure Form Recognizer Service API EndPoint>"
        AFR_API_KEY="<YOUR Azure Form Recognizer API Key>"
        INDEX_NAME="azure-ml-docs"
        FILE_URL="https://github.com/Microsoft-USEduAzure/OpenAIWorkshop/raw/main/labs/Lab_3_chatWithDocuments/Data/azure-machine-learning-2-500.pdf"
        LOCAL_FOLDER_PATH=""

*   The document processing, chunking, indexing can all be scripted using any preferred language. 
    This repo uses Python. Run the below script to create search index, add semantic configuration and populate few sample documents from Azure doc. 
    The search indexer chunks a sample pdf document(500 pages) and chunks each page into 20 lines. Each chunk is created as a new search doc in the index. The pdf document processing is achieved using the Azure Form Recognizer service. 
    
        python search-indexer.py

<br />        

# 3. Test Azure Function App service deployment
Choose your preferred test method below to confirm your Function App deployment before continuing to Step 4.


## Postman

* Click on 'New' as shown in the below screenshot and then select 'HTTP'.

    ![Alt text](Images/lab3_image4_postman.png)

* Select method to "POST".

    ![Alt text](Images/lab3_image25_postmethod.png)

* Enter the URL of the Function app that you have created. Please refer to [this doc](ShowKeysandSecrets.md) to retrieve Function App Url 

    ![Alt text](Images/lab3_image26_posturl.png)


* Add below text just after the function app URL. The num_search_result query parameter can be altered to limit the search results. **num_search_result** is a *mandatory* query parameter.

        &num_search_result=5

    
    ![Alt text](Images/lab3_image27_numsearch.png)


* Now, you can test the function app by providing the below prompt in the 'Body' tab of postman. Make sure the 'raw' option is selected. Press 'Send'. The result will be displayed in the response window of Postman.

        {"prompt" : "Is GPU supported in AML"}

    ![Alt text](Images/lab3_image28_prompt.png)


## Azure Portal

* Navigate to the function app in the Azure Portal

* Select the `Functions` tab

* Select the `orchestrator` function

* Select the `Code + Test` tab

* Click the `Test/Run` button at the top of the page

* Add a query parameter called `num_search_result` with a value of `5`

* Add the following body content: `{"prompt" : "Is GPU supported in AML"}` to the body section, then click `Run`

    ![Input](Images/function-test-portal.PNG)

* Confirm the response was successful in the `Output` tab 

    ![Output](Images/function-test-success-portal.PNG)

## Visual Studio/VS Code
> :information_source: This method requires the REST Client extension when using VS Code

* Update [test.http](./orchestrator/test.http) with your function URL and key

* Click `Send Request`

    ![HTTP File Test](Images/http-file-test.PNG)

<br />

# 4. Build Chatbot 

Create a bot in Azure Bot Composer:

1. Open Bot Framework Composer in your local machine.


2. Select Create New (+) on the homepage.

    ![Alt text](Images/lab3_image5_bothomepage.png)



3. Under C#, select Empty Bot and click next.

    ![Alt text](Images/lab3_image6_selectbottemplate.png)



4. Provide a name to your bot (e.g.- CustomdataOpenAI_Bot). Select Runtime Type as 'Azure Web App'. Select a location in your local machine to save the bot files. Click on Create and wait untill the bot is created..

    ![Alt text](Images/lab3_image7_createabotproject.png)



5. Click on "Unknown intent". Click on the three virtical dots (node menu) on the right corner of 'send a response' box, and then click on delete. The send a response intent will be deleted.

    ![Alt text](Images/lab3_image8_deleteintent.png)



6. Click on '+' sign under unknown intent to add an intent. 

    ![Alt text](Images/lab3_image9_addintent.png)


7. Move the cursor to 'Access external resources' and select 'send an HTTP request'. The HTTP request is sent to make call to Azure function that has been created initially.

    ![Alt text](Images/lab3_image10_httprequest.png)



8. Select 'HTTP Method' as 'POST'. Copy the function app url (as copied when tested through postman). Please refer to [this doc](ShowKeysandSecrets.md) to retrieve the Function App Url. Put this url in under url box of the bot followed by **num_search_result** query parameter (see below image).

    ![Alt text](Images/lab3_image11_url.png)



9. Select Body box as 'object' from right corner drop down menu and copy and paste the below prompt in the body.
        
        {
            "prompt": "${turn.activity.text}"
        }

    ![Alt text](Images/lab3_image12_bodyprompt.png)



10. Select 'Response type' as 'json'.



11. Click on '+' sign under 'send an HTTP request' intent and select 'send a response'.

    ![Alt text](Images/lab3_image13_sendresponse.png)


12. In the 'responses' box, type the below expression.  
        
        ${turn.results.content.result}
    


Azure bot is now complete. In the next step, the bot is published on the Azure Cloud.

<br />

# 5. Publish the Chatbot


1. On the left most menu pane, select publish. 

    ![Alt text](Images/lab3_image14_publishbot.png)


2. Now, a publish target is selected. For selecting the target, click on 'select a publish target' and then select manage profiles.

    ![Alt text](Images/lab3_image15_manageprofiles.png)

3. Click on 'Add new' to create a new publishing profile. Provide a name to the publihsing profile and select 'Publishing target' as 'Publihs bot to Azure'. Click next.

4. Select 'Create new resources' and click next. A sign in to your Azure subscription is required- provide your Azure portal credentials to sign in. Once signed in, select the subscription and resource group. Resource group that has been created at the beginning of this lab is preffered, but a new resource group can also be created. 
Select operating system as 'windows'. 
Provide a name to the host resource.
Select region as 'East US'.
Select LUIS region as 'West US' and click next.

5. Uncheck the optional resources and click next and then click on Create in the next window. Wait for the publishing profile to be provisioned.

6. Clik on Publish tab and select the bot to be published. Select the profile provisioned in the above step. Click on 'Publish selected bots'. Click on 'Okay' in the next window. Wait for the bot to be published on the Azure Cloud.

    ![Alt text](Images/lab3_image16_publishbot.png)


<br />

# 6. Test 

1. Click on the Home button on the top left corner in the Bot Framework Composer. Select the bot that has been developed in the above steps.

2. Click on 'Start bot' button on the right left cornder of the Bot Framework Composer to start the developed bot. 

3. You can now test the bot either in web chat or in emulator. Click on either 'Open Web Chat' or 'Test in Emulator'. 

You can now ask questions related to 'Azure Machine Learning' to get the response from Azure OpenAI. 

E.g.: 
    
    What is Azure Machine Learning?
    Is GPU supported in AML?
    How to track and monitor training runs in AML?

You can explore integrating bot to other platforms, such as Microsoft Teams, a web application, etc. to augment the response returned from Azure OpenAI.
