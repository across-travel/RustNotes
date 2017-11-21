---
title: Python logging
date: 2017-11-21 18:47:20
category: Python
---

`logger.py`

```python
import logging

LOG_FILE = 'app_history.log'

logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s - %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                    datefmt='%Y_%m_%d_%H:%M:%S',
                    filename=LOG_FILE,
                    filemode='a')

console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)
logging.getLogger(LOG_FILE).addHandler(console)

```
