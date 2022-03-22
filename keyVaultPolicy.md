# This is the policy to enable Key Vault notifications using Azure Event Grid

Policy definition is in the ```keyVaultPolicy.json``` file.

When assigned it creates a managed identity which has the following role:-
- Reader and Data Access
- EventGrid EventSubscription Contributor
- Contributor

Probably need to do some testing to see if Contributor is really needed - and if so you could remove the other roles.

This automatically send notifications to a URI endpoint - this could be an automation runbook like the policy notifications. 

All events are going to be sent at the moment [Event Reference](https://docs.microsoft.com/en-us/azure/event-grid/event-schema-key-vault?tabs=event-grid-event-schema#available-event-types) - to filter this in the policy add a filter section to the properties of the event subscription resource in the policy.

e.g. 

```
"properties": {
    ...
    ...
    "filter": {
        "includedEventTypes": [
            "Microsoft.KeyVault.CertificateNearExpiry",
            "Microsoft.KeyVault.CertificateExpired",
            "Microsoft.KeyVault.KeyNearExpiry",
            "Microsoft.KeyVault.KeyExpired",
            "Microsoft.KeyVault.SecretNearExpiry",
            "Microsoft.KeyVault.SecretExpired"
        ]
    }
}
```

# Receiving events

Events are sent to the endpoint soon after being raised - if you use a runbook it should do the following.

The Event Grid schema is [here](https://docs.microsoft.com/en-us/azure/event-grid/event-schema#event-schema)

1) Parse the input and grab the ```topic``` field (Contains the Key Vault resource Id)
2) Look up the tags on the resource (Should have an email address where the notification are going to be sent or you could use a default if none exists)
3) Construct a JSON body for the Logic App webhook (e.g. add key vault name, secret name, subscription, resource group, email address) \
4) Call the Logic App webhook

# Logic App

Use this to format and send the email.

1) Use the Parse JSON activity to pull apart the payload 
2) Send email as required. 