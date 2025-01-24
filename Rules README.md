# Scanner Rules

To create scanner rules, you can simply create a python file and insert it into the `rules` directory.

This file must contain a function similar to the one below:

```py
def rule(resp, settings, canaries):
    ...
    return hit
```

The `resp` parameter contains an http response object from the requests module.
The `settings` parameter contains a dictionary with each of the settings for the tool.
The `canaries` parameter contains a set containing all of the canary strings.

This function must return a "hit" object (or None), which is a dictionary in the format:

```py
{   
    'category': category, 
    'url': url, 
    'severity': severity (informational/low/medium/high), 
    'info': description
}
```

# Example

Below is an example from the `reflectedheaders.py` rule:

```py
from tools.utilities import Fore, tether_rule


@tether_rule('reflectheaders','reflectheadersscript', tether_only=True)
def rule(resp, settings, canaries):
    *_, (k, v), _ = resp.request.headers.items() #assumes there is only one canary cookie
    if v in canaries and v in resp.text:
        hit = {
            'category': 'Risk of Web Cache Poisoning!',
            'url': resp.request.url, 
            'severity': 'high',
            'info': f'Reflected value from `{k}: {v}` - Response headers: {resp.headers}'
            }
        return hit
```

# Decorator

This rule uses the `tools.utilities.tether_rule` decorator, which takes 2 strings:
    The first string is the name of the rule file that should be tethered to this script (excluding the .py extension)
    The second string is the name of the script file that this rule should be tethered to (excluding the .py extension)

Optionally, this decorator also takes a `tether_only` parameter that prevents the rule from being run by flagger mode, meaning it will only be triggered by the rule(s) it is tethered to.

This will automatically activate the flagging of this rule whenever a script which has a name that matches the first string is invoked by a user.
