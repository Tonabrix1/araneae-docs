# araneae (arr-ah-nay)

A modular web-crawler and fuzzing framework which preforms data collection and analysis for application security testing.

araneae can be used to rapidly enumerate a web based attack surface to make finding initial footholds and points of interest simple.

It can be used to test for both common and niche vulnerabilities once the tester has a good map of the environment.

And it can be used to preform a methodical and in-depth analysis in order to find various types of vulnerablities, misconfigurations, and bugs in every inch of an application.

![image](https://github.com/user-attachments/assets/ee840505-9aed-471d-ba7e-bf4f85a458ed)



# What can araneae do?

The basis of araneae is a web crawler, but the actual tool is much more.

Beyond simply collecting endpoints, by default araneae collects response headers, query strings, response timing data, and status codes.

This is where araneae shines, preforming its own static analysis, it can distill this ocean of information into the most interesting drops.

![image](https://github.com/user-attachments/assets/85a87051-c585-4e38-911a-edfb869682dd)



Active analysis can be preformed to elicit interesting responses or automatically find and fuzz parameters for finding myriad vulnerabilities such as xss, sqli, ssrf, and more.


![image](https://github.com/user-attachments/assets/28ef17bc-05e2-4d30-8640-1883ab527593)
###### *Solving the port swigger unkeyed header web cache poisoning lab using araneae* 



Further, with araneae's dynamic ![scanner scripts](https://github.com/malectricasoftware/araneae/blob/main/scripts/README.md), ![flagger rules](https://github.com/malectricasoftware/araneae/blob/main/rules/README.md), and ![analysis modules](https://github.com/malectricasoftware/araneae/blob/main/analysis/README.md) users can harness the power of araneae in a few lines of code.



The full help page for the tool is below:

![image](https://github.com/user-attachments/assets/1b01e59e-7e2c-41c7-9fc0-58362ac6ffe2)



# What **WILL** araneae be capable of?

The broader idea behind this tool is to build a modular web scanner framework which can iteratively preform scans to collect data, then intelligently leverage data that has been collected to dig into the most interesting parts of any application.

This is achieved by creating sequences of steps for araneae to preform known as "**recipes**", these recipes are constituent of "**ingredients**" which are steps that araneae will preform then yield some data for the next step to work on.

Some examples of hypothetical recipes may consist of the following steps:
    
    - crawl n urls from a website ->
        find admin login pages and search the source and headers for leaked versioning info ->
            feed fingerprint data into searchsploit and return exploits that may be likely
    
    - crawl n urls from a website ->
        mine for input paramaters in forms and query strings ->
            generate dynamic xss payloads for each based on the csp or lackthereof ->
                send payloads ->
                    scan responses for XSS
    
    - crawl n urls from a website ->
        mine for input paramaters in forms and query strings ->
            analyze response timings to find endpoints that take the most time to respond ->
                preform short controlled last byte/single packet attacks to check for how response timings scale ->
                    analyze response timings to discover where DOS might possible

    - crawl n urls from a website ->
        scrape regex from input fields ->
            analyze for evil regex ->
                feed dynamically constructed evil regex payloads into inputs ->
                    analyze response timings to confirm exploit


As you can imagine, many of these recipes applied in conjunction lead to powerfully complex behaviors.

And the simplicity of this approach even lends itself well even to application with reinforcement learning algorithms such as a DQNN, which can actively learn from data we collect and integrate it to build better recipes.



Strides towards achieving this have been made in araneae, by making almost all capabilities of the tool modular, making the implementation of recipes (hopefully) simple once the tool has matured enough to begin moving that direction.
