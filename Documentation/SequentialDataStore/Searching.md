---
uid: sdsSearching
---

# Search in SDS

You can search for objects using texts, phrases and fields in Sequential Data Store (SDS). The REST API requests (or .NET client libraries methods ``GetStreamsAsync``, ``GetTypesAsync``, and ``GetStreamViewsAsync``) return items that match specific search criteria within a given namespace. By default, the query parameter applies to all searchable object fields.

Let's say a namespace contains the following streams:

**streamId** | **Name**  | **Description**
------------ | --------- | ----------------
stream1      | tempA     | Temperature from DeviceA
stream2      | pressureA | Pressure from DeviceA
stream3      | calcA     | Calculation from DeviceA values

A ``GetStreamsAsync`` call with different ``query`` values will return the content below:

**Query string**     | **Returns**
------------------  | ----------------------------------------
``temperature``  | stream1
``calc*``        | stream3
``DeviceA*``     | stream1, stream2, stream3
``humidity*``    | nothing

#### Requests
The ``orderby`` parameter supports search in streams and types. Use it to return the result in a sorted order. The default value for this  parameter is ascending. For descending order, specify ``desc`` after the ``orderby`` field value. It can be used in conjunction with ``query``, ``skip``, and ``count`` parameters.

 ```text
	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=name:pump name:pressure&orderby=name

	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=name:pump name:pressure&orderby=id asc

	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=name:pump name:pressure&orderby=name desc

	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=name:pump name:pressure&orderby=name desc&skip=10&count=20
 ```
##### Parameters
`string query`  
An optional parameter representing a string search. 

`int skip`  
An optional parameter representing the zero-based offset of the first SdsStream to retrieve. 
If unspecified, a default value of 0 is used.

`int count`  
An optional parameter representing the maximum number of SdsStreams to retrieve. 
If unspecified, a default value of 100 is used.

`string orderby`  
An optional parameter representing sorted order which SdsStreams will be returned. A field name is required. The sorting is based on the stored values for the given field (of type string). For example, ``orderby=name`` would sort the returned results by the ``name`` values (ascending by default). Additionally, a value can be provided along with the field name to identify whether to sort ascending or descending, by using values ``asc`` or ``desc``, respectively. For example, ``orderby=name desc`` would sort the returned results by the ``name`` values, descending. If no value is specified, there is no sorting of results.

#### .NET client libraries method
Use parameters ``skip`` and ``count`` to return what you need when a large number of results match the ``query`` criteria.``count`` indicates the maximum number of items returned by the ``GetStreamsAsync()`` or ``GetTypesAsync()`` call. The maximum value of 
the ``count`` parameter is 1000. ``skip`` indicates the number of matched items to skip over before returning matching items. You use the skip parameter when more items match the search criteria than can be returned in a single call. 

For example, let's say there are 175 streams that match the search criteria: “temperature”.

The call below will return the first 100 matches:
```csharp
    _metadataService.GetStreamsAsync("temperature", skip:0, count:100)
```

By setting ``skip`` to 100, the following call will return the remaining 75 matches, skipping over the first 100:
```csharp
    _metadataService.GetStreamsAsync("temperature", skip:100, count:100)
```

## Search for streams
Streams search is exposed through the REST API call and the client libraries method ``GetStreamsAsync``.

For more information on SdsStreams properties, see [Streams](xref:sdsStreams#streampropertiestable).

**Searcheable Properties**
| Property          | Searchable  |
|-------------------|-------------|
| Id                | Yes		  |
| TypeId            | Yes		  |
| Name              | Yes		  |
| Description       | Yes		  |
| Indexes           | No		  |
| InterpolationMode | No		  |
| ExtrapolationMode | No		  |
| PropertyOverrides | No		  |

**Searcheable Child Resources**
| Property          | Searchable  |
|-------------------|-------------|
| [Metadata](xref:sdsStreamExtra)*		| Yes		  |
| [Tags](xref:sdsStreamExtra)*	| Yes		  |
| ACL | No		  |
| Owner | No		  |
**\*Notes on metadata and tags:** You can access stream metadata and tags through Metadata and Tags API respectively. Metadata and tags are associated with SdsStream objects and can be used as search criteria. See [below](#Stream_Metadata_search_topic) for more information.

#### Request
Search for streams using the REST API and specifying the optional `query` parameter:
 ```text
      GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query={query}&skip={skip}&count={count}
 ```
##### Parameters
`string query`  
An optional parameter representing a string search. 

`int skip`  
An optional parameter representing the zero-based offset of the first SdsStream to retrieve. 
If unspecified, a default value of 0 is used.

`int count`  
An optional parameter representing the maximum number of SdsStreams to retrieve. 
If unspecified, a default value of 100 is used.

#### .NET client libraries method
``GetStreamsAsync`` is an overloaded method that is used to search for and return streams. When you call an overloaded method, the software determines the most appropriate method to use by comparing the argument types specified in the call to the method definition.
```csharp
      _metadataService.GetStreamsAsync(query:"QueryString", skip:0, count:100);
```

The Stream fields valid for search are identified in the fields table located on the [Streams](xref:sdsStreams) page. Note that Stream Metadata has unique syntax rules, see [How Searching Works: Stream Metadata](#Stream_Metadata_search_topic).

## Search for types
Types search is exposed through the REST API and the client libraries method ``GetTypesAsync``. 

For more information on SdsType properties, see [Types](xref:sdsTypes#typepropertiestable).

**Searcheable Properties**
| Property          | Searchable |
|-------------------|------------|
| Id                | Yes        |
| Name              | Yes        |
| Description       | Yes        |
| SdsTypeCode       | No         |
| InterpolationMode | No         |
| ExtrapolationMode | No         |
| Properties        | Yes, with limitations* |
**\*Notes on Properties field:** each SdsTypeProperty of a given SdsType has its Name and Id included in the Properties field. This includes nested SdsTypes of the given SdsType. Therefore, the searching of Properties will distinguish SdsTypes by their respective lists of relevant SdsTypeProperty Ids and Names.

#### Request
Search for types using the REST API and specifying the optional `query` parameter:
 ```text
      GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Types?query={query}&skip={skip}&count={count}
 ```
##### Parameters
`string query`  
An optional parameter representing a string search. 

`int skip`  
An optional parameter representing the zero-based offset of the first SdsType to retrieve. 
If unspecified, a default value of 0 is used.

`int count`  
An optional parameter representing the maximum number of SdsTypes to retrieve. 
If unspecified, a default value of 100 is used.

#### .NET client libraries method
``GetTypesAsync`` is an overloaded method that is used to search for and return types. 
```csharp
      _metadataService.GetTypesAsync(query:"QueryString", skip:0, count:100);
```
## Search for stream views
Stream views search is exposed through the REST API and the client libraries methodd ``GetStreamViewsAsync``. 
For more information on SdsStreamViews properties, see [Stream Views](xref:sdsStreamViews#streamviewpropertiestable).

**Searcheable Properties**
| Property     | Searchable |
|--------------|------------|
| Id           | Yes		|
| Name         | Yes		|
| Description  | Yes		|
| SourceTypeId | Yes		|
| TargetTypeId | Yes		|
| Properties   | Yes, with limitations* |
**\*Notes on Properties field:** SdsStreamViewProperty objects are not searchable. Only the SdsStreamViewProperty's SdsStreamView is searchable by its Id, SourceTypeId, and TargetTypeId, which are used to return the top level SdsStreamView object when searching. This includes nested SdsStreamViewProperties.

#### Request
Search for stream views using the REST API and specifying the optional `query` parameter:
 ```text
    GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/StreamViews?query={query}&skip={skip}&count={count}
 ```
##### Parameters
`string query`  
An optional parameter representing a stream view search. 

`int skip`  
An optional parameter representing the zero-based offset of the first SdsStreamView to retrieve. 
If unspecified, a default value of 0 is used.

`int count`  
An optional parameter representing the maximum number of SdsStreamView to retrieve. 
If unspecified, a default value of 100 is used.

#### .NET client libraries method
``GetStreamViewsAsync`` is an overloaded method that is used to search for and return stream views. 
```csharp
    _metadataService.GetStreamViewsAsync(query:"QueryString", skip:0, count:100);
```

## Tokenization
Tokenization is the process of breaking a string sequence into pieces called tokens using specific characters to delimit tokens. User-specified queries are tokenized into search terms. How the query string is tokenized can affect search results.

Delimit the terms with 1) a space, or 2) one or more punctuation characters (``*``, ``!``, ``?``, ``.``, for example) and a space. Query string followed without space by other punctuation characters does not trigger tokenization and is treated as part of the term. 

If your query has a wildcard (``*``) operator after a punctuation character, neither the punctuation nor the wildcard operator is tokenized. To specifically search for a term that has trailing punctuation, enclose the query in quotation marks to ensure that the punctuation is part of the query. See examples below:

 Term | Tokenized Term | Description
----------|--------------|-----------
``Device.1`` | ``Device.1``| The token includes ``.1`` because there is no space between it and ``Device``.
``Device!!1`` | ``Device!!1``| The token includes ``!!1`` because there is no space between it and ``Device``.
``Device. ``*space*  | ``Device``| ``.`` and the following space demarcates ``Device`` as the token term.
`Device!! `*space* | ``Device``| ``!!`` and the following space demarcates ``Device`` as the token term.
``Device!*`` | ``Device``| The token does not include ``!*`` because neither is tokenized if a wildcard operator follows a punctuation character.
``"Device!"*`` | ``Device!``| ``Device!`` is the token because the string is enclosed in double quotes.

## Search operators
You can use search operators in the ``query`` string to get more refined search results. Use operators ``AND``,``OR``, and ``NOT`` in all caps. 

Operator | Description
----------|-------------------------------------------------------------------
``AND`` | AND operator. ``cat AND dog`` searches for both "cat" and "dog".  
``OR``  | OR operator. ``cat OR dog`` searches for either "cat" or "dog", or both. 
``NOT`` | NOT operator. ``cat NOT dog`` searches for "cat" or those without "dog".
``*``   | Wildcard operator. Matches 0 or more characters. ``log*`` searches for those starting with "log" ("log", "logs" or "logger" for example.); ignores case.
``:``   | Field-scoped query. Specifies a field to search. ``id:stream*`` searches for streams whose ``id`` field starts with "stream", but will not search other fields like ``name`` or ``description``. See [Field-scoping operator] (#fieldScoped) below.
``" "`` | Quote operator. Scopes the search to an exact sequence of characters. While ``dog food`` (without quotes) searches for instances with "dog" or "food" anywhere in any order, ``"dog food"`` (with quotes) will only match instances that contain the whole string together and in that order.
``( )`` | Precedence operator. ``motel AND (wifi OR luxury)`` searches for either "wifi" or "luxury", or "wifi" and "luxury" at the intersection of "motel".

### Examples
**Query string**     | **Matches field value** | **Does not match field value**
------------------ | --------------------------------- | -----------------------------
``mud AND log`` | log mud<br>mud log | mud<br>log
``mud OR log`` | log mud<br>mud<br>log | 
``mud AND (NOT log)`` | mud | mud log
``mud AND (log OR pump*)`` | mud log<br>mud pumps | mud bath
``name:stream* AND (description:pressure OR description:pump)`` | The name starts with "stream" and the description has the either "pressure" or "pump", or both. | 

### <a name="fieldScoped">Field-scoping (``:``) operator</a>
You can qualify the search to a specific field using the ``:`` operator.  

	fieldname:fieldvalue

#### Request
 ```text
	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=name:pump name:pressure
 ```

#### .NET client libraries method
```csharp
	GetStreamsAsync(query:"name:pump name:pressure");
```

### Wildcard (``*``) operator
You can use the wildcard operator (``*``) to complement an incomplete string. It can only be used once per token, unless in a 'contains-type' query clause: one at the beginning and another at the end (``*Tank*`` but not ``*Ta*nk``, ``Ta*nk*`` or ``*Ta*nk*``, for example). Token is 

**Query string**     | **Matches field value** | **Does not match field value**
------------------ | --------------------------------- | -----------------------------
``log*`` | log<br>logger | analog
``*log`` | analog<br>alog | logg
``*log*`` | analog<br>alogger | lop
``l*g`` | log<br>logg | lake swimming (``*`` does not span across two tokens)

**Supported**     | **Not Supported**
------------------ | ----------------------------------------
``*``<br>``*log``<br>``l*g``<br>``log*``<br>``*log*`` <br>``"my log"*``	| ``*l*g*``<br>``*l*g``<br>``l*g*`` <br>``"my"*"log"``

#### Request
 ```text
	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=log*
 ```

#### .NET client libraries method
```csharp
	GetStreamsAsync(query:"log*");
```

### Double quotes (``""``) operator	
Tokenized search terms are delimited by whitespace and punctuation. To include these delimiters in a search, enclose them in double quotes.	

When using double quotes, the matching string must include the whole value of the field on the object being searched. Partial strings will not be matched unless wildcards are used. For example, if you're searching for a stream with description ``Pump three on unit five``, a query ``"unit five"`` will not match the description, but ``*"unit five"`` will.

Note that while wildcard (``*``) can be used either in or outside of quotes, it is treated as a string literal inside quotes.  For example, you can search for ``"dog food"*`` to find a string that starts with "dog food", but if you search for ``"dog food*"``, it will only match the value of "dog food*".

 **Query string**     | **Matches field value** | **Does not match field value**	
------------------ | --------------------------------- | -----------------------------	
``"pump pressure"`` | pump pressure | pressure <br> pressure pump <br> pump pressure gauge	
``"pump pressure"*`` | pump pressure <br> pump pressure gauge | pressure <br> pressure pump <br> the pump pressure gauge
``*"pump pressure"`` | pump pressure <br> the pump pressure | pressure <br> pressure pump <br> the pump pressure gauge
``*"pump pressure"*`` | pump pressure <br> pump pressure gauge <br> the pump pressure gauge | pressure <br> pressure pump 
``"pump*pressure"`` | pump\*pressure | pump pressure <br> the pump pressure gauge

#### Request
```text	
	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query="pump pressure"	
```	

#### .NET client libraries method
```csharp	
	GetStreamsAsync(query:"\\"pump pressure\\"");	
```	

## <a name="Stream_Metadata_search_topic">How search works with stream metadata</a>
[Stream metadata](xref:sdsStreamExtra) behaves differently with search syntax rules described in the previous sections. Let's say that there's a namespace that has streams below with respective metadata key-value pairs:

**streamId** | **Metadata**
------------ | --------- 
stream1      | { manufacturer, company }<br>{ serial, abc }
stream2      | { serial, a1 }
stream3      | { status, active }<br>{ second key, second value }   
 

### Field-scoping (``:``) Operator
A stream metadata key is only searchable in association with its value. This pairing is defined using the field-scoping (``:``) operator. 

	myStreamMetadataKey:myStreamMetadataValue

Metadata key is not searched if the operator (``:``) is missing in the query string: the search is then limited to metadata values along with other searchable fields in the stream.

**Query string**            | **Returns** | **Description**
------------------------   | ------------- |-------------
``manufacturer:company``  | stream1 | Searches and returns stream1.
``company``               | stream1      | Searches only the metadata values due to lack of ``:`` operator and returns stream1.
``a*``  | stream1, stream2, stream3       | Searches the metada values and returns all three streams.

#### Request
 ```text
	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=manufacturer:company
 ```

#### .NET client libraries method
```csharp
	GetStreamsAsync(query:"manufacturer:company");
```

### Wildcard (``*``) Operator  
Wildcard (``*``) character can be used both in metada keys and values with one caveat: wildcard (``*``) used in the field (left of field-scoped operator (``:``)) will only search within stream metadata.

**Query string**     | **Returns** | **Description**
------------------  | ---------------- | ------------------------
``manufa*turer:compan*``  | stream1 | Searches and returns stream1.
``ser*al:a*``  | stream1, stream2 | Searches and retrns stream1 and stream2.
``s*:a*``  | stream1, stream2, stream3 | Searches and returns all three streams. 
``Id:stream*``  |  stream1, stream2, stream3 | Searches all fields and returns three streams. 
``Id*:stream*``  | nothing | Wildcard in the field limits the search to metadata. Returns nothing because there is no metadata by that meets the criteria.

#### Request
 ```text
	GET api/v1/Tenants/{tenantId}/Namespaces/{namespaceId}/Streams?query=manufa*turer:compan*
 ```

#### .NET client libraries method
```csharp
	GetStreamsAsync(query:"manufa*turer:compan*");
```

