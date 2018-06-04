    Title: Logging start and finish in Python
    Date: 2018-06-04T20:07:09
    Tags: python


Just a quick post on how to log, in simple Python scripts, the start and finish
times of a function.

```python
import logging
import time
import sys


def main():
    start = time.time()
    logging.basicConfig(
        level=logging.DEBUG,
        format='%(asctime)s %(levelname)s - %(message)s',
        datefmt='%H:%M:%S',
    )
    logging.info("Start processing after %s", format_time_since(start))
    # Do some processing ...
    logging.info("Finished processing after %s", format_time_since(start))
    sys.exit(0)


def format_time_since(start):
    now = time.time()
    elapsed = now - start
    return time.strftime("%H:%M:%S", time.gmtime(elapsed))
```
