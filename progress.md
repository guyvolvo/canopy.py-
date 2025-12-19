In here I will be documenting my progress as I go and learn new things \
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

Figuring out how to actually use the json file I created and print out all the platforms on there with my name \

<img width="552" height="323" alt="image" src="https://github.com/user-attachments/assets/82427a60-4335-4044-ba5d-2abb5242a6f6" />

Works fine,
