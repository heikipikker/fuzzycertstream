# fuzzycertstream
Uses the FuzzyWuzzy Python library to find possible domain matches in the SSL Transparency logs.
Can be used to find malicious use of SSL certs related to your domain.

Adapted from the Phish_Catcher tool found at https://github.com/x0rz/phishing_catcher

# Next steps
Allow to match against a list of domains

# Example Usage (update 'source' accordingly)

```import re
import certstream
import tqdm
from fuzzywuzzy import fuzz
from termcolor import colored, cprint

log_suspicious = 'suspicious_domains.log'

source = "mydomainname.com"

def match_ratio(domain):
    """Match ratio of `domain`.
    The higher the ratio, the more probable `domain` is a phishing/spoof site.
    Args:
        domain (str): the domain to check.
    Returns:
        int: the match ratio of `domain`.
    """
    ratio = fuzz.partial_ratio( source, domain)
    return ratio

def callback(message, context):
    """Callback handler for certstream events."""
    if message['message_type'] == "heartbeat":
        return

    if message['message_type'] == "certificate_update":
        all_domains = message['data']['leaf_cert']['all_domains']

        for domain in all_domains:
            ratio = match_ratio(domain.lower())

            if ratio >= 90:
                tqdm.tqdm.write(
                    "[!] Suspicious: "
                    "{} (ratio={})".format(colored(domain, 'red', attrs=['underline', 'bold']), ratio))
            elif ratio >= 80:
                tqdm.tqdm.write(
                    "[-] Not too suspicious: "
                    "{} (ratio={})".format(colored(domain, 'green', attrs=['underline']), ratio))
            if ratio >= 80:
                with open(log_suspicious, 'a') as f:
                    f.write("{}\n".format(domain))
            

certstream.listen_for_events(callback)

```
