# Start The First Inference Job on AIbro

<aside class="success">
Play around with the executable Colab tutorial <a href = "https://colab.research.google.com/drive/1NH2Aj1bbCgqJXyNKK9TkzwlO4JDBkTQC?usp=sharing"> here</a>
</aside>

## Step 1: Install

```python
pip install aibro
```

The first step is to install the [Aibro Python library](https://pypi.org/project/aibro/) using `pip`.

During the installation, if the `OSError: protocol now found` message appears, then it indicates an error caused by a missing file that can be easily resolved. The missing file is `/etc/protocols`, and entering the following command should remedy it.

`sudo apt-get -o Dpkg::Options::="--force-confmiss" install --reinstall netbase`

## Step 2: Prepare repository

The second step is to prepare a formatted inference model repository. The following instructions and source code can be found on our GitHub page, [Aibro-examples](https://github.com/AIpaca-Inc/Aibro-examples).

The repo should be structured using the following format:

repo <br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[predict.py](#predict-py)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[model](#39-model-39-and-39-data-39-folders)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[data](#39-model-39-and-39-data-39-folders)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[requirement.txt](#requirement-txt)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;|\_\_&nbsp;[other artifacts](#other-artifacts)<br/>

### **predict.py**

This is the entry point that will be called by Aibro. It should contain two methods, `load_model()` and `run(...)`.

#### _load_model()_:

```python
def load_model():
    # Portuguese to English translator
    translator = tf.saved_model.load('model')
    return translator
```

This method is required to load and return your machine learning model from the `model` folder. As an example, a transformer-based Portuguese to English translator is used.

#### _run()_:

This method accepts the model as input. It loads the data from the `data` folder, generates predictions, and returns the results of the inference.

```python
def run(model):
    fp = open("./data/data.json", "r")
    data = json.load(fp)
    sentence = data["data"]
    result = {"data": model(sentence).numpy().decode("utf-8")}
    return result
```

</br></br></br></br></br></br></br>

_test tip_: predict.py() should be able to return an inference result by:

```python
run(load_model())
```

### **'model' and 'data' folders**

There are no format restrictions on these two folders, as long as the input and output of `load_model()` and `run(...)` from `predict.py` are correct.

### **requirement.txt**

Prior to starting model deployment, packages from `requirement.txt` are installed as part of setting up the environment.

NOTE: If your `requirement.txt` contains paths (such as `pandas @ file:///...`), then it is the subject of an open issue with `pip freeze` in version `20.1`. As a workaround, the following line can be used:

`pip list --format=freeze > requirements.txt`

### **Other Artifacts**

This refers to all of the other files and folders.

## Step 3: Test the Repo using Dryrun

```python
from aibro.inference import Inference
api_url = Inference.deploy(
    artifacts_path = "./aibro_repo",
    dryrun = True,
)
```

Dryrun locally validates the repo structure and tests to ensure that an inference result can be successfully returned.

## Step 4: Create an inference API with one-line code

```python
from aibro.inference import Inference
api_url = Inference.deploy(
    model_name = "my_fancy_transformer",
    machine_id_config = "c5.large.od",
    artifacts_path = "./aibro_repo",
)
```

Assume the formatted model repo is saved at path `"/aibro_repo"`. It can now be used to create an inference job. The model name must be unique with respect to all current [active inference jobs](https://aipaca.ai/inference_jobs) under your profile.

In this example, we deploy a public custom model from `"./aibro_repo"` called "my_fancy_transformer" on machine "c5.large.od", using an access token for authentication.

Once the deployment is complete, an API URL is returned with the syntax: </br>

- **http://api.aipaca.ai/v1/{username}/{client_id}/{model_name}/predict** </br>

**{client_id}**: if your inference job is public, **{client_id}** is simply "public". Otherwise, **{client_id}** will indicate one of your [clients' ID](#add-clients).

In this tutorial, the API URL is:

- **http://api.aipaca.ai/v1/{username}/public/my_fancy_transformer/predict** </br>

## Step 5: Test an Aibro API with curl:

```python
curl -X POST "http://api.aipaca.ai/v1/{username}/public/my_fancy_transformer/predict" -d '{"data": "Olá"}'
# replace the {username} by your own
```

In this example, we demonstrate the use of the Aibro API using the `curl` utility. However, feel free to use whatever API tool you feel comfortable with.

**Note:** The syntax when using `curl` depends on the file type in the `data` folder. In this tutorial, we use a `JSON` file.

| File Type | syntax                                                                                                   |
| --------- | -------------------------------------------------------------------------------------------------------- |
| json      | curl -X POST {{api url}} -d '{"your": "data"}'<br/>curl -X POST {{api url}} -F file=@'path/to/json/file' |
| txt       | curl -X POST {{api url}} -d 'your data'<br/>curl -X POST {{api url}} -F file=@'path/to/txt/file'         |
| csv       | curl -X POST {{api url}} -F file=@'path/to/csv/file'                                                     |
| others    | curl -X POST {{api url}} -F file=@'path/to/zip/file'                                                     |

You may have observed some patterns from the syntax lookup table above. The rules are summarized as follows:

- If the file type is `JSON` or `TXT`, you can use the `-d` flag to post the string data directly.
- If the file type is either `JSON`, `TXT`, or `CSV`, you can use the `-F` flag to post the data file, specified by its path.
- If the file type is something other than `JSON`, `TXT`, or `CSV`, then you have the option of using `ZIP` to archive the entire data folder and post it using its path.

**Important!** The posted data will replace everything in the `data` folder. Therefore, the data that you post should have the same format as what was originally there.

_Tips_: If your inference time is more than one minute, we recommend either reducing the data size or increasing the `--keepalive-time` value when using `curl`.

## Step 6: Limit API Access to Specific Clients (Optional)

As the API owner, you have the option of restricting access to specific clients by assigning them a unique ID. This mitigates the risk of receiving an overwhelming number of API requests from different clients. The client ID is included with API endpoints, as shown in step 4. If no client ID is added, this inference job will be public.

```python
from aibro.inference import Inference
Inference.update_clients(
    job_id,
    add_client_ids = ["client_1", "client_2"]
)
```

## Step 7: Complete Job

Once the inference job is no longer required, to avoid unnecessary costs, please remember to close it with the `Inference.complete()` method.

```python
from aibro.inference import Inference
Inference.complete(job_id)
```
