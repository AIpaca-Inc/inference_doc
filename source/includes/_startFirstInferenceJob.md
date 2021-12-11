# Start The First Inference Job on AIbro

<aside class="success">
Play around the executable Colab tutorial <a href = "https://colab.research.google.com/drive/19sXZ4kbic681zqEsrl_CZfB5cegUwuIB#forceEdit=true&sandboxMode=true&scrollTo=Et8ivBtkckme"> here</a>
</aside>

## Step 1: Install

```python
pip install aibro
```

Install [aibro python library](https://pypi.org/project/aibro/) by pip.

If `OSError: protocol not found` shows up, it is caused by missing `/etc/protocols` file. This command should be able to resolve the error: `sudo apt-get -o Dpkg::Options::="--force-confmiss" install --reinstall netbase`

## Step 2: Prepare a formatted model repo

Source: [https://github.com/AIpaca-Inc/AIbro_model_repo](https://github.com/AIpaca-Inc/AIbro_model_repo).

The repo should be structured in the following format:

repo <br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[predict.py](#predict-py)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[model](#39-model-39-and-39-data-39-folders)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[data](#39-model-39-and-39-data-39-folders)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[requirement.txt](#requirement-txt)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[other artifacts](#other-artifacts)<br/>

### **predict.py**

This is the entry point of AIbro.

predict.py should contain two methods:

#### _load_model()_:

```python
def load_model():
    # Portuguese to English translator
    translator = tf.saved_model.load('model')
    return translator
```

this method should load and return your machine learning model from the "model" folder. A transformer-based Portuguese to English translator is used in this example repo.

#### _run()_:

```python
def run(model):
    fp = open("./data/data.json", "r")
    data = json.load(fp)
    sentence = data["data"]

    result = {"data": model(sentence).numpy().decode("utf-8")}
    return result
```

this method uses a model as the input, load data from the "data" folder, predict, then return the inference result.
</br></br></br></br></br></br></br>

_Local test tip_: predict.py() should be able to return an inference result by:

```python
run(load_model())
```

### **'model' and 'data' folders**

There is no format restriction on the "model" and "data" folder as long as the input and output of load_model() and run() from predict.py are correct.

### **requirement.txt**

Before starting deploying the model, packages from requirement.txt are installed to set up the environment.

### **Other Artifacts**

All other files/folders.

## Step 3: Create an inference API with one-line code

```python
from aibro import Inference
Inference.deploy(
    model_name = "my_fancy_transformer",
    machine_id_config = "c5.large.od",
    artifacts_path = "./aibro_repo",
    access_token = "secret_e0000000bd614c0000000b1a8a9cadb3",
    client_ids = [], # if no clients are specified, the inference job becomes public
)
```

Assume the formatted model repo is saved at path "./aibro_repo", we can now use it to create an inference job. The model name should be unique respect to all current [active inference jobs](https://aipaca.ai/inference_jobs) under your profile.

In this example, we deployed a public custom model from "./aibro_repo" called "my_fancy_transformer" on machine type "c5.large.od" and used access token for authentication.

Once the deployment finished, an API URL is returned with the syntax: </br>

- **https://aipaca.ai/v1/{username}/{client_id}/{model_name}/predict** </br>

**{client_id}**: if your inference job is public, **{client_id}** is filled by "public". Otherwise, **{client_id}** should be filled by one of your [clients' ID](#add-clients).
