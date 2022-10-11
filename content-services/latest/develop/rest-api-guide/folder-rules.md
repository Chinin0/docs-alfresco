---
title: Managing Folder Rules
---

This section walks through how to manage Alfresco [folder rules]({% link content-services/latest/using/content/rules.md %}) 
via the ReST API.

The ReST API has a full set of calls to do most things around rules and rule sets.

## Common error messages
These endpoints work only with folder nodes, if you use other node types, then the following error is returned:

```json
{
  "error": {
     "errorKey": "NodeId of a folder is expected!",
     "statusCode": 400,
     "briefSummary": "09060008 NodeId of a folder is expected!",
     "stackTrace": "For security reasons the stack trace is no longer displayed, but the property is kept for previous versions",
     "descriptionURL": "https://api-explorer.alfresco.com"
  }
}
```

If you use an incorrect folder node id, then the following error is returned:

```json
{
  "error": {
    "errorKey": "framework.exception.EntityNotFound",
    "statusCode": 404,
    "briefSummary": "09110003 The entity with id: 1cfa1e0b-ee26-44c1-b092-8c04d684a16d was not found",
    "stackTrace": "For security reasons the stack trace is no longer displayed, but the property is kept for previous versions",
    "descriptionURL": "https://api-explorer.alfresco.com"
  }
}
```

## List rules {#list-rules}
Listing the rules applied to a folder in the repository is done with the following endpoint.

**API Explorer URL**: [http://localhost:8080/api-explorer/#/rules/listRules](http://localhost:8080/api-explorer/#/rules/listRules){:target="_blank"}

Listing rules can be done with the following HTTP GET call:

`http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{id}/rule-sets/{ruleSetId}/rules?skipCount={skipCount}&maxItems={maxItems}`

The `{id}` part can be any of the constants `-root-`, `-my-`, `-shared-` or an Alfresco Node Identifier
(e.g. d8f561cc-e208-4c63-a316-1ea3d3a4e10e) for the folder that have rules applied. The `ruleSetId` parameter is the 
identifier for a rule set. The alias `-default-` can be used to refer to the only rule set on a folder. There's currently 
no support for multiple rule sets linked to a single folder, and the behaviour of `-default-` is not defined in this case. 
The `skipCount` and `maxItems` parameters are explained [here]({% link content-services/latest/develop/oop-ext-points/rest-api-java-wrapper.md %}#common-parameters) 

In the following example we are listing the rules of a folder with the Alfresco Node ID `1cfa1e0b-ee26-44c1-b092-8c04d684a16d`:

```bash
$ curl -X GET -H 'Accept: application/json' -H 'Authorization: Basic VElDS0VUXzA4ZWI3ZTJlMmMxNzk2NGNhNTFmMGYzMzE4NmNjMmZjOWQ1NmQ1OTM=' 'http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/1cfa1e0b-ee26-44c1-b092-8c04d684a16d/rule-sets/-default-/rules?skipCount=0&maxItems=100' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   381    0   381    0     0   1545      0 --:--:-- --:--:-- --:--:--  1607
{
  "list": {
    "pagination": {
      "count": 2,
      "hasMoreItems": false,
      "totalItems": 2,
      "skipCount": 0,
      "maxItems": 100
    },
    "entries": [
      {
        "entry": {
          "isEnabled": true,
          "name": "Apply Summarizable Aspect",
          "id": "28a12c53-60bc-4834-959f-ee3bf97db916",
          "isInheritable": false,
          "triggers": [
            "inbound"
          ],
          "actions": [
            {
              "actionDefinitionId": "add-features",
              "params": {
                "aspect-name": "cm:summarizable"
              }
            }
          ],
          "isAsynchronous": false
        }
      },
      {
        "entry": {
          "isEnabled": true,
          "name": "Move MS Word files to Shared folder",
          "id": "e0e8afde-08ad-4e53-98b2-035788d7fc26",
          "isInheritable": true,
          "triggers": [
            "inbound"
          ],
          "conditions": {
            "inverted": false,
            "booleanMode": "and",
            "compositeConditions": [
              {
                "inverted": false,
                "booleanMode": "and",
                "simpleConditions": [
                  {
                    "field": "mimetype",
                    "comparator": "equals",
                    "parameter": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
                  }
                ]
              }
            ]
          },
          "actions": [
            {
              "actionDefinitionId": "move",
              "params": {
                "destination-folder": "37f97bbd-62c7-447b-b1c8-f987d94c7795"
              }
            }
          ],
          "isAsynchronous": false
        }
      }
    ]
  }
}
```

The above response shows us that there are two rules applied to this folder. 

The first rule is called *Apply Summarizable Aspect*, and it triggers when something is created or uploaded to this folder 
(i.e. `"triggers": ["inbound"`). When the rule is triggered it executes the repository action with the `add-features` id. The result is the 
`cm:summarizable` aspect being applied to the file.

The second rule is called *Move MS Word files to Shared folder*, and it triggers when something is created or uploaded to 
this folder (i.e. `inbound`). It will also trigger if something is created or uploaded to a sub-folder 
(i.e. `"isInheritable": true`). A simple condition has been applied to the trigger, and it will only trigger when a 
MS Word 2007 document is uploaded. When the rule is triggered it executes the repository action with the `move` id. The 
result is that the uploaded MS Word document is moved to the destination folder with the Node Id the 
"37f97bbd-62c7-447b-b1c8-f987d94c7795".

## Create a rule {#createrule}
Creating a folder rule involves defining the following things:

* The event that triggers the rule
* The conditions the content has to meet
* The action performed on the content

The following endpoint can be used for this:

**API Explorer URL:** [http://localhost:8080/api-explorer/#/rules/createRule](http://localhost:8080/api-explorer/#/rules/createRule){:target="_blank"}

Creating a rule is done with the following POST call:

`http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{id}/rule-sets/{ruleSetId}/rules`

The `{id}` part can be any of the constants `-root-`, `-my-`, `-shared-` or an Alfresco Node Identifier
(e.g. d8f561cc-e208-4c63-a316-1ea3d3a4e10e) for the folder that should have the rule applied. The `ruleSetId` parameter is the
identifier for a rule set. The alias `-default-` can be used to refer to the only rule set on a folder. There's currently
no support for multiple rule sets linked to a single folder, and the behaviour of `-default-` is not defined in this case.

The POST data body for a rule create call looks like this:

```json
{
  "name": "{the name of the rule}",
  "description": "{optional description of the rule}",
  "isEnabled": true,      
  "isInheritable": false, 
  "isAsynchronous": false,
  "errorScript": "string", 
  "triggers": [
    "inbound"
  ],
  "conditions": {
    "inverted": false,
    "booleanMode": "and",
    "compositeConditions": [],
    "simpleConditions": []
  },
  "actions": [
    {
      "actionDefinitionId": "{repoActionId}",
      "params": {}
    }
  ]
}
```

Property explanations:

* `isEnabled`: Set this to `true` if the rule should be active immediately, this is the default.
* `isInheritable`: Set this to `true` if the rule should be inherited by sub-folders, default is `false`
* `isAsynchronous`: Set this to `true` if the rule should be executed in a separate transaction from the content, default is `false`. By default, your rule could interrupt content creation, updates etc if the rule action code does not work properly, as it is executed in the same transaction.
* `errorScript`: name of a JavaScript file that should be run in case of an error in the rule execution.
* `triggers`: possible values are `inbound`, `update`, and `outbound`. They can be combined.
* `conditions`: an array of conditions that must be met for this rule to trigger. 
  * `compositeConditions`: an array of `simpleConditions`
  * `simpleConditions`: consists of an array of:
    * `field`: the content model property that should be part of the condition, for example `mimetype`.
    * `comparator`: the operator to be used for the condition, for example `equals`.
    * `parameter`: the value that the `field` should be compared to. Make sure to specify it with correct data type.
* `actions`: an array of actions, and their parameters, to be executed when the rule triggers:
  * `actionDefinitionId`: the repository action identifier, for example `move`.
  * `params`: object with parameters that should be passed to the action, for example `destination-folder`

We are now going to look at an example rule called **Apply the Effective aspect to PDFs**. This rule will apply the 
`cm:effective` aspect when a PDF file with `size > 1KB` is uploaded or updated in a folder.

Because there are quite a few parameters that you need to get right in the POST body, it is a good idea to first 
create the rule in Alfresco Share and test it, so it works as expected. Then after that use the [list call](#list-rules) to 
figure out what each parameter should look like.

Here is how the rule looks like when created in Alfresco Share:

![rule-definition-share]({% link content-services/images/apply-aspect-rule-definition-share.png %})

Making the list call shows the rule definition as follows:

```json
{
  "entry": {
    "isEnabled": true,
    "name": "Apply the Effective aspect to PDFs",
    "description": "When a PDF with size>1KB is uploaded or updated apply the Effective aspect (cm:effective)",
    "id": "be0c68e9-ca31-447b-aee4-4da3a1d35543",
    "isInheritable": true,
    "triggers": [
      "inbound",
      "update"
    ],
    "conditions": {
      "inverted": false,
      "booleanMode": "and",
      "compositeConditions": [
        {
          "inverted": false,
          "booleanMode": "and",
          "simpleConditions": [
            {
              "field": "mimetype",
              "comparator": "equals",
              "parameter": "application/pdf"
            },
            {
              "field": "size",
              "comparator": "greater_than",
              "parameter": "1000"
            }
          ]
        }
      ]
    },
    "actions": [
      {
        "actionDefinitionId": "add-features",
        "params": {
          "aspect-name": "cm:effectivity"
        }
      }
    ],
    "isAsynchronous": false
  }
}
```
We can see that the list call JSON response is very similar to what we need for the POST data. Remove the `entry` 
encapsulation and the `id` property, and you end up with what you need: 

```json
{
  "isEnabled": true,
  "name": "Apply the Effective aspect to PDFs",
  "description": "When a PDF with size>1KB is uploaded or updated apply the Effective aspect (cm:effective)",
  "isInheritable": true,
  "triggers": [
    "inbound",
    "update"
  ],
  "conditions": {
    "inverted": false,
    "booleanMode": "and",
    "compositeConditions": [
      {
        "inverted": false,
        "booleanMode": "and",
        "simpleConditions": [
          {
            "field": "mimetype",
            "comparator": "equals",
            "parameter": "application/pdf"
          },
          {
            "field": "size",
            "comparator": "greater_than",
            "parameter": "1000"
          }
        ]
      }
    ]
  },
  "actions": [
    {
      "actionDefinitionId": "add-features",
      "params": {
        "aspect-name": "cm:effectivity",
        "cm:from": "2022-09-10T08:00:00.000+0000",
        "cm:to": "2022-09-12T21:00:00.000+0000"
      }
    }
  ],
  "isAsynchronous": false
}
```

When you define the rule in Alfresco Share it is not possible to set the properties contained in the `cm:effective` aspect.
So they are not visible in the list call response. They (i.e. `cm:from` and `cm:to`) have been added in the above POST data.

Save the above POST data JSON in a file called something like `create-rule.json`. We will POST this file with the following 
call to create the rule.

Here is how the call looks like, assuming that we have stored the query JSON data in a file called `create-rule.json` 
(it does not work to write the query with the `-d` curl parameter on the command line), and the folder we are creating
the rule in have the Alfresco Node ID `1cfa1e0b-ee26-44c1-b092-8c04d684a16d`:

```bash
curl -X POST -H 'Content-Type: application/json' -H 'Accept: application/json' --header 'Authorization: Basic VElDS0VUXzIxYzAzOWMxNjFjYzljMDNmNmNlMzAwYzAyMDY5YTQ2OTQwZmYzZmM=' --data-binary '@create-rule.json' 'http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/1cfa1e0b-ee26-44c1-b092-8c04d684a16d/rule-sets/-default-/rules' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1752    0   743  100  1009   1325   1799 --:--:-- --:--:-- --:--:--  3173
{
  "entry": {
    "isEnabled": true,
    "name": "Apply the Effective aspect to PDFs",
    "description": "When a PDF with size>1KB is uploaded or updated apply the Effective aspect (cm:effective)",
    "id": "799d25bf-a41d-401a-8e9b-ac5b6c256dfd",
    "isInheritable": true,
    "triggers": [
      "inbound",
      "update"
    ],
    "conditions": {
      "inverted": false,
      "booleanMode": "and",
      "compositeConditions": [
        {
          "inverted": false,
          "booleanMode": "and",
          "simpleConditions": [
            {
              "field": "mimetype",
              "comparator": "equals",
              "parameter": "application/pdf"
            },
            {
              "field": "size",
              "comparator": "greater_than",
              "parameter": "1000"
            }
          ]
        }
      ]
    },
    "actions": [
      {
        "actionDefinitionId": "add-features",
        "params": {
          "cm:from": "2022-09-10T08:00:00.000+0000",
          "aspect-name": "cm:effectivity",
          "cm:to": "2022-09-12T21:00:00.000+0000"
        }
      }
    ],
    "isAsynchronous": false
  }
}
```

The response contains the `id` for the new rule definition (i.e. "799d25bf-a41d-401a-8e9b-ac5b6c256dfd"). This id can be 
used later on to get the rule definition.

## Update a rule
Updating the metadata for an Alfresco folder rule.

**API Explorer URL:** [http://localhost:8080/api-explorer/#/rules/updateRule](http://localhost:8080/api-explorer/#/rules/updateRule){:target="_blank"}

**See also:** [How to create a rule](#createrule)

It’s also possible to update the site metadata after the site has been created. We can for example change the 
description and title, and it is even possible to change the visibility of the site. Use the following PUT call:

`http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/sites/{id}`

The identifier for the site to be updated is specified with the `{id}` parameter.

The body for a site update call looks like this:

```json
{
  "title": "{the updated name of the site}",
  "description": "{the updated description of the site}",
  "visibility": "[PUBLIC|PRIVATE|MODERATED]"
}
```

To update a site with the `my-stuff` `id` make the following call:

```bash
$ curl -X PUT -H 'Content-Type: application/json' -H 'Accept: application/json' -H 'Authorization: Basic VElDS0VUXzA4ZWI3ZTJlMmMxNzk2NGNhNTFmMGYzMzE4NmNjMmZjOWQ1NmQ1OTM=' -d '{ "title": "My stuff UPDATED", "description": "My stuff Desc UPDATED", "visibility": "PRIVATE"}' 'http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/sites/my-stuff' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   303    0   208  100    95   1004    458 --:--:-- --:--:-- --:--:--  1463
{
  "entry": {
    "role": "SiteManager",
    "visibility": "PRIVATE",
    "guid": "7ef8798f-bbcd-4d92-8db5-4d51edf739f6",
    "description": "My stuff Desc UPDATED",
    "id": "my-stuff",
    "preset": "site-dashboard",
    "title": "My stuff UPDATED"
  }
}
```

The updated site metadata is returned so you can make sure it's correct.

## Get rule definition by id
The rule definition can be fetched using the rule id with the following endpoint.

**API Explorer URL**: [http://localhost:8080/api-explorer/#/rules/getRule](http://localhost:8080/api-explorer/#/rules/getRule){:target="_blank"}

Listing the rule definition based on rule id can be done with the following HTTP GET call:

`http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/{id}/rule-sets/{ruleSet}/rules/{ruleId}`

The `{id}` part can be any of the constants `-root-`, `-my-`, `-shared-` or an Alfresco Node Identifier
(e.g. d8f561cc-e208-4c63-a316-1ea3d3a4e10e) for the folder that contains the rule. The `ruleSetId` parameter is the
identifier for a rule set. The alias `-default-` can be used to refer to the only rule set on a folder. There's currently
no support for multiple rule sets linked to a single folder, and the behaviour of `-default-` is not defined in this case.
The `ruleId` is the identifier of the rule.

In the following example we are getting the rule definintion for a rule with the id `34cbe98b-1755-4161-87f5-6083044e0843` 
contained in a folder with the Alfresco Node ID `1cfa1e0b-ee26-44c1-b092-8c04d684a16d`:

```bash
$ curl -X GET -H 'Accept: application/json' -H 'Authorization: Basic VElDS0VUXzA4ZWI3ZTJlMmMxNzk2NGNhNTFmMGYzMzE4NmNjMmZjOWQ1NmQ1OTM=' 'http://localhost:8080/alfresco/api/-default-/public/alfresco/versions/1/nodes/1cfa1e0b-ee26-44c1-b092-8c04d684a16d/rule-sets/-default-/rules/34cbe98b-1755-4161-87f5-6083044e0843' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   381    0   381    0     0   1545      0 --:--:-- --:--:-- --:--:--  1607
{
  "entry": {
    "isEnabled": true,
    "name": "Apply the Effective aspect to PDFs",
    "description": "When a PDF with size>1KB is uploaded or updated apply the Effective aspect (cm:effective)",
    "id": "34cbe98b-1755-4161-87f5-6083044e0843",
    "isInheritable": true,
    "triggers": [
      "inbound",
      "update"
    ],
    "conditions": {
      "inverted": false,
      "booleanMode": "and",
      "compositeConditions": [
        {
          "inverted": false,
          "booleanMode": "and",
          "simpleConditions": [
            {
              "field": "mimetype",
              "comparator": "equals",
              "parameter": "application/pdf"
            },
            {
              "field": "size",
              "comparator": "greater_than",
              "parameter": "1000"
            }
          ]
        }
      ]
    },
    "actions": [
      {
        "actionDefinitionId": "add-features",
        "params": {
          "cm:from": "2022-09-10T08:00:00.000+0000",
          "aspect-name": "cm:effectivity",
          "cm:to": "2022-09-12T21:00:00.000+0000"
        }
      }
    ],
    "isAsynchronous": false
  }
}
```

The above response shows the definition of the rule. 