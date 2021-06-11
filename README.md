# argo-wf-events
This project shows how to use the new Argo Workflows events feature

# Events

> Events are supporter from the Argo Workflows v2.11 and after

## Overview

The official doc about Workflow Events ca be found here:
[events official doc](https://argoproj.github.io/argo-workflows/events/)

## How to trigger a Workflow with an event

In order to trigger the submission of a Workflow Template by using an event, you have to follow the following steps:

- Create a submittable WorkflowTemplate on Argo
- Create a WorkflowEventBinding that realizes the details of the binding between an event and a Workflow
- Apply the WorkflowEventBinding in the correct Argo namespace
- Trigger the submission of the WorkflowTemplate

It follows a detailed explanation of these steps:

### Create a submittable WorkflowTemplate on Argo

Before the binding between an event and a workflow template, you must create the workflow template that you want to trigger.
The following one takes in input the "message" parameter specified into the API call body, passed through the WorkflowEventBinding parameters section, and finally resolved here as the message of the whalesay image.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: my-wf-tmple
  namespace: argo
spec:
  templates:
    - name: main
      inputs:
        parameters:
          - name: message
            value: "{{workflow.parameters.message}}"
      container:
        image: docker/whalesay:latest
        command: [cowsay]
        args: ["{{inputs.parameters.message}}"]
  entrypoint: main
```

### Create a WorkflowEventBinding

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowEventBinding
metadata:
  name: event-consumer
spec:
  event:
    selector: payload.message != "" && metadata["X-Argo-E2E"] == ["true"] && discriminator == "my-discriminator"
  submit:
    workflowTemplateRef:
      name: my-wf-tmple
    arguments:
      parameters:
      - name: message
        valueFrom:
          event: payload.message
```

### Apply the WorkflowEventBinding

```bash
kubectl apply -f event-template.yml
```

### Trigger the submission of the WorkflowTemplate

```bash
curl $ARGO_SERVER/api/v1/events/argo/my-discriminator \
    -H "Authorization: $ARGO_TOKEN" \
    -H "X-Argo-E2E: true" \
    -d '{"message": "hello argo-events!"}'
```
