# Azure End-To-End Adventure Data Engineering Project
In this end to end project I have used the following:
 - GitHub 
 - Azure Data Factory
 - Azure Databricks
 - Azure Synapse Analytics
 - Microsoft PowerBi

## Project Architecture
![Azure ADF, Databricks, synapse   PowerBi](https://github.com/user-attachments/assets/3e16a5de-108c-4c87-8dc1-8dd619d8ea93)

## Going into the project
### Ingestion
In the Azure account I created a resource group and in that **resource group**, created the various **resources**.
  - Data Lake (This can be confused with blob storage during creation. Inorder to create a data lake which enbles us to create hierachy - Folder then files. Make sure to check on the hierachical namespace.)
      . In this I created the bronze,silver and gold layer
    ![Uploading Screenshot (70).png…]()
    
  - Azure Data Factory
  - Azure Databricks


I used **API(github)** as a source of my data and the destination being the bronze layer in the ADLS Gen 2.
Inorder to succesfully load the data I used Azure Data Factory and created a linked service to the ADLS Gen 2 and the Github account using **Http connection**.
After that I used copy activity and loaded the data.

Because I had various files in the folder, I needed to make the pipeline **dynamic**. So I created a json file to save the relative urls of each files, file name and folder name in an array to make it easy to loop through.

This created, I added another container in the data lake named 'parameters' and uploaded the json file in this seperate container.

To create the pipeline I used **Lookup** activity connected to a **forEach activity** and in the forEach activity created a copy activity to collect the data. In the lookup activity, uncheck file row to load all the rows in the json folder. And to get the values add the '.value' in the expression builder after the parameter.
<img width="1366" height="768" alt="Screenshot (75)" src="https://github.com/user-attachments/assets/0f765990-40f5-49de-9854-be8689f9c123" />


### Transformation using Azure Databricks
To acces data in ADLS Gen 2 I created an application through the following process.
   1. Microsoft Entra Id
   2. App registration > New Registration (Save the details)
   3. Certificates and Secrets

Then assign the app a role by going to the storage account and to the IAM > Add Storage blob contributor(read and write powers). The selected member and searched the name of the app created.
The details of the app created is the saved in the notebook inside the Aatabricks as below:

<img width="1366" height="768" alt="Screenshot (76)" src="https://github.com/user-attachments/assets/01bc8b6e-96de-454a-aa64-08357503f3ac" />
<img width="1366" height="768" alt="Screenshot (78)" src="https://github.com/user-attachments/assets/32bb6533-50c5-4ba1-b5a5-07204b4c591a" />
As seen in the screenshot above I managed to read the data and do some transformations using pySpark. And then write the data to the silver layer in the ADLS Gen 2 in Azure.

### Serving using Synapse Analytics
Inorder for Synapse to access ADLS Gen 2 I created a managed identity/system managed identity through these steps:
   1. IAM in the storage account
   2. Add role the storage blob contributor
   3. select memebers and added synapse which I created in my resource group.

      - Also create an IAM Access for admin by creating blob contributor and selecting your email.

Here I did not use service principle unlike databricks because synapse is a product of microsoft.
In the Synapse Analytics I used serveless SQL pool and openrowset() fuction to read files without creating a database.
<img width="1366" height="768" alt="Screenshot (81)" src="https://github.com/user-attachments/assets/1bb25536-4d01-4406-8548-f34940c3fb5d" />
<img width="1366" height="768" alt="Screenshot (82)" src="https://github.com/user-attachments/assets/8e89afb6-e455-48bd-bef9-1d564fbd0d4f" />
I create a view first and then External tables using the format below:
  1. CREDENTIALS
  2. EXTERNAL DATA_SOURCE
  3. FILE FORMAT

And as seen below you can see that a sample data of customers was loaded into the gold layer in the Data Lake which can then be used by Analysts.

### Visualizing in PowerBi
Inorder to create visualization you need to load data into powerbi by using SQL end point.
To get the endpoint I copied the serverless SQL end point and together with my username plus password.
In PowerBi went to get data and searched for azure and then azure synapse analystics and then database and filled the required areas. As below:
<img width="1366" height="768" alt="Screenshot (86)" src="https://github.com/user-attachments/assets/60dea086-6406-49f2-a231-0020b2bbe04f" />



