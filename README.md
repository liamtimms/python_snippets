# python_snippets
a place to throw some python code patterns


## Multiproc

```python

import multiprocessing as mp
from functools import partial


def function(x, y, z):
    return

def pool_handler_multifile(x, y, z_list):
    print("starting pool")
    with mp.Pool(mp.cpu_count()) as pool:
        pool.map(
            partial(function, x, y), z_list)
```
