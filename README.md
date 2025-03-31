---
# Batch Processing Pipeline with Azure Data Factory

## **Project Overview**

### **Description**
This project demonstrates the development of a robust batch processing pipeline designed to handle large-scale data ingestion and transformation. Using **Azure Data Factory (ADF)**, the pipeline ingests data from multiple sources, applies transformations with **Dataflows (powered by Spark)**, and loads the processed data into an **Azure Data Lake Storage Gen2** lakehouse with incremental loading capabilities.

---

## **Tech Stack**
- **Azure Data Factory**: Orchestrates the pipeline.
- **Azure Data Lake Storage Gen2**: Stores raw and processed data in a lakehouse structure.
- **Dataflows (Spark)**: Handles transformations like cleansing, deduplication, and aggregation.
- **Azure CLI**: Automates deployment and configuration.
- **GitHub**: Hosts configuration files, scripts, and documentation.

---

## **Prerequisites**
Before starting, ensure you have:
1. An active **Azure subscription**.
2. **Azure CLI** installed ([Installation Guide](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)).
3. Basic knowledge of JSON (for ADF pipeline configs).
4. Access to sample data sources (e.g., CSV files, SQL database).

---

## **Setup Instructions**

Follow these steps to deploy and run the pipeline. Each step includes Azure CLI commands (where applicable), detailed explanations, and screenshot placeholders for visual clarity.

### **Step 1: Create an Azure Resource Group**
**Why**: A resource group organizes all related Azure resources (e.g., ADF, Data Lake) in one logical container for easier management and cleanup.  
**Command**:
```bash
az group create --name BatchPipelineRG --location eastus
```
**Explanation**:  
- `BatchPipelineRG` is a unique name for the resource group.
- `eastus` is chosen for proximity (adjust based on your region for latency optimization).


![Image](https://github.com/user-attachments/assets/ea2a574f-7242-49bd-9a84-c669f9854265)

---

### **Step 2: Provision Azure Data Lake Storage Gen2**
**Why**: The lakehouse architecture relies on Data Lake Storage Gen2 for hierarchical namespaces and scalability, ideal for raw and processed data storage.  
**Command**:
```bash
az storage account create \
  --name batchpipelinestorage \
  --resource-group BatchPipelineRG \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2 \
  --hns true
```
**Explanation**:  
- `batchpipelinestorage` is a globally unique name.
- `Standard_LRS` (Locally Redundant Storage) balances cost and reliability.
- `--hns true` enables hierarchical namespace for lakehouse compatibility.

  
![Image](https://github.com/user-attachments/assets/30e968a7-9e9a-4725-bb71-f841c8e400b7)

![Image](https://github.com/user-attachments/assets/5b491de1-67b1-49a4-8b64-3f290ff96c23)

![Image](https://github.com/user-attachments/assets/31dd87f3-1d68-4fcd-9b07-0135330fb3f0)



---

### **Step 3: Create Containers in Data Lake Storage**
**Why**: Containers segment data (e.g., raw, staging, processed) for organization and access control.  
**Command**:
```bash
az storage fs create \
  --account-name batchpipelinestorage \
  --name raw \
  --auth-mode login
az storage fs create \
  --account-name batchpipelinestorage \
  --name staging \
  --auth-mode login
az storage fs create \
  --account-name batchpipelinestorage \
  --name processed \
  --auth-mode login
```
**Explanation**:  
- Three containers: `raw` (source data), `staging` (intermediate), and `processed` (final output).
- `--auth-mode login` uses your Azure CLI credentials for authentication.  


![Image](https://github.com/user-attachments/assets/9ee37766-9ba6-4aeb-aaa7-60a3f31a93b0)

![Image](https://github.com/user-attachments/assets/ca52e278-87b6-44c0-b662-b89588d866bd)

![Image](https://github.com/user-attachments/assets/cc3e07a0-2bb2-4e1e-9d19-7e7c6a9b49e3)

![Image](https://github.com/user-attachments/assets/03ae63e2-5a5c-4f55-84d0-14c0bb7d37dd)

---

### **Step 4: Deploy Azure Data Factory**
**Why**: ADF orchestrates the pipeline, connecting data sources, transformations, and sinks.  
**Command**:
```bash
az datafactory create \
  --name BatchP-ADF \
  --resource-group BatchPipelineRG \
  --location eastus
```
**Explanation**:  
- `BatchP-ADF` is a unique name for the factory.
- Placed in the same region as other resources to minimize latency.

   
![Image](https://github.com/user-attachments/assets/82220c28-f4fa-401c-8674-064ffa3d5d90)

![Image](https://github.com/user-attachments/assets/725bc65c-1435-4673-abaa-bca47f7af1f4)

![image](https://github.com/user-attachments/assets/ca92f112-8bd5-42ec-919f-96341832154e)

---

### **Step 5: Link Data Lake Storage to ADF**
**Why**: A linked service connects ADF to the Data Lake for data movement and storage.  
**Manual Step**:  
1. Open ADF in the Azure Portal.
2. Go to "Author" > "Manage" > "New Linked Service."
3. Select "Azure Data Lake Storage Gen2."
4. Enter:
   - Name: `DataLakeLinkedService`
   - Storage Account: `batchpipelinestorage`
   - Authentication: Use "System Assigned Managed Identity" for security.
5. Test the connection and save.  
**Explanation**:  
- Managed Identity avoids hardcoding credentials, enhancing security.
- This links ADF to the lakehouse for read/write operations.  
**Screenshot**: Show the "Linked Services" tab with `DataLakeLinkedService` configured.

![Image](https://github.com/user-attachments/assets/37e95852-fdf6-4977-aebc-216249f94e93)


I ran into an error while testing the connection.
The error says, I need to enable auto-resolve-integration-runtime 
on the ADF.
At the time I could not find the edit tab for the ADF 
and because I'd just made this resource,
So I started over and made a new data factory 
and this time enabled auto-resolve-integration-runtime
in the configuration settings under the networking tab. 


![Image](https://github.com/user-attachments/assets/bbd816ba-8956-4fb9-9295-2bbbc88b3fbd)



I had to recreate the "BatchP-ADF" pipeline 
to accept auto-resolve-integration-runtime. 



![Image](https://github.com/user-attachments/assets/f4f127cd-a75b-482c-9646-1856db7785e7)







![Image](https://github.com/user-attachments/assets/67e0c8e7-7c73-4d00-83ab-c98c23ce9fda)




![image](https://github.com/user-attachments/assets/2b9e6cf6-554b-4dc4-a663-4be3e1bc5e09)





![Image](https://github.com/user-attachments/assets/d3d1e64f-b13c-4683-8536-6cc51077b70f)







![Image](https://github.com/user-attachments/assets/bcecfb5a-1274-463d-963b-89a53f840473)



Connected with private endpoint.


![Image](https://github.com/user-attachments/assets/dc9ee967-707b-4c72-b915-c818343188e8)


Then I waited for approval state to clear. 


![Image](https://github.com/user-attachments/assets/688cf055-c0f1-44c2-8e74-82512637da18)




Connection approved along with interactive authroring being enabled. 


![image](https://github.com/user-attachments/assets/f55ffa86-1669-4190-8fc7-9f39b4552595)



Edited storage account (ADLS) 
Added a new role assignment, 
my user account assigned to storage blob contributer. 
Selected members, 
system assigned user identities and ADF resource as the selected member. 



![Image](https://github.com/user-attachments/assets/382735fc-9044-44c2-9bc3-4067fc74234b)



Then reviewed and assign. 


![Image](https://github.com/user-attachments/assets/478d686a-10b8-4e01-813b-8e162a6c6372)



Refreshed the private endpoint on ADF and 
tested the connection again this time was successful.


![Image](https://github.com/user-attachments/assets/4706f6a5-5e45-40a0-9c69-49215a0d6307)


Afterwards I click create, to create the linked service. 




![Image](https://github.com/user-attachments/assets/bb11270e-0ec7-4f77-951f-0eee12e40e75)



---

### **Step 6: Create a Pipeline in ADF**
**Why**: The pipeline defines the workflow: ingest → transform → load.  
**Manual Step**:  
1. In ADF, go to "Author" > "Pipelines" > "New Pipeline."
2. Name it `BatchProcessingPipeline`.
3. Add activities (detailed in Step 7).  
**Explanation**:  
- The pipeline is the core orchestration logic, chaining tasks in sequence.



  
![Image](https://github.com/user-attachments/assets/9fd8bde9-c2b1-46d8-9b6e-71278aa062b9)

---

### **Step 7: Configure Data Ingestion**
**Why**: Ingesting from multiple sources (e.g., CSV, SQL) simulates real-world heterogeneity.  
**Manual Step**:  
1. Add a "Copy Data" activity to the pipeline.
2. Source: Upload a sample CSV to `raw` container (e.g., via Azure Portal or CLI).
3. Sink: Set to `staging` container via `DataLakeLinkedService`.
4. Configure mappings if needed.  
**CLI (Optional)**:
```bash
az storage blob upload \
  --account-name batchpipelinestorage \
  --container-name raw \
  --name sample_data.csv \
  --file ./sample_data.csv \
  --auth-mode login
```
**Explanation**:  
- The "Copy Data" activity moves data without transformation, preserving the raw state.
- Sample CSV mimics a common enterprise data format.



![image](https://github.com/user-attachments/assets/01d71d09-8b81-44af-a70d-2f336810fabb)


![Image](https://github.com/user-attachments/assets/9818cbec-cecc-447d-bfc9-db448dbe67e6)



![Image](https://github.com/user-attachments/assets/a0b7ca4e-fae2-4bf5-8300-b81eb643c500)




![Image](https://github.com/user-attachments/assets/47b9129a-00d0-4d72-a568-40d9fbf1c003)



![Image](https://github.com/user-attachments/assets/1dbeb49c-2ad2-42d9-ae9d-82dbe7c4752e)




---

### **Step 8: Add Transformation with Dataflows**
**Why**: Dataflows (Spark-based) handle complex transformations like cleansing and deduplication scalably.  
**Manual Step**:  
1. In ADF, go to "Author" > "Dataflows" > "New Dataflow."
2. Name it `TransformData`.
3. Source: `staging` container.
4. Add transformations (e.g., remove duplicates, filter nulls).
5. Sink: `processed` container.  
6. Link the Dataflow to the pipeline after the "Copy Data" activity.  
**Explanation**:  
- Spark ensures scalability for large datasets.
- Transformations improve data quality, a key Azure Data Engineer skill.  



![Image](https://github.com/user-attachments/assets/1dbeb49c-2ad2-42d9-ae9d-82dbe7c4752e)



![Image](https://github.com/user-attachments/assets/8beb37f4-5e77-44e1-bad6-f6f164af4cd9)

---

### **Step 9: Implement Incremental Loading**
**Why**: Incremental loading processes only new/changed data, optimizing performance.  
**Manual Step**:  
1. Add a "Lookup" activity before "Copy Data" to check the last processed timestamp (stored in a metadata file).
2. Use a "Filter" activity to select only new rows based on the timestamp.
3. Update the metadata file post-run with the latest timestamp.  
**Explanation**:  
- Reduces redundant processing, critical for large-scale systems.
- Metadata-driven logic is an industry best practice.  
**Screenshot**: Show the pipeline with "Lookup" and "Filter" activities connected.

---

### **Step 10: Test and Monitor the Pipeline**
**Why**: Validation ensures reliability; monitoring tracks performance.  
**Manual Step**:  
1. In ADF, click "Debug" to test the pipeline.
2. Go to "Monitor" to view run status and logs.  
**Explanation**:  
- Debugging catches errors early.
- Logs provide metrics (e.g., runtime, data volume) for optimization.  
**Screenshot**: Capture the "Monitor" tab with a successful pipeline run.

---

## **Repository Structure**
```
├── configs/
│   ├── pipeline.json         # ADF pipeline configuration
│   ├── dataflow.json        # Dataflow transformation script
├── scripts/
│   ├── deploy.sh            # Azure CLI deployment script
├── sample_data/
│   ├── sample_data.csv      # Sample input data
├── README.md                # This file
```

