# aibro.Inference Methods

## deploy()

```python
def deploy(
    artifacts_path: Union[str, list],
    model_name: str = None,
    machine_id_config: Union[str, dict] = None,
    dryrun: bool = False,
    cool_down_period_s: int = 600,
    client_ids: List[str] = [],
    access_token: str = None,
    description: str = "",
    wait_request_s: int = 30,
) -> str:
```

The `deploy()` method starts an inference job, deploying models on cloud instances.

### Parameters

**artifacts_path**: _Union[str, list]_ <br/>
The path to the formatted repository. The input type can be either of type **string** or **list**:

| Type | Syntax                                                                                                                                                 | Meaning                                                                                                                                            |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| str  | `"path/to/repo"`                                                                                                                                       | The whole repo would be uploaded and deployed on cloud                                                                                             |
| list | `["./path/to/model",`<br/> `"./path/to/predict.py",`<br/> `"./path/to/data",`<br/> `"./path/to/requirements.txt",`<br/> `"./path/to/other_artifacts"]` | Select important artifacts one-by-one. Aibro will combine and recreate a model repository called `aibro_repo` under the root path of aibro library |

**model_name**: _str_ = None <br/>
This parameter specifies the model name used in the deployed inference job. It can only be set to None if, and only if dryrun is set to `False`. Within the scope of the user profile, the model name should be unique with respect to those among the active inference jobs.

**machine_id_config**: _Union[str, dict]_ <br/>
This parameter specifies the machine configuration to be used to deploy the model. In the configuration, machines are categorized as either **Standby** or **Cooling**.

- A **Standby** instance is one that never turns off.
- A **Cooling** instance is one that only turns on when it receives a new inference request, and only turns off when no inference request is received within `cool_down_period_s` seconds.

The macine_id_config can be set to `None` if and only `dryrun` is set to `False`. The input type can be either a string or a dictionary:

| Type | Syntax                                                     | Meaning                                                                                                                         |
| ---- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| str  | `"c5.large.od"`                                            | use the on-demand "c5.large" instance as the standby instance                                                                   |
| dict | `{"standby": "c5.large.od"}`                               | use the on-demand "c5.large" instance as the standby instance                                                                   |
| dict | `{"standby": "c5.large.od", "cooling": "g4dn.4xlarge.od"}` | use the on-demand "c5.large" instance as the standby instance and the on-demand "g4dn.4xlarge" instance as the cooling instance |

<u>Important:</u>

1. <u>The machine id has to be on-demand (ends by `.od`).</u>
2. <u>One standby instance is mandatory whereas cooling instance is optional (e.g. syntax such as {"cooling": "g4dn.4xlarge.od"} is invalid).</u>

**For more details about the usage of standby and cooling instances, check out the [section below](#standby-and-cooling-instances).**

<aside class="notice">
<span style="font-weight: bold">FAQ: Why does `machine_id_config` only accept on-demand instances? Isn't it more expensive than using a spot instance?</span> <br/>

Answer: Spot instances are not always available, so on-demand instances are required in order to guarantee that the job can always get a server. If spot instances are available, Albro automatically replaces its same-type on-demand instances to operate more cost-effectively.

</aside>

**dryrun**: _bool = False_<br/>
The `dryrun` option is used to perform a local test of the model. When set to `True`, the `deploy()` method will validate the structure of the repository and test whether the inference result can be successfully returned.

**cool_down_period_s**: _int = 600_<br/>
This parameter specifies the cool-down period of a cooling instance. By default, the cooling instance will stop after there have been no new inference requests for 600 seconds (10 minutes).

**client_ids**: _List[str] = []_<br/>
This argument is used to restrict access to your inference API, for use only by specific clients. The client IDs can be customized in any syntax, provided that there is no duplication. If there are no client IDs specified then the inference job is public.

The `Inference.update_clients()` method can be used to add/remove client IDs. As mentioned in the tutorial, the client ID is used as a part of the API URL.

The table below defines each role.

| Role       | Description                                   |
| ---------- | --------------------------------------------- |
| API owner  | The person that created the inference         |
| API client | Individuals that have access to the inference |

**access_token**: _str_ = None <br/>
The access token is used to [authenticate](#authentication) the API request. If its value is `None`, the client’s email and password are required for the request to be accepted.

**description**: _str_ = "" <br/>
The description is used to briefly explain the inference, allowing users to better distinguish them.

**wait_request_s**: _int = 30_<br/>
This parameter specifies the number of seconds that Aibro will wait for instance requests to be fulfilled.

## complete()

```python
def complete(
    model_name: str = None,
    job_id: str = None,
    access_token: str = None,
):
```

The `complete()` method signals the end of an inference, shutting down all of its API services.

### Parameters

**model_name**: _str_ = None <br/>
This parameter is used to identify the inference job to be stopped. Since a model name is required to be unique, it can be used instead of the `job_id` to locate it. If values for both `model_name` and `job_id` are specified then the `job_id` will be used for the search.

**job_id**: _str_ = None <br/>
This identifies the deployed inference job using the job ID. Specifying an ID will cause the `model_name` parameter to be ignored.

**access_token**: _str_ = None <br/>
The access token is used to [authenticate](#authentication) the API request. If its value is None, the client’s email and password are required for the request to be accepted.

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

The `update_clients()` method is used to modify the list of authorized client IDs for the inference job.

### Parameters

**add_client_ids**: _Union[str, List[str]]_ = [] <br/>
This parameter will add a single client ID, or a list of client IDs. If the duplicate IDs are found within the list or have already been authorized then the update will be canceled.

**remove_client_ids**: _Union[str, List[str]]_ = [] <br/>
This parameter will remove a single client ID or a list of client IDs from the set of authorized clients. IDs that are not found will be ignored.

**be_public**: _bool_ = False <br/>
Setting the `be_public` parameter to `True` removes all of the client IDs from the access list, leaving the inference job accessible to all.

**model_name**: _str_ = None <br/>
This parameter is used to identify the model within the inference job. Since a model name is required to be unique, it can be used instead of the `job_id` to locate it. If values for both `model_name` and `job_id` are specified then the `job_id` will be used for the search.

**job_id**: _str_ = None <br/>
This identifies the deployed inference job using the job ID. Specifying an ID will cause the `model_name` parameter to be ignored.

**access_token**: _str_ = None <br/>
The access token is used to [authenticate](#authentication) the API request. If its value is `None`, the client’s email and password are required for the request to be accepted.

## list_clients()

```python
def list_clients(
    model_name: str = None,
    job_id: str = None,
    access_token: str = None,
)->List[str]:
```

The `list_clients()` method returns a list of client IDs for a specific inference job.

### Parameters

**model_name**: _str_ = None <br/>
This parameter is used to identify the model within the inference job. Since a model name is required to be unique, it can be used instead of the `job_id` to locate it. If values for both `model_name` and `job_id` are specified then the job_id will be used for the search.

**job_id**: _str_ = None <br/>
This identifies the deployed inference job using the job ID. Specifying an ID will cause the `model_name` parameter to be ignored.

**access_token**: _str_ = None <br/>
The access token is used to [authenticate](#authentication) the API request. If its value is `None`, the client’s email and password are required for the request to be accepted.

## Standby and Cooling instances

### Definitions:

Instances are classified in one of two ways; **Standby** or **Cooling**.

- A **Standby** instance is one that never turns off.
- A **Cooling** instance is one that only turns on when it receives a new inference request, and only turns off when no inference request is received within `cool_down_period_s` seconds.

If both standby and cooling instances are turned on, the cooling instance would have a higher priority to handle upcoming inference requests.

### Configure for non-uniform traffic

Dealing with non-uniform traffic and having the ability to scale are important features of the system, and it is helpful to understand how sporadic and non-uniform traffic can affect your strategy. In simple words, the combination of standby and cooling instances, properly configured, will lead you toward optimal pricing for your usage.

Consider that you have an inference API for a web application and the call frequency is proportional to the site traffic. If the traffic is typically non-uniform, such as the case where there is more traffic during the day and relatively little during the night, then the configuration should be set accordingly. To take advantage of known traffic patterns, in this case, you might set a CPU or cheaper GPU instance as the standby instance, and dedicate a powerful GPU for the cooling instance. In this approach, the powerful cooling instance would efficiently handle intensive traffic during the day and the standby instance would save you on costs because it is running at night. Importantly, this balance maintains the near real-time performance.

### Configure for high traffic

On the other hand, some sites have traffic that is uniform. Constant traffic is common, for example, in applications that are depended on by clients operating in many different time zones. If your site traffic is more uniform, and people never stop using it, using standby instances with a powerful GPU is the recommended configuration.

The reason for this becomes clear when you consider the previous configuration, which uses a weak standby processor and a powerful cooling one. With constant traffic, the cooling instance would never shut down because the timeout period would never expire. Consequently, the lesser-powered standby processor would not be utilized and thus wasted.

### Case study: calculate the saving

![](https://drive.google.com/uc?export=view&id=1sLxQ6SZDamjT9qGp2JE7zElUZwIpLQ3I)

**Graph explanation**: This graph describes an inference job with configuration `machine_id_config = machine_id_config = {"standby": "g4dn.4xlarge", "cooling": "g4dn.12xlarge.od"}`. The job was operated between 8:00 am and 11.59 pm, and it is clear that most of the inference requests (IRs) were received during the morning and afternoon.

Every time an IR was received by the cooling instance, the cooldown period reset. If the cooldown period passed by without receiving an IR, the instance stopped, and the standby instance handled the next IR. At that time, the standby invoked the previously stopped cooling instance. During the period it took the cooling instance to fully come online, the standby instance processed the IRs.

Over the course of the 16 hours, cooling instances were turned on for 6 hours and the standby instance was never turned off. The relevant machine pricing is shown below:

| Machine id       | Pricing  |
| ---------------- | -------- |
| g4dn.12xlarge.od | $3.91/hr |
| g4dn.4xlarge.od  | $1.20/hr |
| g4dn.12xlarge    | $1.61/hr |
| g4dn.4xlarge     | $0.36/hr |

**Let's compare the savings with and without the hybrid configuration.**

| Configuration           | machine_id_config                                                  | Cost                          | Spot Cost |
| ----------------------- | ------------------------------------------------------------------ | ----------------------------- | --------- |
| Without standby&cooling | `{"standby": "g4dn.12xlarge.od"}`                                  | 3.91 \* 16 = $62.56           | $25.76    |
| With standby&cooling    | `{"standby": "g4dn.4xlarge",`<br/>`"cooling": "g4dn.12xlarge.od"}` | 1.2\_ 16 + 3.91 \* 6 = $42.66 | $15.42    |

In this scenario, the savings from using the hybrid standby and cooling configuration was more than 30%. Furthermore, if spot instances were available over the period, Albro would further cut the cost from $62.56 to $15.42. This is a savings of more than 75%!

## Inference Job & Instance Status

Once a job starts, its states and substates are updated on the [Jobs page within the Aibro Console](https://aipaca.ai/inference_jobs).

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
| CANCELED          | Canceled due to errors                  |
| COMPLETED         | Completed the job                       |

| Instance Status | Description                                                                          |
| --------------- | ------------------------------------------------------------------------------------ |
| LAUNCHING       | Setting up instance for inference                                                    |
| EXECUTING       | Jobs being processed and ready to receive requests                                   |
| COOLING         | Within cooling Period (must be a [cooling instance](#standby-and-cooling-instances)) |
| CLOSING         | Stopping/terminating instance                                                        |
| CLOSED          | Instance has been stopped/terminated                                                 |

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
