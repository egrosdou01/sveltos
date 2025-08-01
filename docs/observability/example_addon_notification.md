---
title: Notifications - Projectsveltos
description: Sveltos is an application designed to manage hundreds of clusters by providing declarative APIs to deploy Kubernetes add-ons across multiple clusters.
tags:
    - Kubernetes
    - add-ons
    - helm
    - clusterapi
    - multi-tenancy
    - Sveltos
    - Slack
    - Teams
authors:
    - Gianluca Mardente
    - Magnus R. Christoffersen
---

In these examples, we will configure Sveltos to send a notification whenever all the add-ons and applications listed in the `ClusterProfile` are deployed to the clusters matching the cluster label selector set to **env=fv**.

Once the defined conditions are met, a notification will be generated and send out.

## Slack

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: slack
        type: Slack
        notificationRef:
          apiVersion: v1
          kind: Secret
          name: slack
          namespace: default
    ```

The above `slack` secret contains the Slack channel and the token.

  ```bash
  $ kubectl create secret generic slack \
    --from-literal=SLACK_CHANNEL_ID=<your channel id> \
    --from-literal=SLACK_TOKEN=<your token> \
    --type=addons.projectsveltos.io/cluster-profile
  ```

## Webex

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: webex
        type: Webex
        notificationRef:
          apiVersion: v1
          kind: Secret
          name: webex
          namespace: default
    ```

The above `Webex` secret contains the Webex room id and the token.

  ```bash
  $ kubectl create secret generic webex \
    --from-literal=WEBEX_ROOM_ID=<your channel id> \
    --from-literal=WEBEX_TOKEN=<your token> \
    --type=addons.projectsveltos.io/cluster-profile
  ```

## Teams

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: teams
        type: Teams
        notificationRef:
          apiVersion: v1
          kind: Secret
          name: teams
          namespace: default
    ```

The above `Teams` secret contains the Teams webhook URL.

  ```bash
  $ kubectl create secret generic teams \
    --from-literal=TEAMS_WEBHOOK_URL="<your URL>" \
    --type=addons.projectsveltos.io/cluster-profile
  ```

## Discord

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: discord
        type: Discord
        notificationRef:
          apiVersion: v1
          kind: Secret
          name: discord
          namespace: default
    ```

The above `discord` secret contains the Discord channel id and the token.

  ```bash
  $ kubectl create secret generic discord \
    --from-literal=DISCORD_CHANNEL_ID=<your channel id> \
    --from-literal=DISCORD_TOKEN=<your token> \
    --type=addons.projectsveltos.io/cluster-profile
  ```

## Telegram

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: telegram
        type: Telegram
        notificationRef:
          apiVersion: v1
          kind: Secret
          name: telegram
          namespace: default
    ```

The above `telegram` secret contains the Telegram chat id and the token.

  ```bash
  $ kubectl create secret generic telegram \
    --from-literal=TELEGRAM_CHAT_ID=<your int64 chat id> \
    --from-literal=TELEGRAM_TOKEN=<your token> \
    --type=addons.projectsveltos.io/cluster-profile
  ```

## SMTP

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: smtp
        type: SMTP
        notificationRef:
          apiVersion: v1
          kind: Secret
          name: smtp
          namespace: default
    ```

The above `smtp` secret contains the SMTP info.

  ```bash
  $ kubectl create secret generic smtp \
    --from-literal=SMTP_RECIPIENTS=<to email addresses> \
    --from-literal=SMTP_BCC=<optional, cc email addresses> \
    --from-literal=SMTP_BCC=<optional, bcc email addresses> \
    --from-literal=SMTP_SENDER=<send email address> \
    --from-literal=SMTP_PASSWORD=<sender app passowrd> \
    --from-literal=SMTP_HOST=<host> \
    --from-literal=SMTP_PORT=<OPTIONAL, SMTP SERVER PORT, DEFAULTS TO "587">
    --type=addons.projectsveltos.io/cluster-profile \
  ```

## Kubernetes event

!!! example ""
    ```yaml
    ---
    apiVersion: lib.projectsveltos.io/v1beta1
    kind: ClusterHealthCheck
    metadata:
      name: production
    spec:
      clusterSelector:
        matchLabels:
          env: fv
      livenessChecks:
      - name: addons
        type: Addons
      notifications:
      - name: kubernetes-events
        type: KubernetesEvent
    ```

To list the events generated by Sveltos, use the below command.

```bash
$ kubectl get events -n default --field-selector reason=ClusterHealthCheck
LAST SEEN   TYPE      REASON               OBJECT                  MESSAGE
31s         Normal    ClusterHealthCheck   clusterhealthcheck/hc   cluster Capi:default/sveltos-management-workload...
16s         Warning   ClusterHealthCheck   clusterhealthcheck/hc   cluster Capi:default/sveltos-management-workload...
```

!!! tip
    The Event type will be set to: `type: Normal` when the add-ons are deployed.


The Event message contains the below information on the cluster:
  1. Cluster `type: Capi or Sveltos`
  1. Cluster `namespace`
  1. Cluster `name`
