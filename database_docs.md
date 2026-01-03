# Database Documentation

`araneae`'s database consists of two tables:

##### requests

| Field            | Description                                                                                              |
| ---------------- | -------------------------------------------------------------------------------------------------------- |
| `request_id`     | Primary key UUID for the request. Auto-generated using `uuid_generate_v4()`.                             |
| `url`            | Target URL of the request.                                                                               |
| `method`         | HTTP method used (e.g. GET, POST).                                                                       |
| `req_headers`    | Request headers stored as JSONB. Defaults to an empty object.                                            |
| `req_hash`       | Optional MD5-style hash of the request (32-char string). Must be unique.                                 |
| `code`           | HTTP response status code. Defaults to `-1` if not yet set.                                              |
| `resp_headers`   | Response headers stored as JSONB. Defaults to an empty object.                                           |
| `hits`           | JSONB array tracking hits or occurrences of this request. Defaults to an empty array.                    |
| `response_time`  | Time taken to receive a response, in seconds. Defaults to `-999`.                                        |
| `response_size`  | Size of the response body, typically in bytes. Defaults to `-1`.                                         |
| `query`          | Optional query string or extracted query data.                                                           |
| `response`       | Optional response body content.                                                                          |
| `source_request` | Optional UUID reference to another request that led to this one. Set to `NULL` if the source is deleted. |

and,

##### unscanned

| Field            | Description                                                                                              |
| ---------------- | -------------------------------------------------------------------------------------------------------- |
| `url`            | URL that has not yet been scanned.                                                                       |
| `source_request` | Optional UUID reference to the request that discovered this URL. Set to `NULL` if the source is deleted. |


### Database Utilities for Writing Memory Efficient Widgets

Database entries are serialized and deserialized in a format following this structure:

```py
entry = {
    'method': resp.request.method,
    'req_headers': dumps(dict(resp.request.headers)),
    'req_hash': self.hash_req(resp.request), # uuid can uniquely identify requests, but when placing new requests into 
    'code': resp.status_code,
    'resp_headers': dumps([dict(resp.headers)]),
    'hits': '[]',
    'response_time': resp.elapsed.total_seconds(),
    'response_size': len(resp.text),
    'query': self.clean_pg_string(query) if query else None,
    'response': self.clean_pg_string(resp.text) if self.settings['save-bodies'] else None,
    'source_request': await self.get_req_id(resp.request)
}
```


> [!NOTE]
> The manager interface defines an async generator method called `stream_found`, which can be used to stream entries directly from the database.
>
> Entries are to be looped over like so: 
> ```py
> async for url, data in manager.stream_found(): ...
> ```
> 
> `data` matches the `entry` format shown above.
>
> This is useful to keep the amount of memory used constant to ensure that the program doesn't crash when attempting to write output.


To collect a single entry from the database from a url, you can use:

```py
entry = await manager.database.get_entry(URL_TO_FIND)
```

> [!IMPORTANT]
> **URLs are keyed without removing query parameters.**
>
> This choice was made because in some cases query parameters change the behavior of an application.
> 
> The downside however, is that you must know the query params for a request you are searching for if they existed.

To recieve a generator of all urls that are present as keys in the database, you can use

```py
url = await manager.database.get_all_urls()
```

This method *does not* stream urls, thus it should be avoided for modules that should be available in long-term scan contexts.