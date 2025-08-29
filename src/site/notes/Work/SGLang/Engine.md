---
{"dg-publish":true,"permalink":"/work/sg-lang/engine/"}
---




Dependency on a [[Work/SGLang/Scheduler\|Scheduler]]


```python title:entrypoints/engine.py

class Engine:
	def _launch_subprocesses():
		proc = mp.Process(target=run_scheduler_process,...)

```