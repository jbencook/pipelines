# TFX Pipeline Example

This sample walks through running [TFX](https://github.com/tensorflow/tfx) [Taxi Application Example](https://github.com/tensorflow/tfx/tree/master/examples/chicago_taxi_pipeline) on Kubeflow Pipelines cluster. 

## Overview

This pipeline demonstrates the TFX capablities at scale. The pipeline uses a public BigQuery dataset and uses GCP services to preprpocess data (Dataflow) and train the model (Cloud ML Engine). The model is then deployed to Cloud ML Engine Prediction service.


## Setup

Enable DataFlow API for your GKE cluster: <https://console.developers.google.com/apis/api/dataflow.googleapis.com/overview>

Create a local Python 3.5 conda environment
```
conda create -n tfx-kfp pip python=3.5.3
```
then activate the environment.


Install TFX and Kubeflow Pipelines SDK
```
!pip3 install https://storage.googleapis.com/ml-pipeline/tfx/tfx-0.12.0rc0-py2.py3-none-any.whl 
!pip3 install https://storage.googleapis.com/ml-pipeline/release/0.1.10/kfp.tar.gz --upgrade
```

Clone TFX github repo
```
git clone https://github.com/tensorflow/tfx
```

Upload the utility code to your storage bucket. You can modify this code if needed for a different dataset.
```
gsutil cp tfx/examples/chicago_taxi_pipeline/taxi_utils.py gs://my-bucket/<path>/
```

If gsutil does not work, try `tensorflow.gfile`:
```
from tensorflow import gfile
gfile.Copy('tfx/examples/chicago_taxi_pipeline/taxi_utils.py', 'gs://<my bucket>/<path>/taxi_utils.py')
```

## Configure the TFX Pipeline

Modify the pipeline configuration file at 
```
tfx/examples/chicago_taxi_pipeline/taxi_pipeline_kubeflow_large.py
```
Configure 
- Set `_input_bucket` to the GCS directory where you've copied taxi_utils.py. I.e. gs://<my bucket>/<path>/
- Set `_output_bucket` to the GCS directory where you've want the results to be written
- Set GCP project ID (replace my-gcp-project). Note that it should be project ID, not project name.
- The original BigQuery dataset has 100M rows, which can take time to process. Modify the selection criteria (% of records) to run a sample test. 

## Compile and run the pipeline
```
python tfx/examples/chicago_taxi_pipeline/taxi_pipeline_kubeflow_large.py
```
This will generate a file named chicago_taxi_pipeline_kubeflow_large.tar.gz 
Upload this file to the Pipelines Cluster and crate a run.
