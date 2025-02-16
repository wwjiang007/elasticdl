# ElasticDL: A Kubernetes-native Deep Learning Framework

[![Travis-CI Build Status](https://travis-ci.com/sql-machine-learning/elasticdl.svg?branch=develop)](https://travis-ci.com/sql-machine-learning/elasticdl)
[![Code Coverage](https://codecov.io/gh/sql-machine-learning/elasticdl/branch/develop/graph/badge.svg)](https://codecov.io/gh/sql-machine-learning/elasticdl)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![PyPI Status Badge](https://badge.fury.io/py/elasticdl-client.svg)](https://pypi.org/project/elasticdl-client/)

ElasticDL is a Kubernetes-native deep learning framework
that supports fault-tolerance and elastic scheduling.

**IMPORTANT**: This repository is deprecated.

1. ElasticDL repository is no longer actively maintained. Users are encouraged to switch to [DLRover](https://github.com/intelligent-machine-learning/dlrover).
  
1. In addition to the elasticity and fault-tolerance, DLRover also implements
  [auto-scaling distributed training](https://github.com/intelligent-machine-learning/dlrover#auto-scaling-to-improve-training-performance).

1. In addition to TensorFlow and Horovod, DLRover Supports TorchElastic
  and users can use the elasticity and fault-tolerance
  without any modification of the training code like the [TorchElastic example](https://github.com/intelligent-machine-learning/dlrover/blob/master/docs/tutorial/torch_on_cloud.md).
  
1. To deploy a distributed job using `kubectl`, DLRover implements an [ElasticJob](https://github.com/intelligent-machine-learning/dlrover/tree/master/dlrover/go/operator)
  CRD.

## Main Features

### Elastic Scheduling and Fault-Tolerance

Through Kubernetes-native design, ElasticDL enables fault-tolerance and works
with the priority-based preemption of Kubernetes to achieve elastic scheduling
for deep learning tasks.

### Support TensorFlow and PyTorch

- TensorFlow Estimator.
- TensorFlow Keras.
- PyTorch

### Minimalism Interface

Given a [model](model_zoo/mnist_functional_api/mnist_functional_api.py) defined
with Keras API, train the model distributedly with a command line.

```bash
elasticdl train \
  --image_name=elasticdl:mnist \
  --model_zoo=model_zoo \
  --model_def=mnist.mnist_functional_api.custom_model \
  --training_data=/data/mnist/train \
  --job_name=test-mnist \
  --volume="host_path=/data,mount_path=/data"
```

## Quick Start

Please check out our [step-by-step tutorial](docs/tutorials/get_started.md) for
running ElasticDL on local laptop, on-prem cluster, or on public cloud such as
Google Kubernetes Engine.

[TensorFlow Estimator on MiniKube](docs/tutorials/elasticdl_estimator.md)

[TensorFlow Keras on MiniKube](docs/tutorials/elasticdl_local.md)

[PyTorch on MiniKube](docs/tutorials/elasticdl_torch.md )

## Background

TensorFlow/PyTorch has its native distributed computing feature that is
fault-recoverable. In the case that some processes fail, the distributed
computing job would fail; however, we can restart the job and recover its status
from the most recent checkpoint files.

ElasticDL supports fault-tolerance during distributed training.
In the case that some processes fail, the job would
go on running. Therefore, ElasticDL doesn't need to save checkpoint nor recover
from checkpoints.

The feature of fault-tolerance makes ElasticDL works with the priority-based
preemption of Kubernetes to achieve elastic scheduling.  When Kubernetes kills
some processes of a job to free resource for new-coming jobs with higher
priority, the current job doesn't fail but continues with less resource.

Elastic scheduling could significantly improve the overall utilization of a
cluster. Suppose that a cluster has N GPUs, and a job is using one of
them. Without elastic scheduling, a new job claiming N GPUs would have to wait
for the first job to complete before starting. This pending time could be hours,
days, or even weeks. During this very long time, the utilization of the cluster
is 1/N. With elastic scheduling, the new job could start running immediately
with N-1 GPUs, and Kubernetes might increase its GPU consumption by 1 after the
first job completes.  In this case, the overall utilization is 100%.

The feature of elastic scheduling of ElasticDL comes from its Kubernetes-native
design -- it doesn't rely on Kubernetes extensions like Kubeflow to run
TensorFlow/PyTorch programs; instead, the master process of an ElasticDL job calls
Kubernetes API to start workers and parameter servers; it also watches events
like process/pod killing and reacts to such events to realize fault-tolerance.

In short, ElasticDL enhances TensorFlow/PyTorch with fault-tolerance and elastic
scheduling in the case that you have a Kubernetes cluster. We provide a tutorial
showing how to set up a Kubernetes cluster on Google Cloud and run ElasticDL
jobs there.  We respect TensorFlow's native distributed computing feature, which
doesn't require specific computing platforms like Kubernetes and allows
TensorFlow running on any platform.

## Development Guide

Please refer to [this document](elasticdl/README.md) for development guide.
