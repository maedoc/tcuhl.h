# tiniest CUDA helper library

A collection of helpers to keep sanity while writing CUDA code.

## Featurebugs

### error check macro

```c++
err(cudaMemcpy(foo, bar, 2<<99, cudaMemcpyDeviceToHost);
```
I could keep copying it from Stackoverflow but easier to keep it in
a handy util header.

### modp2

```c++
int n = 9234;
int m = 256;
int o = modp2<int>(n, m);
```
Uses the `n & (m - 1)` so I don't have to look it up every time.

### vector types with checked access

If I pass a pointer to stuff on the host to a device function,
I want the compiler to complain, I want bounds checking for debugging
and I want to never write a `cudaMemcpy` call by hand again. Here's
what I've got:

```c++
void do_device(dvec<int> foo);
void do_host(hvec<int> foo);

hvec<int> foo(5);
dvec<int> foo_on_device(foo);
hvec<int> bar(5);
bar.from(foo_on_device);

auto has_4_element = foo + 1;
auto has_2_element = (foo + 1).slice(1,2);

// compile with -DTCUHL_BOUNDSCHECK
// results in a printf message about out of bounds
auto x = foo_on_device[6];
```

### pybind11 "integration"

If you `-DTCUHL_HVEC_PY_ARRAY_CTOR_PLZ` then constructor for `hvec<T>` 
from `pybind11:array_t<T>` is made available, so you can just
```c++
method(pybind11::array_t<T> x) {
	do_device(x);
}
```
and the transfer happens automatically.

### Pinned memory

`hvec<T>` vectors allocate so-called pinned memory on the host for faster transfers.
This can be disabled with `-DTCUHL_DONT_PIN`.


