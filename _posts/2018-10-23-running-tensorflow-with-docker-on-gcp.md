---
layout: "post"
title: "Running Tensorflow with Docker on GCP"
date: "2018-10-23 00:00:00"
---

Provision Virtual Machine:

```
$ gcloud auth login
 
$ gcloud config set project machine-learning-000000 # Your project id
 
$ gcloud beta compute \
  addresses create mlvm \
  --region=us-east1 \
  --network-tier=PREMIUM
 
$ MLVM_IP="$(gcloud beta compute \
  addresses describe mlvm \
  --region=us-east1 \
 | head -n1 | awk '{print $2}')"
 
$ gcloud beta compute \
  instances create mlvm \
  --zone=us-east1-b \
  --machine-type=n1-standard-2 \
  --subnet=default \
  --network-tier=PREMIUM \
  --address="$MLVM_IP" \
  --maintenance-policy=TERMINATE \
  --no-service-account \
  --no-scopes \
  --accelerator=type=nvidia-tesla-p100,count=1 \
  --image=centos-7-v20181011 \
  --image-project=centos-cloud \
  --boot-disk-size=40GB \
  --boot-disk-type=pd-standard \
  --boot-disk-device-name=mlvm
```

Configure Virtual Machine:

```
$ gcloud beta compute ssh user@mlvm
 
$ sudo su
 
$ cd ~/
 
$ curl https://download.docker.com/linux/centos/docker-ce.repo \
  > /etc/yum.repos.d/docker-ce.repo
 
$ curl https://nvidia.github.io/nvidia-docker/centos7/nvidia-docker.repo \
  > /etc/yum.repos.d/nvidia-docker.repo
 
$ yum install --assumeyes \
  "@Development Tools" \
  "kernel-devel-$(uname -r)" \
  "kernel-headers-$(uname -r)" \
  "docker-ce-18.06.1" \
  "nvidia-docker2-2.0.3"
 
$ curl https://us.download.nvidia.com/tesla/396.44/NVIDIA-Linux-x86_64-396.44.run \
  > NVIDIA-Linux-x86_64-396.44.run
 
$ sh NVIDIA-Linux-x86_64-396.44.run --silent
 
$ systemctl enable docker
 
$ systemctl start docker
 
$ docker run \
  --runtime=nvidia \
  -it \
  --rm \
  tensorflow/tensorflow:1.11.0-devel-gpu \
  python -c "import tensorflow as tf; print(tf.contrib.eager.num_gpus())"
```

😄🙌🎉 … 🔥💰

Destroy Virtual Machine:

```
$ exit # Exit from 'sudo su'
 
$ exit # Exit from 'gcloud beta compute ssh user@mlvm'
 
$ gcloud beta compute \
  addresses delete mlvm \
  --region=us-east1
 
$ gcloud beta compute \
  instances delete mlvm \
  --zone=us-east1-b
```
