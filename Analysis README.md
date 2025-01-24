# Analysis Modules

Analysis modules are scripts that preform analysis on the data araneae has collected over a scan.

To add an analysis module you must simply create a python file containing a function similar to the following:

```py
def analyze(found, settings, canaries, hits):
    ...
```

The `found` parameter contains the full dictionary of information araneae has recieved from a request, the following defines the format from `tools.utilities.insert_val`:

```py
found[href] = found.get(href, {
    'codes': [resp.status_code],
    'resp-headers': [dict(resp.headers)], #needs to be converted from CaseInsensitiveDict to be serialized
    'req-headers': [dict(resp.request.headers)],
    'response_times': [resp.elapsed.total_seconds()],
    'methods': [resp.request.method]
})
if query: found[href] |= {'queries': [query]}
if settings['save-bodies']: found[href] |= {'responses': [resp.text]}
```

The `settings` parameter is a dictionary of settings for the tool

The `canaries` parameter is a set of canary strings, which can be used to check for a reflected value in O(1) avg time

The `hits` parameter is a dictionary of hits the tool has recieved in the format `{category: {'severity': severity, 'urls': [urls], 'info': [info], 'pairs': [(url, info)]}}`

# Example

Below is an example of a spider script `analysis/endpoints.py`:

```py
from tools.utilities import add_analysis_cli_arg, join, remove_canary, Fore, BASE_PATH


@add_analysis_cli_arg("-e", "--endpoints", help='-e | Writes all stored endpoints, does not include duplicate urls with canaries\n')
def analyze(found, settings, canaries, hits):
    print(f"{Fore.LIGHTMAGENTA_EX}Collecting endpoints file...")
    with open(join(BASE_PATH, 'output/endpoints.txt'), 'w') as f: f.write('\n'.join(sorted({remove_canary(canaries, x) for x in found})) + '\n')
```

The `tools.utilities.add_analysis_cli_arg` decorator can be used to automatically add arguments via `argparse.ArgumentParser.add_argument`.

`action='store_true'` will automatically be passed along with the arguments specified, making the flag a bool which defaults to `False`.

This decorator will automatically activate the module when the flag is used.
