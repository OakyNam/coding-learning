# 10 - Modules and Packages

A module is a Python file you can import.

```python
import math

print(math.sqrt(16))
```

Import your own file:

```text
project/
  main.py
  helpers.py
```

```python
from helpers import greet
```

Packages are reusable code installed from tools like `pip`.

```bash
pip install requests
```

Practice: create `helpers.py` with one function and import it in `main.py`.
