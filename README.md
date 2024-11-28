# schwifty

Fully customizable Kubernetes web view.

You can define different view depending on the group of the connected user (available with SSO feature).

## Getting Started

Schwifty is running entirely in your browser. All your data stay in browser or in your Kubernetes clusters.

For Schwity to work, CORS must be enable on Kubernetes API.

### Enable CORS to access Kubernetes API

#### Option 1: setup Kubeapi server

```
- kube-apiserver
  - --cors-allowed-origins=https://kube.pewty.fr
```

#### Option 2: setup HAProxy as Kubeapi server proxy

Install HAProxy pod in your cluster to proxy Kubernetes api and add the following in your backend config:

```
# Add headers
http-response set-header Access-Control-Allow-Origin "https://kube.pewty.fr"
http-response set-header Access-Control-Allow-Headers "*"
http-response set-header Access-Control-Max-Age 600
http-response set-header Access-Control-Allow-Methods "GET, OPTIONS, POST, PUT, PATCH, DELETE"

# Handle option request for CORS
acl is_option method OPTIONS
http-request set-method GET if is_option
http-request set-path /readyz if is_option
```

#### Option 3: install Helm chart

Helm chart can provide HAProxy already configured, with all needed CRD for Schwifty customization.

WARNING: avoid exposing your Kubernetes API publicly through ingress. Please use VPN or port-forwarding. Schwifty is running entirely in your browser, and can access your Kubernetes clusters through your VPN.

```
helm upgrade --install schwifty chart/ -f values.yaml
```

## Values

### Customizations

Define what customization parameters for which user's group:

| Parameter                         | Type   | Description                                                   |
|-----------------------------------|--------|---------------------------------------------------------------|
| `customizations.*`                | string | Name of the customization block                               |
| `customizations.*.groupSelector`  | string | Regex of SSO user's group to affect this customization        |
| `customizations.*.action`         | string | Name of custom actions block                                  |
| `customizations.*.navigation`     | string | Name of custom navigations block                              |
| `customizations.*.list`           | string | Name of custom lists block                                    |
| `customizations.*.edit`           | string | Name of custom edits block                                    |
| `customizations.*.link`           | string | Name of custom links block                                    |

### Actions

Define what your user can do on each resource:

| Parameter             | Type   | Description                                              |
|-----------------------|--------|----------------------------------------------------------|
| `actions.*`           | string | Name of the action block                                 |
| `actions.*.*`         | string | Name of the action                                       |
| `actions.*.*.include` | list   | Enable an action for a list of Kubernetes resources      |
| `actions.*.*.exclude` | list   | Disable an action for a list of Kubernetes resources     |

#### Available actions

| Action  | Description                                       |
|---------|---------------------------------------------------|
| get     | Display details of Kubernetes resource            |
| update  | Allow edition of Kubernetes resource              |
| delete  | Allow deletion of Kubernetes resource             |
| create  | Allow creation of Kubernetes resource             |
| cordon  | Cordon a node                                     |
| drain   | Drain a node                                      |
| exec    | Start a terminal on pod                           |
| logs    | Display logs of pod                               |
| sync    | Add an annotation to trigger update of resource   |
| pause   | Pause a FluxCD resource                           |
| resume  | Resume a FluxCD resource                          |

### Navigations

Define what resource are accessible from navigation:

| Parameter                     | Type   | Description                                                                                               |
|-------------------------------|--------|-----------------------------------------------------------------------------------------------------------|
| `navigations.*`               | string | Name of the navigation block                                                                              |
| `navigations.*.items`         | list   | First level of nav                                                                                        |
| `navigations.*.items.*.label` | string | Nav label                                                                                                 |
| `navigations.*.items.*.icon`  | string | Nav icon (can be svg or png or [int](https://api.flutter.dev/flutter/material/Icons-class.html#constants))|
| `navigations.*.items.*.route` | string | Kubernetes resource name like `namespaces` or `apps/deployments`                                          |
| `navigations.*.items.*.items` | list   | Second level of nav                                                                                       |
| `navigations.*.other.enabled` | bool   | Special nav for all unhandled Kubernetes resources                                                        |
| `navigations.*.other.label`   | string | Label for this special nav                                                                                |

### Lists

Defines the columns to display for each Kubernetes resource. If undefined, use default Kubernetes columns:

| Parameter               | Type   | Description                                                       |
|-------------------------|--------|-------------------------------------------------------------------|
| `lists.*`               | string | Name of the list block                                            |
| `lists.*.*`             | string | Kubernetes resource name like `namespaces` or `apps/deployments`  |
| `lists.*.*.*.label`     | string | Label of column                                                   |
| `lists.*.*.*.key`       | string | JSON Path selection of value                                      |
| `lists.*.*.*.type`      | string | Unused                                                            |
| `lists.*.*.width`       | int    | Width allocated in table                                          |
| `lists.*.*.selectable`  | bool   | If text is selectable                                             |

### Edits

Define what fields are displayed on view/edition page:

| Parameter               | Type   | Description                                                       |
|-------------------------|--------|-------------------------------------------------------------------|
| `edits.*`               | string | Name of the edit block                                            |
| `edits.*.*`             | string | Kubernetes resource name like `namespaces` or `apps/deployments`  |
| `edits.*.*.*.label`     | string | Label of column                                                   |
| `edits.*.*.*.key`       | string | JSON Path selection of value                                      |
| `edits.*.*.*.type`      | string | `block` or `fields` or `base64fields`                             |

#### Edit types

| Type          | Description                                                 |
|---------------|-------------------------------------------------------------|
| block         | Default type. Display all content as block. Can be a map.   |
| fields        | More compact. Can only be a string.                         |
| base64fields  | Allow base64 decoding.                                      |
