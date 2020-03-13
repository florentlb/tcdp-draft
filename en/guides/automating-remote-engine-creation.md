---
categories: orchestration, deployment
related_apis: runtime/engines, runtime/clusters
version: $version
keywords: remote engine, remote engine clusters, talend management console, talend cloud
description: In this guide, learn how to automate your Talend Remote Engine deployment and pairing, as well as managing a cluster of Remote Engines.
---

# Automating Remote Engine creation

Manage engine clusters, create Remote Engines and make them ready to run by pairing them quickly.

## Generating a Personal Access Token

Use Personal Access Tokens to authenticate to Talend Cloud APIs or to connect Talend Studio to Talend Cloud. You generate these access tokens from Talend Cloud in the form of JSON Web Tokens (JWT).

Access Tokens are personal and the permissions they grant are synchronized with the user's permissions, at any time. When you generate a token, it is only displayed once.

You can have up to five active tokens at the same time. It is a best practice to keep as few active tokens as possible. Delete unused tokens and replace old tokens to increase the system security.

> **Important**: Access Tokens are the safest and recommended authentication mode to use.

1. Log in to the Talend Cloud portal or an application.
In the top right corner, click the user menu then Profile preferences.
2. Go to the **PERSONAL ACCESS TOKENS** page. This page allows you to manage all your tokens.
3. Select **ADD TOKEN**.
A new page opens. If you have the *Roles - Manage* permission, you can see all the permissions associated to the token.
4. Enter the Token name.
5. Click **GENERATE**.
A Bearer Token is generated and opens in a new window that is displayed only once. If you close it without copying the token, you will not be able to use this token.
6. Copy the token and paste it where you need.

From the **PERSONAL ACCESS TOKENS** page, you can check each token creation date and last use.

If you use access tokens to call Talend Cloud APIs, make sure you include the Bearer type in front of the token value. For example:

```
curl 
    -X GET 
    -H "Authorization: Bearer <your_token> 
    "https://api.cloud.talend.com/<endpoint>"
```
The permissions of the generated token correspond to your current permissions. If your permissions change after generating the token, the token permissions change accordingly.

After generating a token, you can only edit its name or delete it. Security Administrators can also see and delete your tokens. Refer to [Managing personal access tokens for all users](http://www.help.talend.com).

## Creating and pairing a Remote Engine programmatically
Create and pair a Remote Engine programmatically through an API to automate your flow.

Before proceeding, make sure you have downloaded the archive file of the Remote Engine and unzipped it locally on the machine that will execute the following API requests. The Remote Engine is not paired yet.

The code samples in the following example show the different parts of a shell script.

1. From your profile preferences in Talend Cloud, generate a Personal Access Token. This token allows to sign the API calls.
2. Create a Remote Engine with a POST on the [runtimes/remote-engines endpoint](../api-refs/orchestration/runtimes/remote-engines).
A few parameters are defined in the payload, some of which are optional.
    ```
    curl -X POST 
    --header 'Content-Type: application/json' 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer 'VtBmuAoYSse6FW4zh8JqlnQsl73P8xAB4j9-qD98qx8HI48DDJEUKCBKncU5FSGi' -d '{
        "name": "My Remote Engine",
            "environmentId": "5d1619bb818cfe3dca795e41",  
            "workspaceId": "5d1619bb818cfe3dca795e44" 
        }' 
    'https://api.us.cloud.talend.com/tmc/v1.3/runtimes/remote-engines'
    ```
3. Retrieve and insert the pairing key of the new Remote Engine into the preauthorized.key.cfg file of the Remote Engine installation directory.
    ```
    echo "remote.engine.pre.authorized.key = $PAIRING_KEY" > <RemoteEngineInstallationDirectory>/etc/preauthorized.key.cfg 
    ```
    > **Tip**: You can retrieve the pairing key from the preAuthorizedKey parameter of the Remote Engine creation response body.
4. Update the URL of the pairing service to match the region of your deployment in the `org.talend.ipaas.rt.pairing.client.cfg` file of the Remote Engine installation directory.
5. Adapt the following example to your Talend Cloud region.
    ```
    echo "pairing.service.url=https://pair.us.cloud.talend.com" > <RemoteEngineInstallationDirectory>/etc/org.talend.ipaas.rt.pairing.client.cfg
    ```
Your new Remote Engine will be paired when started. 

The next example shows how to add the created Remote Engine to an existing Remote Engine Cluster.

## Adding an existing Remote Engine to a Remote Engine Cluster
Enrich the shell script created in the previous example to add the Remote Engine to an existing Remote Engine Cluster.

At this point, make sure that:
* You have a Remote Engine installed and already paired.
* You already have a Remote Engine Cluster.

1. If you do not have one, generate a Personal Access Token from your profile preferences in Talend Cloud. This token can be used to sign your API calls.
2. Add the Remote Engine to the Remote Engine Cluster of your choice with a PUT on the [runtimes/remote-engine-clusters endpoint](../api-refs/orchestration/runtimes/remote-engine-clusters) as follows.
    ```
    curl -X PUT 
    --header 'Content-Type: application/json' 
    --header 'Accept: application/json' 
    --header 'Authorization: Bearer 'VtBmuAoYSse6FW4zh8JqlnQsl73P8xAB4j9-qD98qx8HI48DDJEUKCBKncU5FSGi'' 
    'https://api.us.cloud.talend.com/tmc/v1.3/runtimes/remote-engine-clusters/5dc02e89f87cb233a2f83990/engines/5d2f1d9018e83f4406199233'
    ```
    >**Note**: You can retrieve Remote Engine and Remote Engine Cluster IDs with a GET on the /tmc/v1.3/runtimes/remote-engines or /tmc/v1.3/runtimes/remote-engine-clusters endpoints. You can also find their ID from the URL in Talend Cloud when displaying a Remote Engine or Remote Engine Cluster details.

The Remote Engine is added to the Remote Engine Cluster.
