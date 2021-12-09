# aibro.Inference

## deploy()

```python

```

### Parameters

## complete()

```python
def complete(
    model_name: str = None,
    job_id: str = None,
    access_token: str = None,
):
```

Completing an inference job shuts down all instances, and the inference API stop serving immediately.

### Parameters

**model_name**: _str_ = None <br/>
The model name used in the deployed inference job. Since model name is unique, it can be used to search which job to complete. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**job_id**: _str_ = None <br/>
The job_id of the deployed inference job. You may only need to input either `model_name` or `job_id`. If both parameters have input, `job_id` would be used to search inference job.

**access_token**: _str_ = None <br/>
The access token used in [authentication](#authentication). If its value is None, email and password would be required to input.

### Get more saving by cooling instance

### Inference Job & Instance Status

Once a job starts, its states and substates are updated on the [Jobs page of AIbro Console](https://aipaca.ai/jobs).

| Job Status | Description                                           |
| ---------- | ----------------------------------------------------- |
| QUEUING    | Waiting for training                                  |
| TRAINING   | During training process or returning training results |
| CANCELED   | Canceled due to some errors                           |
| COMPLETED  | Completed the job                                     |

| Job Substatus        | Description                                     |
| -------------------- | ----------------------------------------------- |
| REQUESTING SERVER    | Requesting an instance to train models          |
| CONNECTING SERVER    | Connecting an initializing instance             |
| GEARING UP ENV       | Gearing up tensorflow and mounting GPUs         |
| SENDING MODEL & DATA | Sending model and training data to the instance |
| TRAINING             | Training model                                  |
| RETURNING            | Returning trained model                         |
| CANCELED             | Canceled due to some errors                     |
| COMPLETED            | Completed the job                               |

| Instance Status | Description                              |
| --------------- | ---------------------------------------- |
| LAUNCHING       | Setting up instance for training         |
| EXECUTING       | Having jobs in training process          |
| COOLING         | Within [Cooling Period](#cooling-period) |
| CLOSING         | Stopping/terminating instance            |
| CLOSED          | instance has been stopped/terminated     |

| Instance Substatus     | Description                                                                                                                |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| STOPPING/STOPPED       | Shut down instance but retain root volume [Reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Stop_Start.html) |
| TERMINATING/TERMINATED | Completely delete the instance [Reference](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html) |

In the AIbro usage case, [Setup speed](#setup-speed) is the main advantage of stopped instance over terminated instance.

The following table is a status-substatus map of jobs and instances.

| Job Status | Job Substatus        | Instance Status        | Instance Substatus                                    |
| ---------- | -------------------- | ---------------------- | ----------------------------------------------------- |
| QUEUING    | REQUESTING SERVER    |                        |                                                       |
| QUEUING    | CONNECTING SERVER    | LAUNCHING              |                                                       |
| QUEUING    | GEARING UP ENV       | LAUNCHING              |                                                       |
| QUEUING    | SENDING MODEL & DATA | LAUNCHING              |                                                       |
| ---------- | ------------         | ---------------------- | --------------------------                            |
| TRAINING   | TRAINING             | EXECUTING              |                                                       |
| TRAINING   | RETURNING            | EXECUTING              |                                                       |
| ---------- | ------------         | ---------------------- | --------------------------                            |
| CANCELED   | CANCELED             | COOLING/CLOSING/CLOSED | COOLING/(STOPPING, TERMINATING)/(STOPPED, TERMINATED) |
| COMPLETED  | COMPLETED            | COOLING/CLOSING/CLOSED | COOLING/(STOPPING, TERMINATING)/(STOPPED, TERMINATED) |
