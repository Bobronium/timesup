# timesup

Measure performance and showcase the difference between results without boilerplate.



![timesup_duper_demo](https://user-images.githubusercontent.com/36469655/215218569-1f833d77-d974-49ab-98cf-a03d9ab32899.gif)




### 🚧 Project is in development
There's no source code available yet.

If you have any feedback or ideas, you can [open the issue](https://github.com/Bobronium/timesup/issues), or  contact me directly via [bobronium@gmail.com](mailto:bobronium@gmail.com) or [Telegram](https://t.me/Bobronium).

---

### Why?
I was tired of writing benchmarks that not only measured performance, but also output it in an accessible and intuitive way ([like this one](https://github.com/Bobronium/fastenum/tree/master/benchmark)), but also easy to reproduce.
So I decided to abstract all that away so the only thing left is to write the actual code you want to measure.

I believe that by making benchmarking more accessible (and fun), I can create incentive to build faster software. 


### Features
- Output is a Python code that contains the benchmark itself and ready to be published
- Effortless comparisons of results
- Compare current implementation to one in any given git revision
- Compare interpreted CPython code to compiled code it produces via cython or mypyc
- Profiling support for expressions
- HTML output for viewing individual profiling results by expanding expressions
- Automatically selects the appropriate number of runs and repeats, if they weren't provided
- Out-of-the-box support for multiline expressions


## Showcase 
###### All snippets are self-contained
### Comparing within the same process 
`cat benchmark.py`
```py
import copy
import duper
import timesup

@timesup.it(number=100000, repeats=3)
def duper_vs_deepcopy():
    x = {"a": 1, "b": [(1, 2, 3), (4, 5, 6)], "c": []}  # i
    copy.deepcopy(x)  # t deepcopy
    dup = duper.Duper(x)  # t duper_init deepcopy
    dup.deep()  # t duper deepcopy
```
`python benchmark.py`
```py
import copy
import duper
import timesup

@timesup.it(number=100000, repeats=3)
def reconstruction():
    x = {"a": 1, "b": [(1, 2, 3), (4, 5, 6)], "c": []} 
    copy.deepcopy(x)        # ~0.00643 ms (deepcopy)
    dup = duper.Duper(x)    # ~0.00009 ms (duper_init): 69.44 times faster than deepcopy
    dup.deep()              # ~0.00014 ms (duper): 44.76 times faster than deepcopy
```

### Interpreted vs compiled code
<details>
<summary>cat fib.py</summary>

```py
def fib(n: int) -> int:
    if n <= 1:
        return n
    else:
        return fib(n - 2) + fib(n - 1)
```
`mypyc fib.py`
```
building 'compiled_test' extension
...
copying build/lib.macosx-12.0-arm64-cpython-310/fib.cpython-310-darwin.so -> 
```
</details>

`cat benchmark.py`
```py
import timesup
from fib import fib

@timesup.compare(to_interpreted=True)
def cpython_vs_mypyc():
    fib(32)
```
`python benchmark.py`
```python
import timesup
from fib import fib

@timesup.compare(to_interpreted=True)
def cpython_vs_mypyc():
    fib(32)     # ~510.73292 ms (interpreted)
    #|_________ # ~21.72464 ms (compiled): 23.51 times faster than interpreted
```


### Existing vs new code
Consider you're trying to improve performance of `app.used_to_be_slow` function.

It's crucial to have a baseline that you can compare your changes to.

By allowing to pass a target revision that changes need to be compared to, `timesup`:
- removes the need to capture the baseline beforehand
- since baseline is calculated each time automatically, you're not limited to the input you've got your baseline results with
- creates a perfect code snippet to include in PR with the changes ✨
- this snipped can be run by any of reviewers without them needing to manually download any benchmarks, measuring the baseline and testing the results


`cat benchmark.py`
```py
import timesup
from app import used_to_be_slow

@timesup.compare(to_revision="main")
def measure_improvement():
    used_to_be_slow()
```
`python benchmark.py`
```python
import timesup
from app import used_to_be_slow

@timesup.compare(to_revision="main")
def measure_improvement():
    used_to_be_slow()     # ~0.00106 ms (34dc8225)
    #|___________________ # ~0.00035 ms (changes): 2.99 times faster than 34dc8225 
```
