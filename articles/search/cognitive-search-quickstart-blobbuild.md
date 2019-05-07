# Quickstart: Create an AI indexing pipeline using cognitive skills in Azure Search

Azure Search integrates with [Cognitive Services](https://azure.microsoft.com/services/cognitive-services/), adding content extraction, natural language processing (NLP), and image processing skills to an Azure Search indexing pipeline, making unsearchable or unstructured content more searchable. 

Many Cognitive Services resources - such as OCR, language detection, entity recognition to name a few - can be attached to an indexing process. The AI algorithms of Cognitive Services are used to find patterns, features, and characteristics in source data, returning structures and textual content that can be used in full-text search solutions based on Azure Search.

In this quickstart, create your first enrichment pipeline in the Azure portal before writing a single line of code:

> * Begin with sample data in Azure Blob storage
> * Configure the **Import data** wizard for cognitive indexing and enrichment 
> * Run the wizard (an entity skill detects people, location, and organizations)
> * Use **Search explorer** to query the enriched data

This quickstart runs on the Free service.

> [NOTE]
> As you expand scope by increasing the frequency of processing, adding more documents, or adding more AI algorithms, you will need to attach a billable Cognitive Services resource. Charges accrue when calling APIs in Cognitive Services, and for image extraction as part of the document-cracking stage in Azure Search. There are no charges for text extraction from documents.


## Create a Storage Account on Azure

Cognitive Services provides the AI. This quickstart includes steps for adding these resources in-line, when specifying the pipeline. It's not necessary to set up accounts in advance.

Azure services are required to provide the inputs to the indexing pipeline. You can use any data source supported by Azure Search indexers except for Azure Table Storage, which is not supported for AI indexing. This quickstart uses Azure Blob storage as a container for source data files. 

### Set up Azure Blob service and load sample data
To create a general-purpose v2 storage account in the Azure portal, follow these steps:

1. In the [Azure portal](https://ms.portal.azure.com/#create/Microsoft.StorageAccount-ARM), create a **Storage Account**
1. Select the subscription in which to create the storage account.
1. Under the **Resource group** field, select the existing resource.

1. Next, enter a name for your storage account. The name you choose must be unique across Azure. The name also must be between 3 and 24 characters in length, and can include numbers and lowercase letters only.
1. Select a location for your storage account, or use the default location.
1. Leave these fields set to their default values:

   |Field  |Value  |
   |---------|---------|
   |Deployment model     |Resource Manager         |
   |Performance     |Standard         |
   |Account kind     |StorageV2 (general-purpose v2)         |
   |Replication     |Locally redundant storage (LRS)         |
   |Access tier     |Hot         |

1. Select **Review + Create** to review your storage account settings and create the account.
1. Select **Create**. Creation status is reported in the top menu bell icon.
1. Once your service is created, select the resource, and click on **Blobs**.
1. Click on the **+** icon to create a container.
1. Click on the container you created to open it.
1. In the container you created, click **Upload** to upload the sample files located here: C:\Users\Admin\Desktop\Azure Search sample data

   ![Source files in Azure blob storage](./media/cognitive-search-quickstart-blob/sample-data.png)



## Create an Azure Search Resource on Azure
On the [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.Search) create an Azure Search resource. 

## Create the enrichment pipeline

Return to the Azure Search service dashboard page and click **Import data** on the command bar to set up cognitive enrichment in four steps.

  ![Import data command](media/cognitive-search-quickstart-blob/import-data-cmd2.png)

### Step 1: Create a data source

In **Connect to your data**, choose **Azure Blob storage**, select the account and container you created. Give the data source a name, and use default values for the rest. 

  ![Azure blob configuration](./media/cognitive-search-quickstart-blob/blob-datasource.png)

Continue to the next page.

  ![Next page button for cognitive search](media/cognitive-search-quickstart-blob/next-button-add-cog-search.png)

### Step 2: Add cognitive skills

Next, add enrichment steps to the indexing pipeline. If you do not have a Cognitive Services resource, you can sign up for a free version that gives you 20 transactions daily. The sample data consists of 14 files, so your daily allocation will be mostly used up once you run this wizard.

1. Expand **Attach Cognitive Services** to view options for resourcing the Cognitive Services APIs. For the purposes of this tutorial, you can use the **Free** resource.

   ![Attach Cognitive Services](media/cognitive-search-quickstart-blob/cog-search-attach.png)

2. Expand **Add Enrichments** and select skills that perform natural language processing. For this quickstart, choose entity recognition for people, organizations, and locations.

   ![Attach Cognitive Services](media/cognitive-search-quickstart-blob/skillset.png)

   The portal offers built-in skills for OCR processing and text analysis. In the portal, a skillset operates over a single source field. That might seem like a small target, but for Azure blobs the `content` field contains most of the blob document (for example, a Word doc or PowerPoint deck). As such, this field is an ideal input because all of a blob's content is there.

3. Continue to the next page.

   ![Next page customize index](media/cognitive-search-quickstart-blob/next-button-customize-index.png)

> [!NOTE]
> Natural language processing skills operate over text content in the sample data set. Since we didn't select the OCR option, the JPEG and PNG files found in the sample data set won't be processed in this quickstart. 

### Step 3: Configure the index

The wizard can usually infer a default index. In this step, you can view the generated index schema and potentially revise any settings. Below is the default index created for the demo Blob data set.

For this quickstart, the wizard does a good job setting reasonable defaults: 

+ Default name is *azureblob-index* based on the data source type. 

+ Default fields are based on the original source data field (`content`), plus the output fields (`people`, `organizations`, and `locations`) created by the cognitive pipeline. Default data types are inferred from metadata and data sampling.

+ Default key is *metadata_storage_path* (this field contains unique values).

+ Default attributes are **Retrievable** and **Searchable** for these fields. **Searchable** indicates a field can be searched. **Retrievable** means it can be returned in results. The wizard assumes you want these fields to be retrievable and searchable because you created them via a skillset.

  ![Index fields](media/cognitive-search-quickstart-blob/index-fields.png)

Notice the strikethrough and question mark on the **Retrievable** attribute by the `content` field. For text-heavy blob documents, the `content` field contains the bulk of the file, potentially running into thousands of lines. If you need to pass file contents to client code, make sure that **Retrievable** stays selected. Otherwise, consider clearing this attribute on `content` if the extracted elements (`people`, `organizations`, and `locations`) are sufficient for your purposes.

Marking a field as **Retrievable** does not mean that the field *must* be present in the search results. You can precisely control search results composition by using the **$select** query parameter to specify which fields to include. For text-heavy fields like `content`, the **$select** parameter is your solution for providing manageable search results to the human users of your application, while ensuring client code has access to all the information it needs via the **Retrievable** attribute.
  
Continue to the next page.

  ![Next page create indexer](media/cognitive-search-quickstart-blob/next-button-create-indexer.png)

### Step 4: Configure the indexer

The indexer is a high-level resource that drives the indexing process. It specifies the data source name, a target index, and frequency of execution. The end result of the **Import data** wizard is always an indexer that you can run repeatedly.

In the **Indexer** page, you can accept the default name and use the **Run once** schedule option to run it immediately. 

  ![Indexer definition](media/cognitive-search-quickstart-blob/indexer-def.png)

Click **Submit** to create and simultaneously run the indexer.

## Monitor indexing

Enrichment steps take longer to complete than typical text-based indexing. The wizard should open the Indexer list in the overview page so that you can track progress. For self-navigation, go to the Overview page and click **Indexers**.

The warning occurs because JPG and PNG files are image files, and we omitted the OCR skill from this pipeline. You'll also find truncation notifications. Azure Search limits extraction to 32,000 characters on the Free tier.

  ![Azure search notification](./media/cognitive-search-quickstart-blob/indexer-notification.png)

Indexing and enrichment can take time, which is why smaller data sets are recommended for early exploration. 

## Query in Search explorer

After an index is created, you can submit queries to return documents from the index. In the portal, use **Search explorer** to run queries and view results. 

1. On the search service dashboard page, click **Search explorer** on the command bar.

1. Select **Change Index** at the top to select the index you created.

1. Enter a search string to query the index, such as `search=Microsoft&searchFields=organizations`.

Results are returned in JSON, which can be verbose and hard to read, especially in large documents originating from Azure blobs. If you can't scan results easily, use CTRL-F to search within documents. For this query, you could search within the JSON for specific terms. 

CTRL-F can also help you determine how many documents are in a given result set. For Azure blobs, the portal chooses "metadata_storage_path" as the key because each value is unique to the document. Using CTRL-F, search for "metadata_storage_path" to get a count of documents. 

  ![Search explorer example](./media/cognitive-search-quickstart-blob/search-explorer.png)

## Takeaways

You've now completed your first cognitive-enriched indexing exercise. The purpose of this quickstart was to introduce important concepts and walk you through the wizard so that you can quickly prototype a cognitive search solution using your own data.

Some key concepts that we hope you picked up include the dependency on Azure data sources. Cognitive search enrichment is bound to indexers, and indexers are Azure and source-specific. Although this quickstart uses Azure Blob storage, other Azure data sources are possible. For more information, see [Indexers in Azure Search](https://docs.microsoft.com/en-us/azure/search/search-indexer-overview).

Another important concept is that skills operate over input fields. In the portal, you have to choose a single source field for all the skills. In code, inputs can be other fields, or the output of an upstream skill.

 Inputs to a skill are mapped to an output field in an index. Internally, the portal sets up [annotations](https://docs.microsoft.com/en-us/azure/search/cognitive-search-concept-annotations-syntax) and defines a [skillset](https://docs.microsoft.com/en-us/azure/search/cognitive-search-defining-skillset), establishing the order of operations and general flow. These steps are hidden in the portal, but when you start writing code, these concepts become important.

Finally, you learned that viewing results is achieved by querying the index. In the end, what Azure Search provides is a searchable index, which you can query using either the [simple](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search) or [fully extended query syntax](https://docs.microsoft.com/rest/api/searchservice/lucene-query-syntax-in-azure-search). An index containing enriched fields is like any other. If you want to incorporate standard or [custom analyzers](https://docs.microsoft.com/en-us/azure/search/search-analyzers), [scoring profiles](https://docs.microsoft.com/en-us/azure/search/index-add-scoring-profiles), [synonyms](search-synonyms.md), [faceted filters](https://docs.microsoft.com/en-us/azure/search/search-filters-facets), geo-search, or any other Azure Search feature, you can certainly do so.
