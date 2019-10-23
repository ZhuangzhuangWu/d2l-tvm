# Data Types

The `tvm_vector_add` module we created in :numref:`ch_vector_add` only accepts `float32` tensors. Let's extend it to other data types in this section.


## Specifying a Data Type

To use a data type different to the default `float32`, we can specify it explicitly when creating the placeholders. In the following code block, we generalize the vector addition expression defined in :numref:`ch_vector_add` to accept an argument `dtype` to specify the data type. In particular, we pass `dtype` to `tvm.placeholder` when creating `A` and `B`. The result `C` then obtain the same data type as `A` and `B`. 

```{.python .input}
import tvm
import d2ltvm
import numpy as np

n = 100

def tvm_vector_add(dtype):
    A = tvm.placeholder((n,), dtype=dtype)
    B = tvm.placeholder((n,), dtype=dtype)
    C = tvm.compute(A.shape, lambda i: A[i] + B[i])
    print('expression dtype:', A.dtype, B.dtype, C.dtype)
    s = tvm.create_schedule(C.op)
    return tvm.build(s, [A, B, C])
```

Let's compile an module that accepts `int32` tensors.

```{.python .input}
mod = tvm_vector_add('int32')
```

Then we define a function to verity the results with a particular data type. Note that we pass a constructor that modify the tensor data type by `astype`.

```{.python .input}
def test_mod(mod, dtype):
    a, b, c = d2ltvm.get_abc(n, lambda x: tvm.nd.array(x.astype(dtype)))
    print('tensor dtype', a.dtype, b.dtype, c.dtype)
    mod(a, b, c)
    np.testing.assert_equal(c.asnumpy(), a.asnumpy() + b.asnumpy())

test_mod(mod, 'int32')
```

Let's try other data types as well

```{.python .input}
for dtype in ['float16', 'float64', 'int8','int16', 'int64']:
    mod = tvm_vector_add(dtype)
    test_mod(mod, dtype)
```

## Converting Elements Data Types

Besides constructing a tensor with a particular data type, we can also cast the data type of a tensor element during the computation. The following method is same as `tvm_vector_add` but allows to accept `a` and `b` with the `float32` type. Because of the `astype` used to convert the elements of `a` and `b`, the result `C` will have the data type specified by `dtype`.

```{.python .input}
def tvm_vector_add_2(dtype):
    A = tvm.placeholder((n,))
    B = tvm.placeholder((n,))
    C = tvm.compute(A.shape, 
                    lambda i: A[i].astype(dtype) + B[i].astype(dtype))
    print('expression dtype:', A.dtype, B.dtype, C.dtype)
    s = tvm.create_schedule(C.op)
    return tvm.build(s, [A, B, C])
```

Define a similar test function to verfiy the results. 

```{.python .input}
def test_mod_2(mod, dtype):
    a, b, c = d2ltvm.get_abc(n)
    a_tvm, b_tvm = tvm.nd.array(a), tvm.nd.array(b)
    c_tvm = tvm.nd.array(c.astype(dtype))
    print('tensor dtype', a_tvm.dtype, b_tvm.dtype, c_tvm.dtype)
    mod(a_tvm, b_tvm, c_tvm)
    np.testing.assert_equal(c_tvm.asnumpy(), a.astype(dtype) + b.astype(dtype))

mod = tvm_vector_add('int32')
test_mod_2(mod, 'int32')
```

## Summary

- We can specify the data type by `dtype` when creating placeholders.
- The data type of a tensor element can be cased by `astype`.