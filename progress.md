### **In here I will be documenting my progress as I go and learn new things**
```python

import json

with open('platforms.json', 'r') as f :
    data = json.load(f)

platforms = data['platforms']

username = 'guyvolvo'
for platform_name , platform_info in platforms.items():
    url = platform_info['url'].format(username)
    print(f"{platform_name}: {url}")

```

Figuring out how to actually use the json file I created and print out all the platforms on there with my name 

<img width="552" height="323" alt="image" src="https://github.com/user-attachments/assets/82427a60-4335-4044-ba5d-2abb5242a6f6" />

Works fine,

Now I will probably be using argparse in this project like I did with my directory enumerator since I really like how it turned out \
These are all the arguments I've decided to include in the framework

```python
import json
import argparse


def main():
    parser = argparse.ArgumentParser(
        description='Canopy - Username Enumeration Tool',
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
    Examples:
      canopy -u johndoe
      canopy -u johndoe -t 50 --timeout 15
      canopy -u johndoe -o report.json --format json
      canopy -u johndoe --categories social,gaming
      canopy --list-categories
            """
    )
    # Target group
    target_group = parser.add_argument_group('Target Options')
    target_group.add_argument('-u', '--username', help='Username to search for', type=str)
    target_group.add_argument('-U', '--usernames', help='File containing list of usernames (one per line)', type=str)

    # Performance group
    performance_group = parser.add_argument_group('Performance Options')
    performance_group.add_argument('-t', '--threads', help='Number of concurrent threads (default: 10)', type=int,
                                   default=10)
    performance_group.add_argument('--timeout', help='Request timeout in seconds (default: 10)', type=int, default=10)
    performance_group.add_argument('--delay', help='Delay between requests in seconds (default: 0)', type=float,
                                   default=0)
    performance_group.add_argument('--rate-limit', help='Max requests per second (default: unlimited)', type=int,
                                   default=None)
    # Filter group
    filter_group = parser.add_argument_group('Filtering Options')
    filter_group.add_argument('-c', '--categories', help='Comma-separated categories to check (e.g., social,gaming)',
                              type=str)
    filter_group.add_argument('-p', '--platforms', help='Comma-separated specific platforms to check', type=str)
    filter_group.add_argument('--exclude', help='Comma-separated platforms to exclude', type=str)
    filter_group.add_argument('--only-found', help='Only show found accounts', action='store_true')

    # Output group
    output_group = parser.add_argument_group('Output Options')
    output_group.add_argument('-o', '--output', help='Output file path', type=str)
    output_group.add_argument('-f', '--format', help='Output format: json, csv, html, txt (default: txt)', type=str,
                              choices=['json', 'csv', 'html', 'txt'], default='json')
    output_group.add_argument('-v', '--verbose', help='Verbose output', action='store_true')
    output_group.add_argument('-q', '--quiet', help='Minimal output (only results)', action='store_true')
    output_group.add_argument('--print-found', help='Print found accounts in real-time', action='store_true')

```

Decided to go with the Claude workflow 

<img width="218" height="217" alt="image" src="https://github.com/user-attachments/assets/6f708e63-77f8-46ea-901f-18123534a84b" />

I will build the utils.py first which will handle file I/O and loading the platform database based on filters.\
Here is what I have so far

```python
import json


def laod_platform(file_path, categories=None, exclude=None, specific=None):
    with open(file_path, 'r') as f:
        data = json.load(f)

    filtered = {}
    cat_list = categories.split(',') if categories else []
    excl_list = exclude.split(',') if exclude else []
    spec_list = specific.split(',') if specific else []

    for name, info in data.items():
        if spec_list and name not in spec_list: continue
        if excl_list and name in excl_list: continue
        if cat_list and info.get('category') not in cat_list: continue
        filtered[name] = info

    return filtered


def load_usernames(filepath):
    with open(filepath, 'r') as f:
        return [line.strip() for line in f if line.strip()]
```

The query generator will handle what URL we will actually be testing

```python
def generate_queries(usernames, platform_data):
    
    queries = []
    
    for user in usernames:
        for p_name, p_info in platform_data.items():
            queries.append({
                "username": user,
                "platform": p_name,
                "url": p_info['url_main'].format(username=user),
                "error_type": p_info.get('errorType'),
                "error_msg": p_info.get('errorMsg')
            })

    return queries
```

Added a config.py file 

```python
DEFAULT_TIMEOUT = 10
DEFAULT_THREADS = 10
USER_AGENT = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 " \
             "Safari/537.36"
```

And started work on the username_checker file 

```python
import requests
from config import USER_AGENT


def check_username(query, timeout):
    # These headers make the request look like it's coming from your Chrome browser
    headers = {
        "User-Agent": USER_AGENT,
        "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
        "Accept-Language": "en-US,en;q=0.5",
        "DNT": "1",  # Do Not Track request
        "Connection": "keep-alive",
        "Upgrade-Insecure-Requests": "1"
    }

    try:
        response = requests.get(
            query['url'],
            headers=headers,
            timeout=timeout,
            allow_redirects=True
        )

        if response.status_code == 200:
            return {"status": "FOUND", "platform": query['platform'], "url": query['url']}
        elif response.status_code == 404:
            return {"status": "MISSING", "platform": query['platform']}
        elif response.status_code == 403:
            return {"status": "BLOCKED", "platform": query['platform']}

    except requests.exceptions.Timeout:
        return {"status": "TIMEOUT", "platform": query['platform']}
    except Exception as e:
        return {"status": "ERROR", "platform": query['platform'], "msg": str(e)}

    return {"status": "MISSING", "platform": query['platform']}
```
Pretty barebones but I'll add to it as we go \

Stopped updating because I locked in on the code will have to add more to this in the future !
