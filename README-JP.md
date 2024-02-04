# Docs and Notes

## Todo

[ ] Check how the code works in detail
[x] Read through doc: <https://techcommunity.microsoft.com/t5/ai-azure-ai-services-blog/azure-ai-search-outperforming-vector-search-with-hybrid/ba-p/3929167>
[x] Go through Search Approach: Understand what Hybrid Retrieval means
[ ] Find out how expensive it was to build the App and how the cost was produced. (e.g. for Upload / Chunking for documents with 350 mb in total it was rougly 15 euros)
[ ] Watch the Custom RAG Chatvideo till the end. A lot of customisation opportunities: <https://www.youtube.com/watch?v=vt7oZg4bPAQ>
[x] Go through "builidng a Rag Chat App to slide 26 - a code walkthrough! Very important!
[ ] Add the Github ressources of the video to this doc: <https://www.youtube.com/watch?v=TI85JJVPnrM&t=1212> and  <https://github.com/sqlshep/OpenAI> - data pricacy etc. is mentionend there
[ ] Add these repo for a lot of great patterns: <https://github.com/microsoft/azure-openai-design-patterns>

## Explain how the repo works

![Azure Search OpenAI Demo Components](https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/appcomponents.png "Azure Search OpenAI Demo Components")

### Deployment (follow below)

- Read cost estimations section below first
- Follow the deployment guide: <https://github.com/Azure-Samples/azure-search-openai-demo/tree/main?tab=readme-ov-file#azure-deployment>
- The infrastucture, backend and frontend will be deployed to Azure with the biceps template.
- Configurations can for the infrastructure can be done in via environment variables in the `azd.env` file.
- The frontend and backend can be configured in the `app` folder. (see below)
- The user can chat with the bot in the frontend. The frontend is a React app.
- The backend is a Python app using the Quart framework.


### Adding your data <https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/data_ingestion.md>

- data (folder) - containt the data sources (pdfs, etc.) for the chatbot
- scripts (folder) - contains python scripts to upload the data to Azure.
  - In default, these scripts will take the documents, split the into single pages, pass them into Azure Form Recognizer.
  - You can also parse the documents locally and then upload them to Azure Search. This is cheaper but also more work.
- On Azure, the docs are stored in a blob storage, the chunking is done by Azure Form Recognizer and the indexed into Azure Search.

### Chatting with the bot

- The frontend sends the user input to the backend. The backend then sends the input to the OpenAI API and returns the answer to the frontend.
- The backend also uses Azure Search to retrieve documents and uses the OpenAI API to generate the answer
- The user can use two tabs: "Chat" and "Ask". The "Chat" tab is for multi-turn conversations and the "Ask" tab is for single-turn conversations.
- The answer not only contains the answer but also the sources of the answer. Also the prompt and how the answer was generated is included - basically the thought process.

## A) RAG Basics

## Cost Overview and Deployment Considerations

### Azure AI Search Pricing Insights

- The main cost driver was Azure AI Search, not OpenAI.
- Standard S1 Plan: $245/month, necessary for >2GB data. [Azure Search Pricing Details](https://azure.microsoft.com/en-gb/pricing/details/search/).
- For up to 2GB of data: $75/month.

### Cost Reduction Strategies

- Explore free or lower-cost deployment options:
  - [Low-cost Deployment Guide](https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/deploy_lowcost.md)
  - [Instructional Video for Low-Cost Deployment](https://www.youtube.com/watch?v=nlIyos0RXHw)

### Why RAG and not just GPT (an LLM)?

LLMs are good at Language but not at Reasoning. RAG is a combination of both. It is a hybrid approach.

- **Knowledge Cutoff**: There is a always a cut-off time for the training data of LLMs. So from this day on, the knowledge available is already outdated.
- **Only public knowledge**: LLMs are trained on public knowledge. All sources which are interal to a company or behind a paywall are not included in the training data.

### How can you incorporate your own knowledge?

- **Prompt Engineering**: You can give the LLM a bit of context but it only works if it has the knowledge inside it. Often it also halicinates knowledge which looks correct but if you are a domain expert you can see that it is wrong. So prompt engineering Can be helpful but normally not enough. => Example with Market Salad GPT and "Indian food"
- **Fine tuning**: You can fine tune the LLM on your own data. But this is very expensive and you need a lot of data. This is a valid option when your use case is very specialisd, you have a lot of data and you need very high accuracy, then this is probably the way to go. For most of company use cases it's not a good option economically.
- **Retrieval Augmented Generation (RAG)**: You can use a retrieval system to find the most relevant documents and then use the LLM to generate the answer. This is the approach we are using in this project.

### How does RAG work?

- The user asks a question, then you take it and search for fitting documents in a knowledge base.
- Afterwards you take the orginal user question together with the chunks from the knowledge base and feed it into the LLM to generate the answer.
- Typical for a RAG based system is that the user will get the sources of the answer as well, so he can evalute the answer himself.

TODO: add example prompts / queries of the chat here as code

### Explain typical RAG components

- **Retriver**: A knowledge base which is used to find the most relevant documents for a given question. This can be a search engine or a database which support vector search. (Azure Ai Search, CosmosDB, Postgres (<https://github.com/pgvector/pgvector>), Weaviate, Qdrant, Pineconce...)
- **LLM** A Model which can answers the questions based on the provided sources and can include citations (GPT3.5 / 4 Models, etc.)
- **Integration Layer ("Glue" in MS Slides)**: Optional Middleware which helps to connect the Retriver and the LLM. It can also be used to cache the results of the Retriver to speed up the process. It can be also done in pure Python but there are libaries which can help you with that. (Langchain, LLammaindex, Semantic Kernel, etc.)
- **Additional Features**: You can add additional features to your chatbot like chat history, Feedback buttons, Text to Speech, User Login, File Upload, etc.

### What kind of skillset is needed to build a RAG based chatbot?

- **No Code**: For easy applications using Copilot Studio of Azure or OpenApi GPT Builder. This might be enough for simple use cases.
- **Low Code**: UIs which help you to build more complex cases but within a UI (e.g. Azure Studio - On Your Data). There you can add hardware compontents (Retrievers as Azuer Ai Search, differen LLMs, Features as User Authentication, Chat History persistace.)
- **Code**: For Code base there are a lot of Azure Examples or for other suppliers as well. An example is the Azure RAG Chatbot which is used in this project <https://github.com/Azure-Samples/azure-search-openai-demo>.

## B) How to customize the RAG Chatbot <https://www.youtube.com/watch?v=vt7oZg4bPAQ>

### How to run the chatbot locally

#### Hotloading the Backend

- go to your local folder in vs code, open a terminal and run

```bash
  cd app
./start.sh
```

This will load the azd env file and start the backend and frontend. It automatically reloads the backend if you change something in the code.

- The backend is done on Quart (a Python framework based on Flask which supports async). It is a bit different from Flask but the same concepts apply. The frontend is in the folder `app/backend`. If you build a openai chat app like this, your backend should support async. The reasoning you can find in detail here: <https://blog.pamelafox.org/2023/09/best-practices-for-openai-chat-apps.html>

#### Hotloading the Frontend

- go to your local folder in vs code, open another terminal and run

```bash
  cd app/frontend
  npm run dev
```

then

  ```bash
    npm run dev
  ```

- this will give you a seperate localhost:5173 where you can see the frontend. It automatically reloads the frontend if you change something in the code. The backend request are going to the local server you've spun up in the other terminal.
- Be aware that in company network you may recieve proxy errors.

###  Code Walkthrough

#### Frontend

- **Technology**: TypeScript
- **Libraries/Frameworks**: React, FluentUI
- **Functionality**:
  - `chat.tsx`: Contains the UI components for the chat interface.
    - `makeApiRequest()`: Function to send requests to the backend API.
  - `api.ts`: Manages the API calls.
    - `chatApi()`: Interface to communicate with the chat service in the backend.

#### Backend

- **Technology**: Python
- **Libraries/Frameworks**: Quart, Uvicorn
- **Functionality**:
  - `app.py`: Main application file.
    - `chat()`: Function that handles chat requests.
  - `chatreadretrieveread.py`: Module for retrieving chat history and processing requests.
    - `run()`: Main entry point for running the chat service.
    - `get_search_query()`: Function to retrieve the search query from a request.
    - `compute_text_embedding()`: Function to compute embeddings for text analysis.
    - `search()`: Function to execute the search based on the query and embeddings.
    - `get_messages_from_history()`: Function to retrieve past messages for context.
    - `chat.completions.create()`: Function to generate chat responses.

### Customizing the frontend

To customize specific elements of the application, you should modify the following files:

| Change this file:                         | To customize:                 |
|-------------------------------------------|-------------------------------|
| `app/frontend/index.html`                 | title, metadata, script tag   |
| `app/frontend/public/favicon.ico`         | browser tab icon             |
| `app/frontend/src/pages/layout/Layout.tsx`| Navigation bar, colors        |
| `app/frontend/src/pages/chat/Chat.tsx`    | "Chat" tab and default settings |
| `app/frontend/src/pages/ask/Ask.tsx`      | "Ask" tab and default settings  |

### Customizing the backend

To customize specific elements of the backend, you should modify the following files:

| Change this file:                                     | To customize:                       |
|-------------------------------------------------------|-------------------------------------|
| `app/backend/app.py`                                  | additional routes, app configuration|
| `app/backend/approaches/chatreadretrieveread.py`      | "Chat" tab, RAG prompt and flow     |
| `app/backend/approaches/chatreadretrievereadvision.py`| "Chat" tab, RAG flow when using vision - this is still experimental in this current state|
| `app/backend/approaches/retrieveread.py`              | "Ask" tab, RAG prompt and flow      |
| `app/backend/approaches/retrievereadvision.py`        | "Ask" tab, RAG flow when using vision |

### Frontend - What is the Chat and Ask Tab?

- The Chat Tab is the tab where you can chat with the bot. You get the answer can ask follow up questions based on this. It's got context. It's a multi-turn conversation.
- The Ask Tab is the tab where you can ask a question and get an answer. It's a single turn conversation.



### [ ] Check if there is still things missing in this section

## Azure AI Search Best Practices <https://www.youtube.com/watch?v=ODuDeDrs3F0>

### Retrieval Matters

- As by experience of the MS team, the retrieval is the most important part of the RAG Chatbot. If the answers are not good the cause behind this is often not the LLM but the retrieval.
- So it is important to work on a rebust retrieval system for RAG chat apps.
- For best results in Azure Ai Search, you will probably want to use a hybrid approach.
  - **Vector search** + **Keyword search** in parellel
  - Using a **RFF** (Reciprocal Rank Fusion) to combine the results of both methods
  - Then use the **Semantic Ranking** to rank the top results from the hybrid search

See the following slides for more details:
<https://speakerdeck.com/pamelafox/azure-ai-search-best-practices-for-rag-chat-apps?slide=6>

### Vector Search

What is Vector Search based on? What are their strengths and weaknesses?

#### Vector embeddings

- Take a text and convert it into a list of floating point numbers (vector), so the the text is represented as a vector.
- There are very different models for this. The most common ones are:
  - **Word2Vec**: A classic model which takes in words
  - **OpenAi ada-002**: A model which takes in sentences
  - **Azure Computer Vision**: A model which takes in images or text
  
#### Demo how to compute a vector and use it for search with OpenAi ada-002 vector embeddings

Source: <https://github.com/pamelafox/vector-search-demos/blob/main/vector_embeddings.ipynb>

- create a vector representation of a text (list of floating point numbers)
- with this vector you can search for similarities to other embeddings. For this you calculate the distance, usually the cosine distance. The code shows a couple of examples. It's important to mention that it's the relevant distance between the cosines not the absolutes.
- in the demo there is a list of movies already as embeddings. If you query e.g. "Barbie" it showed "Babies in Toyland" and "Shopgirl" as the highest scores.
- => it's not only the wording but also the context, the meaning, etc. which is included in the vector. The specifics depends on the model you are using.

## Learnings

### 1. What is an optimal chunk size and a optimal overlap?

- There is a blog post of MS which researched on this: [Azure AI Search: Outperforming vector search with hybrid retrieval and ranking capabilities](https://techcommunity.microsoft.com/t5/ai-azure-ai-services-blog/azure-ai-search-outperforming-vector-search-with-hybrid/ba-p/3929167)
- They compared methods with different chunk sizes and different overlap sizes for Azure Ai Search.
- Hybrid Retrieval (Keyword + Vectore search paired with Semantic Ranking) using chunks with 512 tokens and 25% overlap performed best.

#### Technology behin Azure Ai Search

There are two main layers in Azure Ai Search:

- **Layer 1**: Retrieval - search for the most relevant documents. There are three methods supporte in Ai Search:
  - **Keyword**: Traditional keyword search. It is the fastest but least accurate.
  - **Vector**: Uses embeddings and cosine similarity to find the most similar documents. It is more accurate but slower.
  - **Hybrid**: Combination of Keyword and Vector. For Azure Ai Search they are using Reciprocal Rank Fusion (<https://learn.microsoft.com/en-us/azure/search/vector-search-ranking#reciprocal-rank-fusion-rrf-for-hybrid-queries>).
- **Layer 2**: Ranking - Prioritize the most relevant results. There is one method supported in Ai Search:
  - **Semantic Ranking**: Uses a mulit-lingual, deep learning model adapted from Bing Search. It can rank the top 50 results from L1.

#### Experiments for Search Methods

- They tested the different methods with different query types and different retrieval configurations.
- In both cases Hybrid retrieva with semantic ranking outperformed the other methods (namely Keyword, Vector and pure Hybrid).

#### Experiment for Chunking Strategies

- The paper compared different chunking and overalapping of chunks for the Hybrid Retrieval.
- The best results were Chunks of 512 tokens with 25% overlap.

#### What to do with these results in CAI?

- This might be useful for data scientists in CAI as Chunink Strategy was one of the major questions by Francesc.
- Keep in mind that the benchmarks are common benchmarks and might lead to different results in your specific use case.
- The results are only valid for Azure Ai Search and might be different for other search engines. Especially the "Semantic Rankig" is a proprietary method of Microsoft. It has to be checked what this methods does exactly and if it is available in other search engines as well.

### Further interesting points

#### TODO: What kind of Skillsets are needed for building a RAG Chatbot?

- Frontend / Backend: Classical Web Developer
- Data Preparation: Data Engineer

#### TODO: What are your most important learnings and why?

- Cost estimations are really hard for this use case. It depends a lot on the size of the documents, your chunking strategy, the number of documents, the number of users, the number of requests, etc.
- ==> What is the best way to estimate the costs for a RAG Chatbot?

#### TODO: Top Challenges

###  Cost Estimations

### Lessons Learned: Do courses - always include hands on parts

- For non-technical people:
  - The Course by Nng
  - then some hands one with Azure portal ressources?
- For technical people:
  - Hackathons..
- Use the Allianz Boost!

### Lessons Learned: Managing Data Parsing Costs

- Parsing: High-quality data parsing incurs high costs. For example, parsing 20 PDFs (87MB) led to a $30 charge.
  - Microsoft's Advice: Use the local PDF parser to avoid such costs. Incorporate `--localpdfparser` in `prepdocs.sh`. This option is free and can be sufficient. [Discussion on Data Parsing Costs](https://github.com/microsoft/AI-Chat-App-Hack/discussions/45).
  - Note: OCR is notably expensive and should be used sparingly.
- SKUs: Keeping the data size small is crucial for cost management. SKUs fix costs can reach very high. See pricing details for [Azure Cognitive Search](https://azure.microsoft.com/en-gb/pricing/details/search/).
- The openAi model was not the cost driver in my project but in real world it might be as well depending on the usage. You might want to implement a strategy to watch the toke usage and set up alerts.
- It's worth it to spend more time on data preparation to avoid unnecessary costs. Do a proof of concept to evaluate this costs. It's worth it to spend time on this!
  - create an example case with the needed infra. Check cost drivers on a limited set.
  - make sure you can identify the cost drivers and set up alerts for them.

### Lessons Learned: User Interaction to GPT Model

- You probably do not want to give the user direct access to the GPT model behind for minimum two reasons:
  - the results of the chat are heavily dependent on the quality of the query. In this current architecture, the query will be send directly to the Knowledge Base. The results back for this are often not very good, if it's not formulated in a specific way, easy to consume for the knowledge base. Probably it's better to have a middleware which can help to improve the query before it's send to the knowledge base.
  - You don't want to expose the knowledge base to the user. It's a security risk. You want to have a middleware which can help to improve the query before it's send to the knowledge base.
- For this, Cognigy can acutally bring a great value, if it will be transformed into an orchstrator for knowledge base & gpt.

### Lessons Learned: Cogingy
They are doing a lot of things which is described in these Azure Ressources

- Knowlegd Base is basically the same as the RAG repository (https://github.com/Azure-Samples/azure-search-openai-demo/tree/main)
- 

##### Evaluation if data was read correctly

- There might be a lot of noise in the data....

....

### Further Improvements

- Implement the Text Recognition for "Fraktur" (old german font) by using Transkribus OCR <https://readcoop.eu/transkribus/docu/rest-api/upload/>.


[def]: ttps://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/appcomponents.pn


### Hot to put the chatbot into production

[Setting this bot into production](https://github.com/Azure-Samples/azure-search-openai-demo/blob/main/docs/productionizing.md)


### Next Learning steps

- Lang Chain: This is a middleware which can be used to connect the Retriver and the LLM. It can also be used to cache the results of the Retriver to speed up the process. It can be also done in pure Python but there are libaries which can help you with that. (Langchain, LLammaindex, Semantic Kernel, etc.)
- Azure OpenAI Design Patterns:
  - Knowledge Search with Embeddings => this is Azure Cognitive Search Database https://github.com/ruoccofabrizio/azure-open-ai-embeddings-qna 
- Prompt Engineering: