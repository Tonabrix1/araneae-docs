# Developer's Guide to `araneae` Module Creation

Welcome to the **`araneae` Developer's Guide**! This section will walk you through creating your own modules. 

---

> [!IMPORTANT]
> ## What is a Module
> Araneae supports dynamically adding functionality by dropping in python files structured for its use, these are referred to as modules.
> 
> Each function inside a module that is set for araneae's use is known as a widget, this includes any function that contains one of the following strings in its name:
> - **spider**, **miner**, **parse**, **analyze**, **rule**

---

### 1. **Scanner Scripts**
Scanner scripts prepare requests to be sent. 

To add a scanner script widget, include the name of the manager it will be used with, this includes the **spider** and **miner** :

Below is an example of a spider script widget from `default_spider.py`:

```py
@add_cli_arg('-vt', action='store_true', help='-vt | (Spider) Fuzzes DELETE, POST, PUT, PATCH, OPTIONS, and HEAD methods in each request\n')
def verb_tamper_spider(urls, manager): 
    METHODS = ['DELETE', 'POST', 'PUT', 'PATCH', 'OPTIONS', 'HEAD']
    return (prep_req(url, headers=manager.headers, method=method) for method in METHODS for url in urls)
```

##### These functions must either be generators (use the yield/yield from keywords) or return generators.

> [!NOTE]
> The elements of the generators returned should be prepared httpx requests.
> 
> `tools.utilities.prep_req` is made for this purpose and is defined like so:
> 
> ```py
> def prep_req(url, method='GET', **kwargs): return httpx.Request(method.upper(), url, **kwargs)
> ```

---

### 2. **Parser Scripts**
Parser scripts are used to parse the content of web pages and find new URLs for crawling.

To add a parser script, include the word **parse** in the function name.

Below is an example from `default_parser.py`:

```py
@add_cli_arg("-js", action='store_true', help='-js | (Parser) Parses js files for possible endpoints in quotes\n')
def parse_js(resp, manager):
    if not resp.url.endswith('js'): return

    outp = set()
    for g in re.findall(regex, resp.text):
        if g: outp.update({*g})
    return outp
```

These functions should return a set of urls that have been parsed from the content.

Urls that do not begin with a scheme/domain will automatically have the endpoint that the link was found on prefixed.

---

### 3. Rules
Rules are used when you want araneae to point out some information to the user.

These rules define specific patterns or conditions that, when matched, generate "hits" in the analysis.

To add a rule, just include the word **rule** in the function name.

Below is an example from `default_rules.py`:

```py
def cache_information_rule(resp, manager):
    # store the hash of the headers so it's not hard on memory
    if try_add(local_hits, hash(str(resp.headers))): return
    
    CACHE_IDENTIFIERS = ["Age", "X-Cache", "Cache", "X-Cache-Hits", "X-Varnish-Cache", "X-Drupal-Cache", "X-Varnish", "X-Vercel-Cache", "CF-Cache-Status", "Cache-Control"] #"CF-RAY"

    hit = None
    for iden in CACHE_IDENTIFIERS:
        if iden in resp.headers: 
            istr = f'{iden}: {resp.headers[iden]}\n'
            if try_add(discoveries, istr): continue
            if not hit: 
                hit = make_hit(category='Cache Information',
                    url=resp.request.url,
                    severity='informational')
            hit['info'] += istr
    return hit
```

These functions must either return a dictionary with a hit or `None` if nothing is found.

> [!NOTE]
> The `make_hit` function is defined for easily creating dictionaries in the right format, its signature is defined like so:
> ```py
> def make_hit(category, url, severity, info=''): # ...
> ```
> category, url, and info can be aribtrary values, but severity must be one of the following strings:
> **informational**, **low**, **medium**, **high**


---

### 4. **Analysis Modules**
Analysis modules are used to perform post-scan analysis on the collected data. 

In general, they write some information to a file, but this is not a requirement.

To add an analysis module, include the word **analyze** in the function name.

Below is an example from `default_analysis.py`:

```py
@add_cli_arg('-e', help='-e | Writes all stored endpoints, does not include canaries\n')
async def analyze_endpoints(manager):
    out_path = join(BASE_PATH, 'output/endpoints.txt')
    with open(out_path, 'w') as f:
        async for url, data in manager.stream_found(): f.write(url + '\n')
```

There are no requirements for return values on these functions.

---

# Utility Functions

### 1. Dynamically Updating CLI flags

> [!IMPORTANT]
> The `tools.utilities.add_cli_arg` decorator can be used to dynamically add arguments to araneae by passing them to `argparse.ArgumentParser.add_argument`.
> 
> Below is an example of the usage:
>
> ```py
> @add_cli_arg('-vt', action='store_true', help='-vt | (Spider) Fuzzes DELETE, POST, PUT, PATCH, OPTIONS, and HEAD methods in each request\n')
> def verb_tamper_spider(urls, manager): 
>     METHODS = ['DELETE', 'POST', 'PUT', 'PATCH', 'OPTIONS', 'HEAD']
>     return (prep_req(url, headers=manager.headers, method=method) for method in METHODS for url in urls)
> ```
> 
> The first argument is the short flag, the long flag will automatically be extracted from the function name.
> In this case, the long flag becomes `--verbtamper`.
> 
> This decorator will automatically activate the script when the flag is used.

### 2. Tethering Rules To CLI Flags

> [!IMPORTANT]
> The `tools.utilities.tether_flag` decorator can be used to automatically trigger a rule to be applied when a script or analysis module has been invoked.
> 
> This is done by passing the function you want to tether the decorator to directly in the arguments like so:
> 
> ```py
> # widget defined above with a cli arg set
> @add_cli_arg("-rh", action='store_true', help='-rh | (Miner) Tries a list of over 1,200 headers with unique canaries per endpoint, also sends random strings in commonly keyed fields ensuring a cache miss; this can be used to find cache poisoning in unkeyed headers\n')
> def reflect_headers_miner(urls, manager): ...
> 
> @tether_flag(reflect_headers_miner, tether_only=True)
> def reflection_with_cache_poisoning_rule(resp, manager): ...
> ```
> 
> In this case, whenever `-rh` is used, the `reflection_with_cache_poisoning` rule will always be applied.
> 
> This function also supports the `tether_only` keyword argument, which defaults to `False`
>
> If this value is `True`, then the `-f` flag that activates all flags will not activate it.

### 3. Miscellaneous `tools.utilities` Functions for Clean Modules

> [!NOTE] 
> ```py
> if try_add(my_set, my_finding): return # the finding was already in the set so early return
> ```
> This function is used to easily avoid creating duplicates and early return for efficiency.
> 
> It takes a set and an element and attempts to add the element to the set.
>
> If the element already existed it will return `True` (and you can continue/return), otherwise `False`
> 
> If storing large items inside the set, it is recommended to use the hash of the item instead to preserve memory in long-term scan contexts.

--

```py
stripped = remove_query(url)
``` 

can be used to safely remove the query string from a url, useful if you particularly want to avoid duplicates or only need to hit an endpoint once regardless of response.

--

```py
canary = generate_canary() # or generate_canary(x=5, y=5, z=5) etc
``` 

can be used to generate canary strings, by default `x=4, y=3, z=3` which translates to a random string with 4 uppercase chars, 3 digits, and 3 lowercase chars such as `ABCD123abc`.

--

```py
diff_string = compact_diff(text1, text1)
``` 

can be used to diff two strings and select only the changes for a quick, short output.

