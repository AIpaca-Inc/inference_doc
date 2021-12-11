# aibro.Inference

## deploy()

```python
def deploy(
    model_name: str,
    machine_id_config: Union[str, dict],
    artifacts_path: Union[str, list],
    dryrun: bool = False,
    cool_down_period_s: int = 600,
    client_ids: List[str] = [],
    access_token: str = None,
    description: str = "",
    wait_request_s: int = 30,
) -> str:
```

This method starts an inference job to deploy models on cloud instances.

### Parameters

**model_name**: _str_ <br/>
The model name used in the deployed inference job. In the scope of user profile, the model name should be unique respect to the model names from all active inference jobs.

**machine_id_config**: _Union[str, dict]_ <br/>
The machine configuration used to deploy the model. The input type can be either string or dictionary:

| Type | Syntax                                                     | Meaning                                                                                                                         |
| ---- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| str  | `"c5.large.od"`                                            | use the on-demand "c5.large" instance as the standby instance                                                                   |
| dict | `{"standby": "c5.large.od"}`                               | use the on-demand "c5.large" instance as the standby instance                                                                   |
| dict | `{"standby": "c5.large.od", "cooling": "g4dn.4xlarge.od"}` | use the on-demand "c5.large" instance as the standby instance and the on-demand "g4dn.4xlarge" instance as the cooling instance |

<u>Important:</u>

1. <u>The machine id has to be on-demand (ends by `.od`).</u>
2. <u>One standby instance is mandatory whereas cooling instance is optional (e.g. syntax such as {"cooling": "g4dn.4xlarge.od"} is invalid).</u>

In the configuration, machines are categorized into "Standby" and "Cooling". In general, their definitions are below:

- Standby instance: the instance never turns off.
- Cooling instance: the instance only turns on when receiving a new inference request and only turns off when no inference request is received within `cool_down_period_s` seconds.

For more details about the usage of standby and cooling instances, checkout the [section below](#standby-and-cooling-instances).

<aside class="notice">
<span style="font-weight: bold">FAQ: Why machine_id_config only accept on-demand instances? Isn't it more expensive than spot?</span> <br/>

Answer: spot instance is not always available, using on-demand instance is to ensure that inference job can always get a server. If spot instances are available, AIbro automatically replaces its same-type on-demand instances to save your cost.

</aside>

**artifacts_path**: _Union[str, list]_ <br/>
The path to the formatted repo. The input type can be either string or dictionary:

| Type | Syntax                                                                                                                                                 | Meaning                                                                                                                                       |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| str  | `"path/to/repo"`                                                                                                                                       | The whole repo would be uploaded and deployed on cloud                                                                                        |
| list | `["./path/to/model",`<br/> `"./path/to/predict.py",`<br/> `"./path/to/data",`<br/> `"./path/to/requirements.txt",`<br/> `"./path/to/other_artifacts"]` | Select important artifacts one-by-one. AIbro would combine and recreate a model repo called `aibro_repo` under the root path of aibro library |

**dryrun**: _bool = False_<br/>
The argument used to test model repo locally. If `dryrun==True`, `deploy()` would start validating the repo structure and testing if inference result can be successfully returned.

**cool_down_period_s**: _int = 600_<br/>
The cool down period of cooling instance. By default, the cooling instance would stop once there is no new inference request coming within 600 seconds.

**client_ids**: _List[str] = []_<br/>
The argument used to restrict specific clients to access to your inference API. The client ids can be customized in any syntax as long as there is no duplication. If there is no client id, the inference job becomes public. `Inference.update_clients()` can be used to add/remove client ids. <br/>
As mentioned in the [tutorial](#step-3-create-an-inference-api-with-one-line-code), client id is used as a part of API URL. <br/>
The table below clarifies the definitions of roles.

| Role       | Description                                    |
| ---------- | ---------------------------------------------- |
| API owner  | People who created the inference API           |
| API client | People who have the access to an inference API |

**access_token**: _str_ = None <br/>
The access token used in [authentication](#authentication). If its value is None, email and password would be required to input.

**description**: _str_ = "" <br/>
The description used to remind which inference job was which.

**wait_request_s**: _int = 30_<br/>
The time in seconds used to wait for instance request to be fulfilled.

## complete()

```python
def complete(
    model_name: str = None,
    job_id: str = None,
    access_token: str = None,
):
```

Completing an inference job shuts down all instances and stops its API services.

### Parameters

**model_name**: _str_ = None <br/>
The model name used in the deployed inference job. Since model name is required to be unique, it can be used to search which job to complete. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**job_id**: _str_ = None <br/>
The job_id of the deployed inference job. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**access_token**: _str_ = None <br/>
The access token used in [authentication](#authentication). If its value is None, email and password would be required to input.

## update_clients()

```python
def update_clients(
        add_client_ids: Union[str, List[str]] = [],
        remove_client_ids: Union[str, List[str]] = [],
        be_public: bool = False,
        model_name: str = None,
        job_id: str = None,
        access_token: str = None,
    )->List[str]:
```

Update the client ids of the targeted inference job.

### Parameters

**add_client_ids**: _Union[str, List[str]]_ = [] <br/>
The argument used to add one single or one list of client ids. If duplicated client ids are found, the update would be canceled.

**remove_client_ids**: _Union[str, List[str]]_ = [] <br/>
The argument used to remove one single or one list of client ids. Client ids that are not found would be ignored.

**be_public**: _bool_ = False <br/>
If the argument is set `True`, it removes all client ids and the inference job becomes public.

**model_name**: _str_ = None <br/>
The model name used in the deployed inference job. Since model name is required to be unique, it can be used to search the targeted inference job. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**job_id**: _str_ = None <br/>
The job_id of the deployed inference job. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**access_token**: _str_ = None <br/>
The access token used in [authentication](#authentication). If its value is None, email and password would be required to input.

## list_clients()

```python
def list_clients(
    model_name: str = None,
    job_id: str = None,
    access_token: str = None,
)->List[str]:
```

Return a list of client ids of the targeted inference job.

### Parameters

**model_name**: _str_ = None <br/>
The model name used in the deployed inference job. Since model name is required to be unique, it can be used to search the targeted inference job. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**job_id**: _str_ = None <br/>
The job_id of the deployed inference job. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**access_token**: _str_ = None <br/>
The access token used in [authentication](#authentication). If its value is None, email and password would be required to input.

## Standby and Cooling instances

### Definitions:

- Standby instance: the instance never turns off.
- Cooling instance: the instance only turns on when receiving a new inference request and only turns off when no inference request is received within `cool_down_period_s` seconds.

If both standby and cooling instances are turned on, the cooling instance would have higher priority to handle upcoming inference requests.

### Configure for non-uniform traffic

In a simple word, the combination of standby and cooling instance configuration is a cross-instance scaling strategy. Let's say you want to build an inference API for a web application. The call frequency is proportional to the site traffic, which is usually non-uniform (e.g. more traffic during the day and a few during the night). To take the advantage of the configuration, you may set a CPU or cheaper GPU instance as the standby instance and set a powerful GPU instance as the cooling instance. In this way, the powerful cooling instance would efficiently handle intensive traffic during the day and standby instance would save your money during the night and still keep the near real-time performance.

### Configure for high traffic

On the other hands, if your site traffic is more uniform or people are just never stop using it, it is more recommended to only use standby instance with the powerful GPU. Why is that? if you kept the previous configuration which uses a weak standby and a powerful cooling, the cooling instance may just never end its cooling period since there are always requests coming. In this case, the weak standby instance is not applied at all and wasted.

### Case study: calculate the saving

![](https://drive.google.com/uc?export=view&id=1sLxQ6SZDamjT9qGp2JE7zElUZwIpLQ3I)

**Graph explanation**: Given that you created an inference job with configuration `machine_id_config = {"standby": "g4dn.4xlarge", "cooling": "g4dn.12xlarge.od"}`. The inference job was applied from 8:00 am to 11.59 pm. Most inference requests (IRs) were received during morning and afternoon. Every time an IR was received by the cooling instance, the instance's cool down period reset. If the cool down period passed by without receiving one IR, the cooling instance stopped, and the standby instance handled the next IR. At that time, the standby started invoking back the stopped cooling instance. The standby instance also processed the following IRs until the cooling instance fully woke up. Within the 16 hours, cooling instances was turned on for 6 hours and standby instance never turned off. The machine pricing are shown below:

| Machine id       | Pricing  |
| ---------------- | -------- |
| g4dn.12xlarge.od | $3.91/hr |
| g4dn.4xlarge.od  | $1.20/hr |
| g4dn.12xlarge    | $1.61/hr |
| g4dn.4xlarge     | $0.36/hr |

Let's compare the saving with and without standby&cooling configuration.

| Configuration           | machine_id_config                                                  | Cost                          | Spot Cost |
| ----------------------- | ------------------------------------------------------------------ | ----------------------------- | --------- |
| Without standby&cooling | `{"standby": "g4dn.12xlarge.od"}`                                  | 3.91 \* 16 = $62.56           | $25.76    |
| With standby&cooling    | `{"standby": "g4dn.4xlarge",`<br/>`"cooling": "g4dn.12xlarge.od"}` | 1.2\_ 16 + 3.91 \* 6 = $42.66 | $15.42    |

In this scenario, the pure saving from standby&cooling configuration was over 30%. Plus, if their spot instances were available over the period, AIbro would end up cutting the cost from $62.56 to $15.42. That was a 75% saving!

## Inference Job & Instance Status

Once a job starts, its states and substates are updated on the [Jobs page of AIbro Console](https://aipaca.ai/inference_jobs).

| Job Status | Description                       |
| ---------- | --------------------------------- |
| QUEUING    | Waiting to be deployed            |
| DEPLOYED   | Inference API is ready to be used |
| CANCELED   | Canceled due to some errors       |
| COMPLETED  | Job was completed                 |

| Job Substatus     | Description                             |
| ----------------- | --------------------------------------- |
| REQUESTING SERVER | Requesting an instance to deploy models |
| CONNECTING SERVER | Connecting an initializing instance     |
| GEARING UP ENV    | Gearing up tensorflow and mounting GPUs |
| DEPLOYING MODEL   | Deploying model                         |
| DEPLOYED MODEL    | Model was deployed                      |
| CANCELED          | Canceled due to some errors             |
| COMPLETED         | Completed the job                       |

| Instance Status | Description                                                                          |
| --------------- | ------------------------------------------------------------------------------------ |
| LAUNCHING       | Setting up instance for inference                                                    |
| EXECUTING       | Having jobs in ready to receive inference requests                                   |
| COOLING         | Within cooling Period (must be a [cooling instance](#standby-and-cooling-instances)) |
| CLOSING         | Stopping/terminating instance                                                        |
| CLOSED          | instance has been stopped/terminated                                                 |

| Instance Substatus     | Description                                                                                                                |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| STOPPING/STOPPED       | Shut down instance but retain root volume [Reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Stop_Start.html) |
| TERMINATING/TERMINATED | Completely delete the instance [Reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html) |

The following table is a status-substatus map of jobs and instances.

| Job Status | Job Substatus     | Instance Status        | Instance Substatus                                    |
| ---------- | ----------------- | ---------------------- | ----------------------------------------------------- |
| QUEUING    | REQUESTING SERVER |                        |                                                       |
| QUEUING    | CONNECTING SERVER | LAUNCHING              |                                                       |
| QUEUING    | GEARING UP ENV    | LAUNCHING              |                                                       |
| QUEUING    | DEPLOYING MODEL   | LAUNCHING              |                                                       |
| ---------- | ------------      | ---------------------- | --------------------------                            |
| DEPLOYED   | DEPLOYED          | EXECUTING/COOLING      |                                                       |
| ---------- | ------------      | ---------------------- | --------------------------                            |
| CANCELED   | CANCELED          | COOLING/CLOSING/CLOSED | COOLING/(STOPPING, TERMINATING)/(STOPPED, TERMINATED) |
| COMPLETED  | COMPLETED         | COOLING/CLOSING/CLOSED | COOLING/(STOPPING, TERMINATING)/(STOPPED, TERMINATED) |
