---
layout: post
title: automatic image extraction for Documents
subtitle: extract images of documents analyzed by DocumentIntelligence 
thumbnail-img: /assets/img/20240912/thumb.jpg
cover-img: /assets/img/20240724/banner.png
share-img: /assets/img/20240724/banner.png
tags: [Azure, DocumentIntelligence, RAG, AI, Azure OpenAI]
comments: true
---

In many documents, images, graphs, and tables contain critical information that often complements, or even surpasses, the relevance of the accompanying text. For effective Retrieval Augmented Generation (RAG), it is essential to index this visual data alongside the textual content. Neglecting to capture information from images, tables, or other figures results in incomplete data retrieval and diminishes the quality of answer generation when using a Large Language Model (LLM). Therefore, extracting and indexing this information is vital for achieving optimal performance.

Historically, extracting visual elements from documents has been a complex task. It required converting bounding regions into bounding box coordinates and frequently involved using external Python frameworks, adding significant complexity to the process.

However, with the introduction of Azure Document Intelligence’s new preview API version (python SDK: 1.0.0b4 / REST API: 2024-07-31-preview), this process has become significantly streamlined. The API now enables the extraction of cropped images as a native feature of the document analysis method. This is a major advancement, especially for customers who heavily rely on Azure Document Intelligence. By integrating this functionality into the SDK, the product becomes even more powerful, allowing for more efficient inclusion of visual elements, leading to improved accuracy and more comprehensive results in RAG workflows.

Below you will find a quick python sample which outlines the extraction process:

```python
from azure.ai.documentintelligence import DocumentIntelligenceClient
from azure.core.credentials import AzureKeyCredential
import dotenv
import os
from azure.ai.documentintelligence.models import AnalyzeDocumentRequest, ContentFormat, AnalyzeOutputOption,AnalyzeResult

dotenv.load_dotenv(override=True)

dic = DocumentIntelligenceClient(os.getenv("AzureDiEndpoint"), AzureKeyCredential(os.getenv("AzureDiKey")))

img = open("path to your image","rb")

poller = dic.begin_analyze_document(
    "prebuilt-layout",
    AnalyzeDocumentRequest(bytes_source=img.read()),
    output_content_format=ContentFormat.MARKDOWN,
    output=[AnalyzeOutputOption.FIGURES]
)
res: AnalyzeResult = poller.result()
resultid = poller.details.get('operation_id')

if res.figures:
    for figure in res.figures:
        if figure.id:
            response = dic.get_analyze_result_figure(
                model_id=res.model_id, result_id=resultid, figure_id=figure.id
            )
            with open(f"{figure.id}.png", "wb") as writer:
                writer.writelines(response)

```