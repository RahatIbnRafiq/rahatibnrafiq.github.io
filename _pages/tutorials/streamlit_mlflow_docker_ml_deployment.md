---
permalink: /
title: "End to End ML Model Deployment"
excerpt: "End to End ML Model Deployment"
author_profile: true
redirect_from: 
  - /tutorials/end-to-end-ml-model-deployment
  - /tutorials/end-to-end-ml-model-deployment.html
---


# End-to-End ML model deployment using mlflow, streamlit, fastapi and docker

Welcome to this straightforward tutorial on end-to-end machine learning model deployment. We'll walk through the following steps to deploy a basic machine learning model:

* ### Model Creation with mlflow:
Start by utilizing the iris dataset to create a basic machine learning model. We'll leverage mlflow for effective performance tracking across multiple models.

* ### FastAPI Endpoint Setup:
Identify the best-performing machine learning model and establish a FastAPI endpoint on your local machine, accessible through port 8000. This endpoint will allow you to make POST requests with sample flower data, obtaining predictions for their species type.

* ### Integration with Streamlit:
Incorporate streamlit to seamlessly combine the backend of the machine learning model prediction with a frontend interface. This integrated application will be exposed through port 8501, providing an interactive environment for users.

* ### Dockerization for Deployment:
Conclude the deployment process by utilizing Docker to containerize both the machine learning model backend and the streamlit frontend. This involves creating two containers: one for the FastAPI backend handling predictions and another for the Streamlit frontend service. Dockerization ensures easy deployment and scalability of your end-to-end machine learning solution.



## Model Creation with mlflow

```python
import mlflow


from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

# Load the Iris dataset
data = load_iris()
X = data.data
y = data.target

# Split data into training and testing sets
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.5, random_state=42)

for n in [10,40,60,80,100]:
    mlflow.start_run()
    # Create and train a RandomForestClassifier
    clf = RandomForestClassifier(n_estimators=n)
    clf.fit(X_train, y_train)
    # Make predictions
    y_pred = clf.predict(X_test)
    # Log metrics
    accuracy = accuracy_score(y_test, y_pred)
    mlflow.log_metric("accuracy", accuracy)
    # Log parameters
    mlflow.log_params({"n_estimators": n, "random_state": 42})

    # Log the model as an artifact
    mlflow.sklearn.log_model(clf, "model")
    mlflow.end_run()
```

FYI, I created a conda environment and ran all my codes in that environment. 

The provided Python code leverages the Iris dataset to train a set of Random Forest Classifier models with varying numbers of estimators (10, 40, 60, 80, 100) using MLflow. 

This Python script showcases the use of MLflow for the end-to-end deployment of machine learning models. It employs the popular Iris dataset and RandomForestClassifier to create multiple models with different numbers of estimators. The key steps include:

* Data Loading and Splitting:
    * Loads the Iris dataset, separating features (X) and labels (y).
    * Splits the data into training and testing sets.

* Model Training Loop:
    * Iterates over a predefined list of estimator values (10, 40, 60, 80, 100).
    * Starts an MLflow run for each iteration.
    * Creates and trains a RandomForestClassifier with the specified number of estimators.
    * Makes predictions on the test set.
* Logging Metrics and Parameters:
    * Logs accuracy metrics for each model using MLflow.
    * Logs model parameters, including the number of estimators and random state.
* MLflow Run Finalization:
    * Ends the MLflow run for each iteration.

[http://127.0.0.1:5000/](http://127.0.0.1:5000/) Will show the mlflow dashboard.


How do we extract the best model?
```python
import mlflow
runs = mlflow.search_runs(experiment_ids="0")
def get_best_model(experiment_id, metric_name='accuracy'):
    runs = mlflow.search_runs(experiment_ids=experiment_id)
    best_accuracy = 0
    best_run_id = None
    for index, run in runs.iterrows():
        run_id = run['run_id']
        run_data = mlflow.get_run(run_id).data
        run_metrics = run_data.metrics

        if metric_name in run_metrics:
            accuracy = run_metrics[metric_name]
            if accuracy > best_accuracy:
                best_accuracy = accuracy
                best_run_id = run_id

    return best_run_id, best_accuracy
best_run_id, best_accuracy = get_best_model("0")
if best_run_id:
    print(f"Best model found in experiment '{0}'")
    print(f"Best Run ID: {best_run_id}")
    print(f"Accuracy: {best_accuracy}")
else:
    print("No model with the specified metric found in the experiment.")
```
The above code returns the best model by going through all the models that was run under the experiment id *0*. 

Now, we will build upon this foundation of best model to deploy the best-performing model using FastAPI, Docker, and Streamlit.



## FastAPI Endpoint Setup

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import mlflow
import pandas as pd
import numpy as np

app = FastAPI()

# Define the input data model using Pydantic
class InputData(BaseModel):
    p_length: float
    p_width: float
    s_length: float
    s_width: float


def get_model():
    model = mlflow.sklearn.load_model(f"deec5493dfea4290ae4e7e2dec81245a/artifacts/model/")
    return model

# Define a route to accept input data and return predictions
@app.post("/predict")
async def predict(input_data: InputData):
    try:
        # Convert input data to a DataFrame
        input_df = pd.DataFrame([input_data.model_dump()])
        model = get_model()
        # Make predictions using the loaded model
        prediction = model.predict(input_df)
        return {"prediction": prediction.tolist()}
    except Exception as e:
        return {"prediction": str(e)}
```

* Input Data Model:
    * class InputData(BaseModel): Define a Pydantic model (InputData) to represent the input data expected by the machine learning model. It includes fields for petal length (p_length), petal width (p_width), sepal length (s_length), and sepal width (s_width).

* MLflow Model Loading
    * def get_model(): Define a function to load the MLflow-trained machine learning model. The model is loaded using mlflow.sklearn.load_model from the specified artifact path. I just copied the best model artifact into the parent folder.

* Prediction Route:
    * `@app.post("/predict")`: Create a FastAPI route for handling HTTP POST requests at the "/predict" endpoint.
    * `async def predict(input_data: InputData)`: Define an asynchronous function to accept input data based on the InputData model.
        * Inside the function. Convert the input data to a Pandas DataFrame for compatibility with the model. Load the machine learning model using the get_model function. Make predictions on the input data using the loaded model.Return the predictions in JSON format.


Now, you can run the fastapi endpoint with this command: 

```bash 
uvicorn app:app --host 0.0.0.0 --port 8000 --reload
```
Then in the terminal, you can use `CURL` to make a POST request with a flower data as json format and check the functionality of the endpoint. An example request: 

```bash
curl -X POST "http://127.0.0.1:8000/predict" -H "accept: application/json" -H "Content-Type: application/json" -d '{"p_length": 7, "p_width": 3.0, "s_length": 4, "s_width": 1.4}'
```

Now that the backend fastapi endpoint is working, we can start working on coding up the front end using Streamlit.


## Integration with Streamlit

```python
import streamlit as st
import requests
import json

# Define the Streamlit app title and description
st.title("Machine Learning Model Deployment")
st.write("Use this app to make predictions with the deployed model.")

# Create input fields for user to enter data
st.header("Input Data")
petal_length = st.number_input("Petal Length")
petal_width = st.number_input("Petal Width")
sepal_length = st.number_input("Sepal Length")
sepal_width = st.number_input("Sepal Width")

# Create a button to trigger predictions
if st.button("Predict"):
    # Define the input data as a dictionary
    input_data = {
        "p_length": petal_length,
        "p_width": petal_width,
        "s_length": sepal_length,
        "s_width": sepal_width,
    }

    # Make a POST request to the FastAPI model
    # model_url = "http://0.0.0.0:8000/predict/"
    model_url = "http://fastapi:8000/predict"
    response = requests.post(model_url, json=input_data)

    if response.status_code == 200:
        prediction = json.loads(response.text)["prediction"]
        st.success(f"Model Prediction: {prediction}")
    else:
        st.error("Failed to get a prediction. Please check your input data and try again.")
```

A very basic streamlit page that takes the four flower features as inputs and a predict button that, once clicked, shows you back the prediction. The model endpoint url has the docker version, so when we finally dockerize the whole application, we will name the fastapi service as *fastapi*, inside the docker environment; and access that endpoint with the mentioned url. If you still want to access the ui front end, feel free to use the commented out url above that line.

## Dockerization for Deployment

Now we will dockerize the whole application. First, lets have requirements.txt file with the following dependencies:

```bash
# requirements.txt
fastapi
uvicorn
streamlit
requests
pandas
scikit-learn
mlflow
```

Then, a Dockerfile for fastapi service; Dockerfile_fastapi

```bash
FROM python:3.8

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
```

Here, we are copying everything to the container. Not a good practice, of course; but let's keep the dirty way to keep the tutorial succint.

Now, the Dockerfile_streamlit for the front end service:
```bash
FROM python:3.8

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 8501

CMD ["streamlit", "run", "ui.py"]
```

Then, the Docker compose file to start the two services accordingly:

```bash
version: '3'
services:
  fastapi:
    build:
      context: .
      dockerfile: Dockerfile_fastapi
    ports:
      - "8000:8000"

  streamlit:
    build:
      context: .
      dockerfile: Dockerfile_streamlit
    ports:
      - "8501:8501"
```

See the service name for fastapi is **fastapi**? That is what the streamlit will contact to connect to the model endpoint API.

Now, do `docker-compose build` and `docker-compose up` to build and run the docker services. Check `docker ps` to see if all the services are up and running.

Access the front end by going to [http://0.0.0.0:8501/](http://0.0.0.0:8501/)

