# Python Scripting Cheatsheet

> Replace this placeholder with your actual EngiDock Python cheatsheet content.

## Debugging quick reference

```python
# Drop into a debugger at this line
import pdb; pdb.set_trace()

# Print with variable name and value (Python 3.8+)
print(f"{my_var=}")

# Catch and inspect any exception
try:
    risky_call()
except Exception as e:
    print(f"{type(e).__name__}: {e}")
```

## Common errors and fixes

| Error | Likely cause |
|---|---|
| `ModuleNotFoundError` | Package not installed in the active venv |
| `IndentationError` | Mixed tabs/spaces |
| `TypeError: NoneType` | Function returned `None` where a value was expected |

---
*Full guide: see EngiDock's Python Scripting Troubleshooting Guide*
