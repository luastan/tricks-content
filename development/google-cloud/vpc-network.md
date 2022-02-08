---
title: VPC network
description: Manage your cloud network infraestructure
---



## External IP addresses



```shell
gcloud compute addresses create luastans-lb \
    --project=luastans-tricks\
    --description="Luastan's load balancer"\
    --region=europe-west1
```
