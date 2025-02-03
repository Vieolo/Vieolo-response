# Vieolo-response
To ease the development of our products and services, all backend servers made by Vieolo will follow a standard pattern while generating a JSON response.

The response pattern is independent of the type of request, the data it carries, status code of the response, programming language, or the intended action.

> [!NOTE]  
> The structures described here are the standards and the actual API response may contain different fields than describe here. Always check the documentation of the API.

## All options
The response JSON can have the following fields. All fields except `result` are optional and may not be included in the response.

```typescript
type baseResponse = {
	result: ServerResponseResult, // See below for all options
	object?: string,
	type?: string,
	operation?: string,
	reason?: string,
	message?: string,
	pagination?: PaginationType, // See below for detail,
    data?: object,
}

// The `result` can have one of the following values
type ServerResponseResult = 'success' |
	'not logged' |
	'not valid' |
	'already exists' |
	'does not exist' |
	'same object' |
	'not allowed' |
	'failure' |
	'not balanced' |
	'maintenance' |
	'limit exceeded' |
	'different type'

// The data in the pagination field of the response
type PaginationType = {
	limit: number,
	page: number,
	startIndex: number,
	sort?: string,
	total: number,
	totalPage: number,
	hasNext: boolean,
}


```

## Successful response
In the event of a successful response, the following will occur:
1. `result` will be `success`
2. `operation` will be present describing the action taken place
3. `data` will be present if there are any data to be returned. In case there are no data to be returned, the `data` field will not be present in the response.
4. `pagination` will be present if a set of data is being fetched with pagination.

#### Structure of `data`
In case `data` is present in the response, it will contain the data meant to be returned after the API call.

`data` will always be an object (or dict or map, based on the programming language, i.e. `{}`) containing one or many keys. The content of `data` will be added as the value of the keys and not as the direct children of `data` itself.

```diff
- {
-     'result': 'success',
-     'operation': 'fetch',
-     'data': [
-         {
-             "name": "Foo",
-             "username": "Bar",
-         }, 
-         {
-             "user": "cat", 
-             "username": "grumpy"
-         }
-     ]
- }

+ {
+     'result': 'success',
+     'operation': 'fetch',
+     'data': {
+         "users": [
+             {
+                 "name": "Foo",
+                 "username": "Bar",
+             }, 
+             {
+                 "user": "cat", 
+                 "username": "grumpy"
+             }
+         ]
+     } 
+ }
```


## Unsuccessful results

#### `not logged`
When the user is not authenticated and such authentication is necessary.

#### `not valid`
When one of the given data of the request is not valid. This invalid field can be a query paramter while sending a `GET` request or a field in the body of a non-get request.

usual fields:
- `object`: The field that is invalid
- `message`: The description on why the input is invalid


#### `already exists`
When an entry already exists in the database and such entry with the given detail cannot be duplicated.

usual fields:
- `object`: The object that is already present in the database.
- `reason`: Occasionally, the client might need additional data on which field of the given entry is duplicate or how the check for existing data was performed.

#### `does not exist`
When the user/client asked to fetch an object, either directly or indirectly, and the said object does not exist in the data.

usual fields:
- `object`: The object that is missing in the database

#### `same object`
When the given object is the same as the target object while the user is supposed to select a different object. For example, a folder trying to be a subfolder of itself

usual fields:
- `object`: The object that is repeated

#### `not allowed`
When the user is not allowed to perform the action.

usual fields:
- `reason`: The reason the user cannot perform this action. In many cases, no such explanation is provided.

#### `failure`
When the action has failed. This result may be an umbrella term for a variety of known or unknown reasons.

usual fields:
- `reason`: The reason the action has failed. In many cases, no such explanation is provided.

#### `not balanced`
When the given values do not create the necessary balance.

#### `maintenance`
When the server is under maintenance. The client should display the necessary information until the maintenance is over.

#### `limit exceeded`
When the user has filled its limit. A primary example is when the user tries to perform an action beyond its limit within the current payment plan.

#### `different type`
When the given type of the object does not match the necessary type.

usual fields:
- `type`: Explanation of the invalid type