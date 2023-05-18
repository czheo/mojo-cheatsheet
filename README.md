# [Mojo](https://github.com/modularml/mojo/) Cheatsheet
### String

Index and Slicing
```py
from String import String
let s : String = "hello world"
print(s[0])
print(s[:5])
print(s[::2])
print(String("hello world")[0])

# !!Code below DO NOT work because `StringRef` is not subscriptable!!
let s = "hello world" # s is a `StringRef`
print(s[:5])
print("hello world"[0])
```

### Type casting

Cast `SIMD`
```py
from DType import DType
let x : UI8 = 42  # alias UI8 = SIMD[DType.ui8, 1]
let y : SI8 = x.cast[DType.si8]()
```

Cast `SIMD` to `Int`
```py
let x : UI8 = 42
let y : Int = x.to_int() # `SIMD` to `Int`
let z : UI8 = y          # `Int` to `SIMD`
```

### Interact with Python

Execute Python code directly in the playground
```py
%%python
print("The answer is", 42)
```

Invoke Python interpreter from Mojo
```py
let x: Int = 42
from PythonInterface import Python
let py = Python.import_module("builtins")
py.print("The answer is", x)
```

### Pointer

Create a `Pointer`
```py
var x : Int = 42  # x must be mutable
let xPtr = Pointer[Int].address_of(x)
print(xPtr.load()) # dereference a pointer
```

Casting type of Pointer
```py
let yPtr : Pointer[UI8] = xPtr.bitcast[UI8]()
```

Null pointer
```py
Pointer[Int]()
Pointer[Int].get_null()
```

### Interact with C
Call C function using `Intrinsics.external_call`

```
external_call[c_function_name, return_type, arg0_type, arg1_type, ...](arg0, arg1, ...) -> return_type
```

```py
from Intrinsics import external_call
# time_t ts = time(NULL);
let ts = external_call["time", Int, Pointer[Int]](Pointer[Int]())
print(ts)
```

```py
from Intrinsics import external_call

alias int = SI32
@value
@register_passable("trivial")
struct C_tm:
    var tm_sec: int
    var tm_min: int
    var tm_hour: int
    var tm_mday: int
    var tm_mon: int
    var tm_year: int
    var tm_wday: int
    var tm_yday: int
    var tm_isdst: int
    
    fn __init__() -> Self:
        return Self {
            tm_sec: 0,
            tm_min: 0,
            tm_hour: 0,
            tm_mday: 0,
            tm_mon: 0,
            tm_year: 0,
            tm_wday: 0,
            tm_yday: 0,
            tm_isdst: 0
        }

# time_t ts
let ts : Int
# time(&ts);
let tsPtr = Pointer[Int].address_of(ts)
external_call["time", Int, Pointer[Int]](tsPtr)
# struct tm *tm = gmtime(&ts)
let tmPtr = external_call["gmtime", Pointer[C_tm], Pointer[Int]](tsPtr)
let tm = tmPtr.load()
print(tm.tm_hour, ":", tm.tm_min, ":", tm.tm_sec)
```
