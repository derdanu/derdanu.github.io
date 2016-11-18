---
layout: post
title:  "Using ARM REST API with Microsoft Cloud Deutschland"
date:   2016-11-17 15:45:54 +0100
categories: azure
---

There a lot of tutorials how-to use the Azure Resource Manager (ARM) REST API available, 
but at the moment none of them are covering the special endpoints for the Microsoft Cloud Deutschland.

First of all we have create a service principal and grant it access to our subscription. 

Create a service principal 
====================

This tutorial covers the creation with PowerShell commands. 

### Authenticate with Azure

By default the Azure Powershell Commandlets are pointing to our public cloud. To Login with Powershell we have 
to provide the environment name for Micrsoft Cloud Deutschland. If you have trouble with setting the option 
please make sure you have installed the latest [Azure PowerShell Commandlet][azure-powershell-wi]. 

`Add-AzureRmAccount -EnvironmentName AzureGermanCloud`

We will need the TenantId of the Azure Active directory and your Subscription Id  later on.

{% highlight PowerShell %}
Environment           : AzureGermanCloud
Account               : user@name.onmicrosoft.de
TenantId              : f1a3d125-e2bf-48a0-a025-28e47c410299
SubscriptionId        : ae455d71-fec0-4d6d-9edc-a291369b6910
SubscriptionName      : Microsoft Cloud Deutschland Subscription
CurrentStorageAccount :
{% endhighlight %}


### Create an Application

Now as we are authenticated  with the Azure Active directory we have to create an application. **Please use a strong password.**

{% highlight PowerShell %}
$app = New-AzureRmADApplication -DisplayName "SP for Rest Calls" -HomePage "https://www.contoso.org/exampleapp" -IdentifierUris "https://www.contoso.org/exampleapp" -Password "DONT_COPY_AND_PASTE_PASSWORDS"
{% endhighlight %}

Please write down the application ID  we will need it together with the password later on. 

{% highlight PowerShell %}
PS C:\> echo $app.ApplicationId

Guid
----
71edf3e9-5e25-a95c-9c37-b74c3d08bcc9
{% endhighlight %}

### Create Service Principal 
With follwing command we can create our service principal.

{% highlight PowerShell %}
New-AzureRmADServicePrincipal -ApplicationId $app.ApplicationId
{% endhighlight %}

### Grant permission to the subscription

Until now we have a service principal but you are not able to create any ressources within soursubscription. We have to grant 
the application permission to the subscription


{% highlight PowerShell %}
New-AzureRmRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $app.ApplicationId.Guid
{% endhighlight %}

Thats it.

---

Howto RestCall
==================== 

As the REST API uses Oauth2 for authentication we have to get the Bearer token. This token has to be sent as request header with 
each operation.

### Login
To get the Bearer token we have to login. Just send a HTTP POST request to the followin url and provide the needed options in the body of this request.
Please replace the variables to fit your environment. 

POST Url: 
`https://login.microsoftonline.de/<TenantId>/oauth2/token`

BODY: 
{% highlight PowerShell %}
Content-Type=application/x-www-form-urlencoded

grant_type=client_credentials 
client_id=<ApplicationId>
client_secret=<YOUR_PASSWORD>
resource=https://management.microsoftazure.de/ 
{% endhighlight %}

If everything went okay you should get the Bearer token as response of the request. Plese write it down as you need it for all
the further operations. The token an expiration date and you have to renew it in a given period of time. 

{% highlight Json %}
{
  "token_type": "Bearer",
  "expires_in": "3599",
  "ext_expires_in": "0",
  "expires_on": "1479376411",
  "not_before": "1479372511",
  "resource": "https://management.microsoftazure.de/",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Il9mMlJNZVZOYVJjVTZPSE1NbHRwc0VmV0VJMCIsImtpZCI6Il9mMlJNZVZOYVJjVTZPSE1NbHRwc0VmV0VJMCJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQubWljcm9zb2Z0YXp1cmUuZGUvIiwiaXNzIjoiaHR0cHM6Ly9zdHMubWljcm9zb2Z0b25saW5lLmRlL2YxYzliMTI1LWUyYmYtNDhjMC1iMDI1LTIzZTQ3YzQxMDI5My8iLCJpYXQiOjE0NzkzNzI1MTEsIm5iZiI6MTQ3OTM3MjUxMSwiZXhwIjoxNDc5Mzc2NDExLCJhcHBpZCI6IjA0NGI3ZTFjLTI5OTQtNGI1NC05ODQyLWJjOWY4ZGUzMTRiMyIsImFwcGlkYWNyIjoiMSIsImlkcCI6Imh0dHBzOi8vc3RzLm1pY3Jvc29mdG9ubGluZS5kZS9mMWM5YjEyNS1lMmJmLTQ4YzAtYjAyNS0yM2U0N2M0MTAyOTMvIiwib2lkIjoiNjE3NGU3MzEtNWRiOS00NDA2LWIyMGQtM2Y3NzM1OWNlYzNiIiwic3ViIjoiNjE3NGU3MzEtNWRiOS00NDA2LWIyMGQtM2Y3NzM1OWNlYzNiIiwidGlkIjoiZjFjOWIxMjUtZTJiZi00OGMwLWIwMjUtMjNlNDdjNDEwMjkzIiwidmVyIjoiMS4wIn0.AWB2t5VixqbLMgMMTPe1J4mYxUXmPbT5pNfdXmCwTp3-u9Ahoby_yaGos2ObiGH1nQCiR0I6bBknC7-XUK3CY92UUrHYIr7qoAMnRAIvCdKxQlKP5AmUHFN3omkkwVvyUHcAdXFInzVIbxD-PeFvaaNPxPSdEfvyBf9b4GJ_l3qRZeQh_K2d6ez0cO7XoVN6-fxKWTC7O66qT5SNkZ6jdnbT3SwUpaK7FudOjzaGlhYVI6nZZS8GlJuxW6RuTfs-oly3egkqG4p2MLPginA7a7Vivkz4jA6EZ6nr4TnO4o9Oe_jcp-chFxLUsc6iCElazmG5n2hxZAEhjFxGYKzD3w"
}
{% endhighlight %}

### List RG
For example we can list now all ressource groups in your subscription just by providing the Bearer token in the request. Check out the [Azure Rest API reference][azure-rest-api] for more info on how to deploy and manage your workloads.


GET Url: `https://management.microsoftazure.de/subscriptions/<SubscriptionId>/resourcegroups?api-version=2015-01-01`

HEADER: 
{% highlight PowerShell %}
Authorization=Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Il9mMlJNZVZOYVJjVTZPSE1NbHRwc0VmV0VJMCIsImtpZCI6Il9mMlJNZVZOYVJjVTZPSE1NbHRwc0VmV0VJMCJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQubWljcm9zb2Z0YXp1cmUuZGUvIiwiaXNzIjoiaHR0cHM6Ly9zdHMubWljcm9zb2Z0b25saW5lLmRlL2YxYzliMTI1LWUyYmYtNDhjMC1iMDI1LTIzZTQ3YzQxMDI5My8iLCJpYXQiOjE0NzkzNzI1MTEsIm5iZiI6MTQ3OTM3MjUxMSwiZXhwIjoxNDc5Mzc2NDExLCJhcHBpZCI6IjA0NGI3ZTFjLTI5OTQtNGI1NC05ODQyLWJjOWY4ZGUzMTRiMyIsImFwcGlkYWNyIjoiMSIsImlkcCI6Imh0dHBzOi8vc3RzLm1pY3Jvc29mdG9ubGluZS5kZS9mMWM5YjEyNS1lMmJmLTQ4YzAtYjAyNS0yM2U0N2M0MTAyOTMvIiwib2lkIjoiNjE3NGU3MzEtNWRiOS00NDA2LWIyMGQtM2Y3NzM1OWNlYzNiIiwic3ViIjoiNjE3NGU3MzEtNWRiOS00NDA2LWIyMGQtM2Y3NzM1OWNlYzNiIiwidGlkIjoiZjFjOWIxMjUtZTJiZi00OGMwLWIwMjUtMjNlNDdjNDEwMjkzIiwidmVyIjoiMS4wIn0.AWB2t5VixqbLMgMMTPe1J4mYxUXmPbT5pNfdXmCwTp3-u9Ahoby_yaGos2ObiGH1nQCiR0I6bBknC7-XUK3CY92UUrHYIr7qoAMnRAIvCdKxQlKP5AmUHFN3omkkwVvyUHcAdXFInzVIbxD-PeFvaaNPxPSdEfvyBf9b4GJ_l3qRZeQh_K2d6ez0cO7XoVN6-fxKWTC7O66qT5SNkZ6jdnbT3SwUpaK7FudOjzaGlhYVI6nZZS8GlJuxW6RuTfs-oly3egkqG4p2MLPginA7a7Vivkz4jA6EZ6nr4TnO4o9Oe_jcp-chFxLUsc6iCElazmG5n2hxZAEhjFxGYKzD3w
{% endhighlight %}

RESPONSE:
{% highlight Json %}
{
  "value": [
    {
      "id": "/subscriptions/<SubscriptionId>/resourceGroups/test",
      "name": "test",
      "location": "germanynortheast",
      "properties": {
        "provisioningState": "Succeeded"
      }
    }
  ]
}

{% endhighlight %}



---


[azure-powershell-wi]: https://www.microsoft.com/web/handlers/webpi.ashx/getinstaller/WindowsAzurePowershellGet.3f.3f.3fnew.appids
[azure-rest-api]: https://msdn.microsoft.com/en-us/library/azure/mt420159.aspx