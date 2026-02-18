---
title: "Notes on torch code compilation"
date: 2026-02-18
---

Before we see what `torch.compile` does, we should first understand pytorch's default mode and why we'd ever want to move away from it.

PyTorch runs in **eager mode** by default. Think of it as PyTorch reading and executing your code op by op, as Python encounters each line. It's immediate, flexible, and great for prototyping — but it pays a Python interpreter cost on every single operation.

For production and deployment, we want to skip that cost. That's where compilation comes in.

# Legacy ways of compiling torch code

For deployment, we need models compiled into a form that bypasses the Python runtime on each op.

## PyTorch JIT (Just in Time compilation)

JIT is a feature that compiles pytorch models into a static graph. Here's what happens under the hood:

1. Go from source code or a trace to a graph
2. Run compiler passes through the graph — moving from `.graph` to an optimized graph, retrievable via `.graph_for(*inputs)`
3. `.graph` → bytecode → executed by the JIT virtual machine

Where JIT is useful is at the Python layer. A good way to think about it: JIT looks at your code once, compiles it into a static graph, and from then on runs that graph without the Python interpreter getting in the way. The ops still execute every time — but without Python's overhead on each one.

Here's the function we'll use to look at scripting and tracing in depth.

```python
import torch as t

def fn(x):
    for _ in range(x.dim()):
        x = x * x
    return x
```

### Scripting

Scripting reads your source code directly and compiles the logic itself into a static graph.

We can use scripting by doing `t.jit.script(fn)`. This returns an object, and we can inspect the IR:

```python
def fn(x: Tensor) -> Tensor:
  x0 = x
  for _0 in range(torch.dim(x)):
    x0 = torch.mul(x0, x0)
  return x0
```

Notice that everything is *statically typed*. Meaning the type of every variable is known before runtime. The loop is preserved as a loop.

### Tracing

Tracing works differently: run the function once with a sample input, record every tensor op that executes, and freeze that recording as the graph.

Here's the IR for the same function, but traced:

```python
def fn(x: Tensor) -> Tensor:
  x0 = torch.mul(x, x)
  return torch.mul(x0, x0)
```
What you see above is the intermediate representation of the function.      
The loop is gone. The sample input was a 2D tensor, so `x.dim()` was 2, so the loop ran twice. As we will know later in the essay, this creates issues.    


## Tracing vs Scripting

The core difference: tracing learns by *watching*, scripting learns by *reading*.

This matters when your code has branches. Consider:

```python
import torch as t

def fn(x):
    if x.sum() > 0:
        return x * 2
    else:
        return x * -1

traced = t.jit.trace(fn, t.tensor([1.0, 2.0]))
scripted = t.jit.script(fn)

print(traced(t.tensor([-1.0, -2.0])))
print(scripted(t.tensor([-1.0, -2.0])))
```

The input is all negative, so the correct answer is `[1.0, 2.0]`. `traced` gets it wrong — it watched the function run with a positive sample input, recorded the `x * 2` branch, and hardcoded it. The `if` condition was never saved. `scripted` gets it right because it compiled the actual logic.      

One could ask, "why tracing at all then?"   
The answer is tracing works when the models don't need data-dependent control flow. If run-it-once works for you function/model, torch.jit.trace will work nicely. Most simple CNNs, feedforward models are just that.  


Scripting has its own limitations though. It only supports a strict subset of Python — no arbitrary Python objects, limited standard library usage, and dynamic typing will cause it to fail. If your model code uses anything outside that subset, scripting won't work.

# Modern torch.compile stack

On Feb 12, 2023, PyTorch released [PyTorch 2.0](https://pytorch.org/blog/pytorch-2.0-release/), which introduced `torch.compile`.

What you do is simply:

```python
model = torch.compile(model)
```

There are three stages happening under the hood:

**TorchDynamo** — captures your model as a clean graph      

**AOTAutograd** — traces both forward and backward passes ahead of time     

**TorchInductor** — generates optimized low-level code for your hardware

Each of these are doing absolutely insane work and deserves their own essays. I'll write them some day.     

# References
[ezyang's blog: core pytorch dev](https://blog.ezyang.com/2025/08/state-of-torch-compile-august-2025/)

[gfg](https://www.geeksforgeeks.org/deep-learning/pytorch-jit-and-torchscript-a-comprehensive-guide/)

[deep dive into tracing and scripting by another core pytorch dev](https://lernapparat.de/jit-optimization-intro)

[pytorch blog after release of PyTorch 2.0](https://pytorch.org/blog/pytorch-2.0-release/)

[pytorch docs on torch.compiler](https://docs.pytorch.org/docs/2.10/user_guide/torch_compiler/torch.compiler.html#torch-compiler-overview)

LLMs: [Claude](claude.ai)

---

Thanks for reading      

~ Aayushya Tiwari