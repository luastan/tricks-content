---
title: Cloud Run
description: Cloud Run is a cheap pay-for use option when your application doesn't have huge amounts of traffic (otherwise, it's not that cheap)
badge: Google Cloud
---


## Basic deployment


Build an image and tag it like this:

```shell
docker build -t "{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}" .
```

Then you need to push it to your project's image repository:

```shell
docker push "{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}"
```

Finally you can deploy your image with the `gcloud run deploy` command:

```shell
gcloud run deploy "{{ container-name cool-container }}" \
    --image "{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}"\
    --region '{{ region europe-west1 }}'\
    --platform managed \
    --allow-unauthenticated
```

You can redeploy images with the same container name. Old instances would remain stopped but linked to your project as revisions. Have this in mind as the storage of those revisions is billable.

### On Cloudbuild

Same would apply on cloudbuild where your `cloudbuild.yaml` could look somewhat like this:
```yaml[cloudbuild.yaml]
steps:

  # Container image build
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', '{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}', '.']

  # Push the container image to Container Registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', '{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}']

  # Deploy container image to Cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args: [
      'run', 'deploy', '{{ container-name cool-container }}',
      '--image', '{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}',
      '--region', 'europe-west1',
      '--platform', 'managed',
      '--allow-unauthenticated'
    ]

timeout: 3000s

images:
  - "{{ cloudrun-image gcr.io/$PROJECT_ID/cool-image }}"
```

With this, the build can be manually triggered with `gcloud builds`:

```shell
gcloud builds submit .
```
