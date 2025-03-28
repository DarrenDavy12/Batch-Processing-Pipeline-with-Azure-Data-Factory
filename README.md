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
**Screenshot**: Take a screenshot of the Azure Portal showing the new resource group under "Resource Groups."

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
**Screenshot**: Capture the storage account in the Azure Portal under "Storage Accounts."

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
**Screenshot**: Show the "Containers" section in the storage account with `raw`, `staging`, and `processed`.

---

### **Step 4: Deploy Azure Data Factory**
**Why**: ADF orchestrates the pipeline, connecting data sources, transformations, and sinks.  
**Command**:
```bash
az datafactory create \
  --name BatchPipelineADF \
  --resource-group BatchPipelineRG \
  --location eastus
```
**Explanation**:  
- `BatchPipelineADF` is a unique name for the factory.
- Placed in the same region as other resources to minimize latency.  
**Screenshot**: Display the new ADF instance in the Azure Portal under "Data Factories."

---

### **Step 5: Link Data Lake Storage to ADF**
**Why**: A linked service connects ADF to the Data Lake for data movement and storage.  
**Manual Step**:  
1. Open ADF in the Azure Portal.
2. Go to "Author" > "Connections" > "New Linked Service."
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

---

### **Step 6: Create a Pipeline in ADF**
**Why**: The pipeline defines the workflow: ingest → transform → load.  
**Manual Step**:  
1. In ADF, go to "Author" > "Pipelines" > "New Pipeline."
2. Name it `BatchProcessingPipeline`.
3. Add activities (detailed in Step 7).  
**Explanation**:  
- The pipeline is the core orchestration logic, chaining tasks in sequence.  
**Screenshot**: Capture the pipeline canvas with the name `BatchProcessingPipeline`.

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
**Screenshot**: Show the "Copy Data" activity settings with source and sink configured.

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
**Screenshot**: Show the Dataflow canvas with transformations applied.

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

