---
layout: archive
title: "A Simple RAG Application with StreamLit, Langchain, Mistral and Ollama"
permalink: /simple_rag_app/
author_profile: true
---

# RAG application with StreamLit, Ollama, Langchain with Mistral

Let's build a very simple RAG application that allows us to chat with a pdf file. We will use Mistral as the LLM, Ollama top create a local Mistral LLM server, Langchain as the library that makes it all happen with the least amount of work and StreamLit as the front end. We will go through the following steps to make it all happen.


* We will start by downloading and installing [Ollama](https://ollama.com/). It allows us to create a very fast and simple LLM server locally. They already support models such as Mistral, Llama, Phi, Vicuna etc. The platform is written in Go; and you only need more or less 8 GBs of RAM to run any of the models. No GPU is necessary! Read kore form their github [here](https://github.com/ollama/ollama)

* I personally love conda environments so I will go ahead with creating a conda environment and working inside of that. You can download the requirements.txt file from [here](https://github.com/RahatIbnRafiq/simple_rag_app).

## Setting up Ollama

Install Ollama and then open up terminal and execute `ollama pull mistral` to pull the mistral model. If everything goes alright, `ollama list` should list all the models that you currently have in your machine. Then to start the Ollama server, execute `ollama serve`. If you see errors such as ``Error: listen tcp 127.0.0.1:11434: bind: address already in use``, that means the server is alreayd in action, so you should be fine.

## Architecture

![Application Architecture](https://rahatibnrafiq.github.io/images/simple_rag_app.png "Our Application Architecture")


## Repo link
Code for this can be found [here](https://github.com/RahatIbnRafiq/simple_rag_app)


## Backend

The `reg_backend.py` file contains the langchain powered backend. Let's go overthe code to understand what each part of it is doing.

```python
class ChatPDF:
    vector_store = None
    retriever = None
    chain = None

    def __init__(self):
        self.model = ChatOllama(model="mistral")
        self.text_splitter = RecursiveCharacterTextSplitter(chunk_size=1024, chunk_overlap=100)
        self.prompt = PromptTemplate.from_template(
            """
            <s> [INST] You are an assistant for question-answering tasks. Use the following pieces of retrieved context 
            to answer the question. If you don't know the answer, just say that you don't know. Use three sentences
             maximum and keep the answer concise. [/INST] </s> 
            [INST] Question: {question} 
            Context: {context} 
            Answer: [/INST]
            """
```

Creating a class called ChatPDF. Initialize it's model as the Mistral from Ollama. Initialize langchain's text splitter with chunk size of 1024 and overlap of 100 characters for each consecutive chunks. Then create the system prompt template that we will always feed to the model to get the user queries' answers. We are using the `[INST] tag to make the the model understands our intent: we are trying to get answers from a given context.


```python
def ingest(self, pdf_file_path: str):
        docs = PyPDFLoader(file_path=pdf_file_path).load()
        chunks = self.text_splitter.split_documents(docs)
        chunks = filter_complex_metadata(chunks)

        vector_store = Chroma.from_documents(documents=chunks, embedding=FastEmbedEmbeddings())
        self.retriever = vector_store.as_retriever(
            search_type="similarity_score_threshold",
            search_kwargs={
                "k": 3,
                "score_threshold": 0.5,
            },
        )

        self.chain = ({"context": self.retriever, "question": RunnablePassthrough()}
                      | self.prompt
                      | self.model
                      | StrOutputParser())
```

The ingest method of `ChatPDF` class is probably the main worker here. It reads the pdf files uploaded by the user, does the chunking, uses `FastEmbeddings` to generate the text embeddings for each chunk, and then stores the embeddings and texts in a database instance powered by `Chroma`. When doing the database search, the system uses `similarity score threshold` to get the top `3` chunks with similarity score higher than `0.5` and addas them as the context for the user query to be fed to the ML model chain.

```python
def ask(self, query: str):
        if not self.chain:
            return "Please, add a PDF document first."

        return self.chain.invoke(query)
```

Invokes the chain that was created in the `ingest` method. Straightforward.



## Frontend

We will use `Streamlit` to  implement the frontend part of the application. the file `streamlit_frontend.py` file has the code.

```python
def page():
    if len(st.session_state) == 0:
        st.session_state["messages"] = []
        st.session_state["assistant"] = ChatPDF()

    st.header("ChatPDF")

    st.subheader("Upload a document")
    st.file_uploader(
        "Upload document",
        type=["pdf"],
        key="file_uploader",
        on_change=read_and_save_file,
        label_visibility="collapsed",
        accept_multiple_files=True,
    )

    st.session_state["ingestion_spinner"] = st.empty()

    display_messages()
    st.text_input("Message", key="user_input", on_change=process_input)
```

First, have the session's assistant and chat messages initialized. Then when the user uploads a pdf, invoke the `read_and_save_file` method that processes the pdf into chunks and stores in the DB.
When the user finishes typing a query and presses enter, invoke the `process_input` method.

```python
def process_input():
    if st.session_state["user_input"] and len(st.session_state["user_input"].strip()) > 0:
        user_text = st.session_state["user_input"].strip()
        with st.session_state["thinking_spinner"], st.spinner(f"Thinking"):
            agent_text = st.session_state["assistant"].ask(user_text)

        st.session_state["messages"].append((user_text, True))
        st.session_state["messages"].append((agent_text, False))



def read_and_save_file():
    st.session_state["assistant"].clear()
    st.session_state["messages"] = []
    st.session_state["user_input"] = ""

    for file in st.session_state["file_uploader"]:
        with tempfile.NamedTemporaryFile(delete=False) as tf:
            tf.write(file.getbuffer())
            file_path = tf.name

        with st.session_state["ingestion_spinner"], st.spinner(f"Ingesting {file.name}"):
            st.session_state["assistant"].ingest(file_path)
        os.remove(file_path)
```

`process_input` takes the user input and takes the assistant that was initialized as the Mistral instance of the ChatPDF class and calls the `ask` method.

`read_and_save_file` takes the user uploaded pdf aand calls the assistant's `ingest` method to start chunking it and store it into the `chroma` DB.


Now, if you run `streamlit run streamlit_frontend.py`, the application will be online!