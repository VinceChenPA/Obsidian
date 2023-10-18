
# C3. Type Annotations
**typecheckers**
```
pip install mypy
mypy <file>.py
```
# C4. Constrainting Types
## 6 techniques
- Optional
Use to replace None references in your codebase.
- Union
Use to present a selection of types.
- Literal
Use to restrict developers to very specific values.
- Annotated
Use to provide additional description of your types.
- NewType
Use to restrict a type to a specific context.
- Final
Use to prevent variables from being rebound to a new value.

> when talking about robust code, you need to consider error conditions.

### Optional Type
```python
from typing import Optional
maybe_a_string: Optional[str] = "abcdef" # This has a value
maybe_a_string: Optional[str] = None     # This is the absence of a value
```
### Union Type
> A Union[int,str] means that either an int or a str can be used for a variable.
```python
from typing import Union
from dataclasses import dataclass
@dataclass
Class Error:
	error_code: int
```
> Optional[int]=Union[int, None]

### Product and Sum Types
### Literal types
> since Python 3.8
Literal types allow you to restrict the variable to a very specific set of values.
```python
from typing import Literal
error_code: Literal[1,2,3,4,5]
```

### Annotated Types
### NewType
A NewType takes an existing type and creates a brand new type that has all the same fields and methods as the existing type.
```python
from typing import NewType
ReadyToServeHotDog = NewType("ReadyToServeHotDog", HotDog)
```
### Final Types
> since 3.8
```python
VENDOR_NAME: Final = "Viafore's Auto-Dog"
```

## C5. Collection Types
**homogeneous**
**heterogeneous**
### TypedDict
> since 3.8
```python
from typing import TypedDict
```
> `TypedDict` is only for the typechecker’s benefit. There is no runtime validation at all; the runtime type is just a dictionary.

### TypeVar
```python
from typing import TypeVar
T = TypeVar('T')
def reverse(coll: list[T]) -> list[T]:
    return coll[::-1]
```

### Generic
> to be further explored

### Abstract base classes (ABCs)
> ABCs are classes intended to be subclassed, and require the subclass to implement very specific functions.


## Typechecker
> The mechanic, who wishes to do his work well, must first sharpen his tools.