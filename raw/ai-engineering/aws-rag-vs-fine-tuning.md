# Comparing Retrieval Augmented Generation and fine-tuning

> Source: https://docs.aws.amazon.com/prescriptive-guidance/latest/retrieval-augmented-generation-options/rag-vs-fine-tuning.html
> Collected: 2026-10-17
> Published: Unknown
> Format: Official Markdown body; site metadata and navigation omitted. Relative links retain their source-page base.

<a name="rag-vs-fine-tuning"></a>

The following table describes the advantages and disadvantages of the fine-tuning and RAG-based approaches.


| 
| 
| Approach | Advantages | Disadvantages | 
| --- |--- |--- |
| Fine-tuning | + If a fine-tuned model is trained using the unsupervised approach, then it is able to create content that more closely matches your organization's style.+ A fine-tuned model that is trained on proprietary or regulatory data can help your organization follow in-house or industry-specific data and compliance standards.  | + Fine-tuning can take a few hours to days, depending on the size of the model. Therefore, it not be a good solution if your custom documents change frequently.+ Fine-tuning requires an understanding of techniques, such as low-rank adaptation (LoRA) and parameter-efficient fine-tuning (PEFT). Fine-tuning might require a data scientist.+ Fine-tuning might not be available for all models.+ Fine-tuned models do not provide a reference to the source in their responses.+ There can be an increased risk of hallucination when using a fine-tuned model to answer questions. | 
| RAG | + RAG allows you to build a question-answering system for your custom documents without fine-tuning.+ RAG can incorporate the latest documents in a few minutes.+ AWS offers fully managed RAG solutions. Therefore, no data scientist or specialized knowledge of machine learning is required.+ In its response, a RAG model provides a reference to the information source.+ Because RAG uses the context from the vector search as the basis of its generated answer, there is a reduced risk of hallucination. | + RAG does not work well when summarizing information from entire documents. | 

 

If you need to build a question-answering solution that references your custom documents, then we recommend that you start from a RAG-based approach. Use fine-tuning if you need the model to perform additional tasks, such as summarization.

You can combine the fine-tuning and RAG approaches in a single model. In the case, the RAG architecture does not change, but the LLM that generates the answer is also fine-tuned with the custom documents. This combines the best of both worlds, and it might be an optimum solution for your use case. For more information about how to combine supervised fine-tuning with RAG, see the [RAFT: Adapting Language Model to Domain Specific RAG](https://arxiv.org/pdf/2403.10131) research from the University of California, Berkeley.
