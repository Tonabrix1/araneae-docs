# Getting Started

This guide will help you install and familiarize yourself with araneae.

## 1. Installation

```bash
git clone https://git.malectrica.com/malectrica/araneae
cd araneae
python -m pip install -r requirements.txt
python -m playwright install
```

### (Optional) Add an alias
```bash
echo "ara='python `pwd`/araneae.py'">~/.bash_aliases
source ~/.bash_aliases
```



## 2. Basic Usage

```bash
ara -u https://tar.get 
```

| Flag | Description            |
| ---- | ---------------------- |
| `-u` | URL to begin crawling. |


The spider will collect responses, then pass them to many parser threads to extract more urls to scan, if any are found the process will repeat.

By default, araneae will run until it has gotten a response from every url it found that was in-scope.



```bash
ara -uf targetfile.txt
```

| Flag  | Description                                                                     |
| ----- | ------------------------------------------------------------------------------- |
| `-uf` | File containing initial endpoints to scan. Overrides `-u` if both are provided. |


#### targetfile.txt
```
https://tar.get
https://exam.ple
...
```



```bash
ara -u https://tar.get -v
```

| Flag | Description                                                          |
| ---- | -------------------------------------------------------------------- |
| `-v` | Enables verbose mode. Required for some modules (e.g. `cache-info`). |



### Limiting Requests Sent

```bash
ara -u https://tar.get -sa 100
```

This will collect at least 100 unique responses that are in-scope before exiting.


| Flag  | Description                                                                                         |
| ----- | --------------------------------------------------------------------------------------------------- |
| `-sa` | Minimum number of in-scope URLs to collect before exiting. May slightly exceed due to async nature. |


### Scoping Rules
```bash
ara -u https://tar.get -sc dev.tar.get account.tar.get
```

This will only send requests to urls if they contain the dev or account subdomain.

| Flag  | Description                                                                                               |
| ----- | --------------------------------------------------------------------------------------------------------- |
| `-sc` | Limits scan to URLs containing listed substrings. Default is the registered domain (e.g., `example.com`). |

By default, araneae will select the second level domain and top level domain from a url as the scope so for example:

```bash
ara -u https://some.long.subdomain.example.com
```

will be scoped as `example.com`.

```bash
ara -uf targetfile
```

where target file is 

#### targetfile.txt
```
https://tar.get
https://exam.ple
```

scopes `['tar.get', 'exam.ple']`

If the user provides the scope `-sc *` the tool will accept all urls.



```bash
ara -u https://tar.get -sk dev.tar.get google.com dev.gov
```

| Flag  | Description                                           |
| ----- | ----------------------------------------------------- |
| `-sk` | Skips URLs that contain any of the listed substrings. |


### Flagger Modules

The flagger tool will be your bread-and-butter for finding interesting entrypoints and general attack surface.

```bash
ara -u https://tar.get -f -wf
```

| Flag  | Description                     |
| ----- | ------------------------------- |
| `-f`  | Enables flagger analysis.       |
| `-wf` | Writes flags to `hits.json`.    |


Existing flagger modules include:

| Module              | Description                                                  | Requires Verbose |
| ------------------- | ------------------------------------------------------------ | ---------------- |
| Application Errors  | Matches known debug/error patterns via regex and substrings. | ✅               |
| Excessive Info      | Identifies risky headers.                                    | ❌               |
| Info Leak           | Finds hardcoded secrets in JS.                               | ❌               |
| Cache Info          | Flags cache-related headers.                                 | ✅               |
| Reflect Headers     | Detects header reflection (via `-m -rh`).                    | ❌               |
| Headers Miner       | Finds unique responses from header fuzzing.                  | ❌               |
| Lang Check          | Detects page language (via `-lc`).                           | ❌               |
| Reflected Endpoints | Detects path reflection using canary (via `-re`).            | ❌               |




### Dealing With Auth/Filtering

```bash
ara -u https://tar.get -c "x=5; paymentPlan=Free; cf_clearance=TredUY8leeLFM2..." -ua "google-crawler"
```

| Flag  | Description                                              |
| ----- | -------------------------------------------------------- |
| `-c`  | Send custom cookie string in HTTP request.               |
| `-ua` | Set User-Agent string. If omitted, a random one is used. |


### Scanning Dynamic Content

```bash
ara -u https://tar.get -d
```

| Flag | Description                                                              |
| ---- | ------------------------------------------------------------------------ |
| `-d` | Uses a WebDriver to execute JS in responses.                             |
| `-w` | Specify WebDriver browser: `firefox` (default), `chromium`, or `webkit`. |

Webdriver mode is much slower, so only use this as a last resort.



## 3. Information Gathering


### Controlling Output and Analysis


```bash
ara -u https://tar.get -o filename.json
```

| Flag | Description                                                                           |
| ---- | ------------------------------------------------------------------------------------- |
| `-o` | Save scan results to this file. Defaults to `BASE_PATH/outputs/output.json`. |




```bash
ara -u https://tar.get -e -sd -sb
```

| Flag  | Output                                        |
| ----- | --------------------------------------------- |
| `-e`  | Writes endpoints to `endpoints.txt`.          |
| `-sd` | Writes unique subdomains to `subdomains.txt`. |
| `-sb` | Saves response bodies into the output JSON.   |




```bash
ara -u https://tar.get -csp -qp -tf
```

| Flag   | Output                                             |
| ------ | -------------------------------------------------- |
| `-csp` | CSPs and their URLs to `cspfile.txt`.              |
| `-qp`  | Query params and source URLs to `queryparams.txt`. |
| `-tf`  | Response timing analysis to `timings.json`.        |



## 4. Resuming A Scan


```bash
ara -u https://tar.get/unscanned -ff outputs/output.json
```

| Flag  | Description                                       |
| ----- | ------------------------------------------------- |
| `-ff` | Load found URLs from previous scan and skip them. |




## 5. The Miner Manager

In addition to the web crawling framework referred to as the spider, there is also a fuzzing framework referred to as the miner.

```bash
ara -u https://tar.get -m -rh
```

| Flag  | Description                                                    |
| ----- | -------------------------------------------------------------- |
| `-m`  | Enables Miner mode.                                            |
| `-rh` | Fuzzes 1,209 headers for reflection-based injection detection. |

Unlike the spider, the miner will not pass its outputs to the parser to collect more endpoints, and will exit after scanning all endpoints and executing all modules.


## 6. In Depth Scanning

```bash
ara -u https://tar.get -vt -js -lc
```

| Flag  | Description                                                           |
| ----- | --------------------------------------------------------------------- |
| `-vt` | Sends additional HTTP verbs (POST, PUT, etc.) for tampering analysis. |
| `-js` | Parses JavaScript files for links.                                    |
| `-lc` | Sends responses to language detection API. *(Slow)*                   |




```bash
ara -u https://tar.get -a -fa
```

| Flag  | Description                                            |
| ----- | ------------------------------------------------------ |
| `-a`  | Enables all scanning modules.                          |
| `-fa` | Enables all analysis modules (e.g. `-e`, `-tf`, etc.). |



```bash
ara -u https://tar.get -re
```

| Flag  | Description                                                      |
| ----- | ---------------------------------------------------------------- |
| `-re` | Detects endpoints reflecting canary values (possible XSS spots). |


## 7. Granular Control


```bash
ara -u https://tar.get -r 25 -p 10 -t 3
```

| Flag | Description                                                                   |
| ---- | ----------------------------------------------------------------------------- |
| `-r` | Number of requests per batch (default: 150). Reduce to prevent socket errors. |
| `-p` | Parser threads (default: 50).                                                 |
| `-t` | Thorough mode: rescan each endpoint `t` times to compare responses.           |


Thorough is very useful if you want to compare multiple responses for determining cacheing, network jitter, etc.