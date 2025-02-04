# Local Mode<a name="pipelines-local-mode"></a>

SageMaker Pipelines local mode is an easy way to test your training, processing and inference scripts, as well as the runtime compatibility of [pipeline parameters](https://sagemaker.readthedocs.io/en/stable/amazon_sagemaker_model_building_pipeline.html#pipeline-parameters) before you execute your pipeline on the managed SageMaker service\. By using local mode, you can test your SageMaker pipeline locally using a smaller dataset\. This allows quick and easy debugging of errors in user scripts and the pipeline definition itself without incurring the costs of using the managed service\.

Pipelines local mode leverages [SageMaker jobs local mode](https://sagemaker.readthedocs.io/en/stable/overview.html#local-mode) under the hood\. This is a feature in the SageMaker Python SDK that allows you to run SageMaker built\-in or custom images locally using Docker containers\. Pipelines local mode is built on top of SageMaker jobs local mode\. Therefore, you can expect to see the same results as if you were running those jobs separately\. For example, local mode still uses Amazon S3 to upload model artifacts and processing outputs\. If you want data generated by local jobs to reside on local disk, you can use the setup mentioned in [Local Mode](https://sagemaker.readthedocs.io/en/stable/overview.html#local-mode)\.

Pipeline local mode currently supports the following step types:
+ [Training Step](build-and-manage-steps.md#step-type-training)
+ [Processing Step](build-and-manage-steps.md#step-type-processing)
+ [Transform Step](build-and-manage-steps.md#step-type-transform)
+ [Model Step](https://docs.aws.amazon.com/sagemaker/latest/dg/build-and-manage-steps.html#step-type-model-create) \(with Create Model arguments only\)
+ [Condition Step](build-and-manage-steps.md#step-type-condition)
+ [Fail Step](build-and-manage-steps.md#step-type-fail)

As opposed to the managed Pipelines service which allows multiple steps to execute in parallel using [Parallelism Configuration](https://sagemaker.readthedocs.io/en/stable/workflows/pipelines/sagemaker.workflow.pipelines.html#parallelism-configuration), the local pipeline executor runs the steps sequentially\. Therefore, overall execution performance of a local pipeline may be poorer than one that runs on the cloud \- this mostly depends on the size of the dataset, algorithm, as well as the power of your local computer\. Also note that Pipelines runs in local mode are not recorded in [SageMaker Experiments](https://docs.aws.amazon.com/sagemaker/latest/dg/pipelines-experiments.html)\.

**Note**  
Pipelines local mode is not compatible with SageMaker algorithms such as XGBoost\. If you to want use these algorithms, you must use them in [script mode](https://sagemaker-examples.readthedocs.io/en/latest/sagemaker-script-mode/sagemaker-script-mode.html)\.

In order to execute a pipeline locally, the `sagemaker_session` fields associated with the pipeline steps and the pipeline itself need to be of type `LocalPipelineSession`\. The following example shows how you can define a SageMaker pipeline to execute locally\.

```
from sagemaker.workflow.pipeline_context import LocalPipelineSession

local_pipeline_session = LocalPipelineSession()

pytorch_estimator = PyTorch(
    sagemaker_session=local_pipeline_session,
    role=sagemaker.get_execution_role(),
    instance_type="ml.c5.xlarge",
    instance_count=1,
    framework_version="1.8.0",
    py_version="py36",
    entry_point="./entry_point.py",
)

step = TrainingStep(
    name="MyTrainingStep",
    step_args=pytorch_estimator.fit(
        inputs=TrainingInput(s3_data="s3://my-bucket/my-data/train"),
    )
)

pipeline = Pipeline(
    name="MyPipeline",
    steps=[step],
    sagemaker_session=local_pipeline_session
)

pipeline.create(
    role_arn=sagemaker.get_execution_role(), 
    description="local pipeline example"
)

// pipeline will execute locally
pipeline.start()

steps = pipeline.list_steps()

training_job_name = steps['PipelineExecutionSteps'][0]['Metadata']['TrainingJob']['Arn']

step_outputs = pipeline_session.sagemaker_client.describe_training_job(TrainingJobName = training_job_name)
```

Once you are ready to execute the pipeline on the managed SageMaker Pipelines service, you can do so by replacing `LocalPipelineSession` in the previous code snippet with `PipelineSession` \(as shown in the following code sample\) and rerunning the code\.

```
from sagemaker.workflow.pipeline_context import PipelineSession

pipeline_session = PipelineSession()
```