# Client hook API

## Introduction

Mimir offers a client hook API mainly intended for custom metadata processing on behalf of a user. A few examples of common use cases that the client hook API can be useful for:

1. Format user written values to a certain predefined format.
2. Provide validation messages for specific fields back to a user that need to be addressed before metadata can be saved.
3. Automatically fill in other metadata fields based on user changes. 
4. Provide generic messages back to the user on any change. 
5. Automatically save metadata after processing on behalf of a user. 
6. Check whether the user is allowed to create a new version. 

The client hook API is intended for quick processing of metadata values. We recommend that your REST endpoint behaves in a deterministic way, and does not do heavy asynchronous processing to be able to set metadata on behalf of a user. For asynchronous workflows where server state needs to be taken into account, we recommend using our webhook API instead.

The client hook API is *not* an event mechanism. There will be no information provided on what the user has changed per payload. 

If Mimir has not received a response from your endpoint before a new payload is triggered by the user, the previous response from your endpoint will be ignored. 

This API contract does not entirely eliminate the potential for race conditions, but it does drastically reduce the chance for them occurring if implemented as advised. 

### Pre requisites

You will need a REST endpoint that the Mimir client can POST payloads to. The REST endpoint needs to be configured to allow CORS requests to be made from the Mimir domain `*.mjoll.no`, since we will POST payloads from a web browser.

### Configuration

Navigate to "System settings" →  "Integrations" → "Client hooks". Under "Your metadata endpoint" configure your REST endpoint where you can receive POST requests.

Optionally, if you'd like to control whether a user is allowed to create a new version, configure a REST endpoint under "Your version endpoint".

### Security

We recommend securing your endpoint, for example configure a query parameter into the URL configured in Mimir, where in your implementation you will check for the presence and value of this query parameter to be sure it is Mimir that's communicating with your endpoint. Ensure that no other domain is allowed to communicate with your endpoint (CORS).

## Payloads from Mimir

Mimir will POST payloads whenever a user persists a change in a field. Here are some examples of when a payload will be POSTed to your endpoint:

1. A user writes a string value into a text field, and exits the field. 
2. A user enables or disables a checkbox field.
3. A user selects a value from a dropdown
4. A user ticks multiple values in a multiple choice dropdown and closes the dropdown
5. A user picks a date in the datepicker
6. A user presses Save

Payloads will always include the latest data a user has entered, and will always contain the full metadata properties together with some additional context properties. Payload posted from Mimir can look like this:

```
{
    // Always included
    userId: string; // The user that is performing changes
    formId: string; // The current selected metadata type
    context: 'item_upload' | 'metadata_update' | 'metadata_merge' | 'copy_item' | 'new_item_version' | 'create_placeholder'; // The context of the metadata change
    formData: { // An object containing all the field values at the time of sending
        [fieldId]: fieldValues,
        ...
    }

    // Included on every first call
    initialization: true;
    
    // Included on every last call (typically when a user hits Save)
    finalization: true;

    // Included with 'metadata_update' context
    technicalMetadata: { 
        [fieldId]: fieldValues,
        ...
    }

    // Included with 'create_placeholder' context
    itemType: 'video' | 'image' | 'file' | 'audio'

    // Included with 'new_item_version' context
    oldVersionId: string;

    // Included with 'copy_item' context
    sourceItemId: string;
    copyMedia: 'high_res' | 'edit_proxy' | 'none'

    // Included with 'metadata_update', 'item_upload', 'metadata_merge' contexts
    itemId: string;

    // Included in create new version payload only
    versionsHistory: {
        id: string;
        versionCreatedOn: string;
        versionState: 'DELETED' | 'IN_RECYCLE_BIN' | 'EXISTS';
    }
}
```

Example first payload from `metadata_update` context: 

```
{
    "userId":" "4dbf7a43-d5f3-41fa-9037-89782288056e",
    "itemId": "a2349639-dcd5-43a9-83ff-8b440cb35455",
    "formId": "Common",
    "formData": {
        "title": "My item title",
        "createdOn": "2022-02-11T07:12:26.555Z",
        "my_field": "My custom field value" 
    },
    "technicalMetadata": {
        "duration": "1234.12"
    },
    "initialization": true
}
```

Example last payload from `copy_item` context:

```
{
    "userId": "4dbf7a43-d5f3-41fa-9037-89782288056e",
    "sourceItemId": "a2349639-dcd5-43a9-83ff-8b440cb35455",
    "formId": "my_custom_type",
    "formData": {
        "title": "My item title",
        "createdOn": "2022-02-11T07:12:26.555Z",
        "my_field": "My custom field value" 
    },
    "finalization": true
}
```

### Expected response payload

Mimir expects a response from your endpoint. If the response is malformed or not recognized, Mimir will ignore the request.

If Mimir does not receive a response before Mimir sends a new payload, the previous response will be ignored. 

The fully documented payload with all possibilities:

```
{
    // Metadata that should be applied. Note that even if no changes to be made, Mimir expects this data to be returned.
    formId: string;
    formData: {
        [fieldId]: fieldValues,
        ...
    }

    // Optionally instruct Mimir to enable/disable the save button for the user
    canSave: boolean,

    // Optionally instruct Mimir to save the metadata after applying the returned metadata
    doSaveMetadata: boolean,

    // Only relevant for create version endpoint, to indicate whether a user is allowed to continue or not.
    isActionAllowed: boolean;

    // Optional property to communicate back to a user with
    message: {
        // Optionally provide a message that will be displayed above the form
        general: string,
        // Optionally provide a status icon that will be displayed next to the message
        status: "error" | "success" | "warning",
        // Optionally provide a map of fieldId to string to display validation messages on specific fields
        perField: {
            [fieldId]: string,
            ...
        }
    }
}
```

Example response for updating the metadata and saving the result on behalf of the user:

```
{
    "formId": "Common",
    "formData": {
        "title": "My title"
    },
    "doSaveMetadata": true
}
```

Example response for validation informing the user that something needs to be addressed:

```
{
    "formId": "Common",
    "formData": {
        "title": "",
        "description": "My description"
    },
    "message": {
        "general": "Please address validation errors",
        "status": "error",
        "perField": {
            "title": "Empty title not allowed"
        }
    },
    "canSave": false
}
```

### Debugging

It is possible to turn on extra verbose logging to get insight on how Mimir is creating the payloads and processing the responses, which will be valuable while developing your client hook endpoint(s).

Open the dev tools in your preferred browser, and type `features.enable("logClientHooks")` in the console. A response of `undefined` is expected. After this Mimir will start to log details to the browser console.

A system wide flag is available that the Mjoll team can enable for your organisation, which will always enable this across all users. Contact us for this.

## A couple end to end examples

What follows is an end to end example with simulated POSTs and responses, which involves a typical use case of a user making some changes and your endpoint providing additional formatting / validation / field values.

1. User opens a Mimir item in the item page or in search, or any other place where an item with metadata is displayed, where only a title is currently set.

Mimir POSTs the initialization payload:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "",
        "amount": "",
        "approved": ""
    },
    "context": "metadata_update",
    "initialization": true
}
```

Let's indicate to the user that the `description` and `amount` fields are required, and respond to Mimir with:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "",
        "amount": "",
        "approved": ""
    },
    "message": {
        "general": "Please correct required fields",
        "perField": {
            "description": "Description is required",
            "amount": "Please enter a numeric value for amount"
        },
        "status": "warning"
    }
}
```

2. User goes into description and enters a description. Once the user exist the field, a new payload is sent to your endpoint:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": "",
        "approved": ""
    },
    "context": "metadata_update"
}
```

Since description now has a value, we still need to require a value for `amount`:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": "",
        "approved": ""
    },
    "message": {
        "general": "Please correct the 'amount' field",
        "perField": {
            "amount": "Please enter a numeric value for amount"
        },
        "status": "warning"
    }
}
```

3. User goes into the `amount` field and provides a value of `20`. The following payload is POSTed:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": 20,
        "approved": ""
    },
    "context": "metadata_update"
}
```

We want to ensure that the amount value is stored as a floating point with 2 decimals, so let's fix that on behalf of the user:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": 20.00,
        "approved": ""
    }
}
```

Note that no message property was provided as there's no validation error to make the user aware of. However, it is still possible to deliver a success message of some kind, for example we could have sent:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": 20.00,
        "approved": ""
    }, 
    "message": {
        "general": "All fields are valid",
        "status": "success"
    }
}
```

4. User is satisfied with their values, and hits the save metadata button in Mimir. You receive the following payload:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": 20.00,
        "approved": ""
    },
    "finalization": true
}
```

Since `finalization:true` is provided, you are guaranteed to know that this is the *last* payload to process for this metadata update process. Let's pretend you're happy with the values and want to set `approved` to `true` on behalf of the user prior to saving:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": 20.00,
        "approved": true
    }
}
```

Alternatively you could have saved the metadata on behalf of the user after having formatted the `amount` field, sparing the user from needing to save manually:

```
{
    "formId": "Common",
    "formData": {
        "title": "New item",
        "description": "A description describing the item",
        "amount": 20.00,
        "approved": ""
    }, 
    "doSaveMetadata": true
}
```

Many different workflows are possible using this workflow, and the above example is one example of the flow between Mimir and your REST endpoint.