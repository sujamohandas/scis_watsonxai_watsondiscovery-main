# Infusing watsonx.ai with IBM Supply Chain Intelligence Suite

The IBM Supply Chain Intelligence Suite is a comprehensive software platform designed to optimize and enhance supply chain operations. It combines the power of AI and the speed of automation to help you quickly gauge the status of your supply chain and mitigate disruptions. The suite offers a range of tools and capabilities, including real-time visibility, predictive analytics, demand forecasting, and risk management. By integrating these features, IBM Supply Chain Intelligence Suite helps organizations improve efficiency, reduce costs, enhance collaboration with suppliers and partners, and respond proactively to disruptions and market changes.

The IBM Supply Chain Intelligence Suite includes an Supply Chain Business Assistant; a conversation AI platform. The Supply Chain Business Assistant leverages IBM Watson Assistant, which uses natural language understanding (NLU) to understand a user's request and responds accordingly.By using natural language processing, Watson Assistant can help users access data, generate reports, and receive insights and recommendations through a conversational interface. The Supply Chain Business Assistant is trained on a pre-defined set of requests on supply chain.

This tutorial details the steps to train the Supply Chain Business Assistant on Scope 3 terminologies,by implementing a [dialog skill](https://www.ibm.com/docs/en/scis?topic=skills-implementing-skill-interface)

## Pre-reqs to integrating Watson Assistant with SCIS tenant 
The Watson Assistant instance with the custom skills need to be registered with the SCIS tenant which needs to
recognize the custom skills.

An IBM support ticket needs to be logged for the [same](https://www.ibm.com/mysupport/s/?language=en_US) with the following details as shown below :
- SCIS tenant Id
- Watson Assistant ID
- Watson Assistant Service URL
- Watson Assistant API key
- Watson Assistant Draft Env Id and Live Env Id

<img src="images/image25.png">

## Configurations for Watson Discovery 
1.Load the Watson Discovery instance from Resource list-> AI/Machine learning as displayed below

<img src="images/image31.png">

2.Create a project by clicking on "New Project"

<img src="images/image24.png">

3.Create the new collection by clicking on "New collection" inside project

<img src="images/image20.png">

4.Upload the required documents as per the need to the Watson Discovery instance 
Click on Upload data tab, as shown in below screen

<img src="images/image21.png">

5.Do relevancy training to improve the watson assistant response .

<img src="images/image23.png">

## Configurations for Watson Assistant
Login into [IBM Cloud](https://cloud.ibm.com/login) using your IBM credentials.
Load the Watson Assistant instance from Resource list-> AI/Machine learning

<img src="images/image31.png">

## Creating intent
Click on "Create intent" button and provide the intent name and add "User Example".
User examples are the questions to be asked in Watson assistant.Add as many user examples, as possible as shown below.

<img src="images/image29.png">

<img src="images/image1.png">

## Addition of extension
Traverse to "Integrations" menu item as shown in image below:

<img src="images/image30.png">

Select the "Build custom extension" button from the Extensions page
<img src="images/image9.png">

### Configure the Watson Discovery extension 
Upload the [wd_openapi.json](./files/Extension/wd_openapi.json)
Provide the apikey and URL for Watson Discovery instance and Save the changes.

<img src="images/image10.png">

The Api key and URL details for Watson Discovery instance can be fetched from below screen in Resource list -->Watson Discovery instance

<img src="images/image27.png">

### Configure the watsonx.ai extension

Upload the [watsonx-openapi.json](./files/Extension/watsonx-openapi.json.json)
Provide the Apikey and URL of watson.ai instance within the Custom Extension and save the changes

<img src="images/image11.png">

The Api key and URL details for watsonx.ai instance can be fetched from below screen in Resource list -->Watson Assistant instance

<img src="images/image28.png">

## Defining variables 
Below variables must be defined, which would be used in the various steps in defining Actions.
We have used the ibm/granite-20b-multilingual model in this tutorial. Feel free to explore and use the [Granite models from IBM](https://www.ibm.com/granite)

<img src="images/image7.png">


## Creating Action
Click on "New action" to create an action by giving a name 
 
 <img src="images/image6.png">

Create 5 steps inside the action as listed below.Add consecutive steps by clicking on the New step button  :-

1.For the action to consume the input from the user and store it in variable "Question", define the step as shown below:

<img src="images/image5.png">

Provide the Watson Discovery extension details as per the below image:

<img src="images/image16.png">

count -->3

query -->"Question" varible defined.

passages.enabled-->true

passages.characters-->250

passages.find_answers-->true

collection_ids -->Should be fetched from as screenshot shown below and stored in variable and use the same

table_results.enabled-->false

project_id-->Should be fetched from as screenshot shown below and stored in variable and use the same

version-->2022-08-01

To fetch collection_id and Project_id please refer to screenshot below

<img src="images/image35.png">

<img src="images/image37.png">

2.Store the output from Watson discovery output in a variable "WD_Result_info".From the result obtained, the 1st result with more confidence is considered here.

<img src="images/image 12.png">

3.Set a temporary string variable, temp_input for concatenation purpose.Provide necessary instruction to the LLM model and empty the variable "Question"
Model prompt is the instruction given to LLM sample. In this use case, we are setting it to `("Answer the following question using information from the article confidently , which is surrounding with ###.  If there is no answer in the article, provide the same article as the response.Article ###").concat("Result from discovery").concat("###").concat("Question")`.

<img src="images/image13.png">

4.Provide the Watsonx.ai extension details as below:

<img src="images/image14.png">

input --> Input to Granite model is given here 

model_id -->Should be fetched from as screenshot shown below and stored in variable and use the same

project_id -->Should be fetched from as screenshot shown below and stored in variable and use the same

parameters.max_new_tokens -->500

parameters.temperature -->.5

parameters.top_k -->50

parameters.top_p -->1

parameters.decoding_method-->sample

parameters.min_new_tokens-->1

version-->2023-05-29


To fetch model_id and Project_id please refer to screenshot below

<img src="images/image36.png">

5.Set the output of LLM to a variable,prompt_result, which will be the final response from the assistant and displayed to the user

<img src="images/image15.png">

## Creating Dialog

1. Traverse to Dialog from the hamburger menu. Click "Add node" and provide the name of the node .Provide the relevant intent names within "If assistant recognizes". Then, call out to the action created above, from drop down.
Set parameters as Question variable with the text input by the user.Return variable is auto populated.

<img src="images/image2.png">

Within the "Assistant responds" section, provide output variable that is auto-created and set the next one to anything_else

<img src="images/image3.png">

## Configuring webhook
Webhooks are a feature of the watsonx Assistant, to interact with external systems
Please give below details to configure the webhook 
  -URL https://api.ibm.com/scassistant/run/named-entities/na/recognize
  
  -Authorization 
      -Bearer $integrations.chat.private.jwt
      -X-IBM-Client-Id scassistant-$integrations.chat.private.tenant_id

<img src="images/image4.png">

## Calling webhook within the dialog

Click on the "Customize" gear icon for "call a webhook" from within the dialog

<img src="images/image32.png">

This is a sample dialog node that takes in a dynamic value Category (stored in $sterling_entities.category_name.value) from the user and displays the suppliers for the specific category.
The webhook, configured above, would be invoked from the dialog node, with the following configurations:

<img src="images/image17.png">
<img src="images/image18.png">
<img src="images/image19.png">

Click on the gear icon for $sterling_entities.Click on three dots icon and "Open JSON editor" to view the category name "compactSuppliersForCategory" which has been provided in card builder as below.

<img src="images/image33.png">

In SCIS, the page "compactSuppliersForCategory" is created, as shown below, which is linked to category name in Watson assistant using webhook callout to [NER API](https://www.ibm.com/docs/en/scis?topic=skills-using-named-entity-recognition-api)

<img src="images/image38.png">

2. Configure "Anything else" as well, in dialog node, in order to handle the queries which Assistant is not able to understand as explained in screenshot below.

<img src="images/image34.png">

## Summary and next steps

In conclusion, the convergence of IBM SCIS,Generative AI and Watson discovery makes a signicant advancement in supply chain management and reduce time and effort for business users.
The power of generative AI could also be harnessed to easen the work of developers too and generate GraphQL code, to create workqueues in SCIS.

## References

IBM Watson Discovery:-
https://github.com/watson-developer-cloud/assistant-toolkit/tree/master/integrations/extensions/starter-kits/watson-discovery

IBM watsonx Assistant Extensions Starter Kit:-
https://github.com/watson-developer-cloud/assistant-toolkit/tree/master/integrations/extensions

Watsonx.ai Integrations
https://github.com/watson-developer-cloud/assistant-toolkit/blob/master/integrations/extensions/starter-kits/language-model-watsonx/README.md

Webhook setup :-
https://cloud.ibm.com/docs/assistant?topic=assistant-dialog-webhooks
