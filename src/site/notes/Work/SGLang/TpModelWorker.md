---
{"dg-publish":true,"permalink":"/work/sg-lang/tp-model-worker/"}
---


self.model_runner: [[Work/SGLang/ModelRunner\|ModelRunner]]
[[Work/SGLang/ForwardBatch\|ForwardBatch]]

```python title:managers/tp_worker.py
class TpModelWorker:
	def forward_batch_generation(self, model_worker_batch: ModelWorkerBatch) -> Tuple(LogitsProcessorOutput, ...)
		forward_batch = ForwardBatch.init_new(model_worker_batch, self.model_runner)
		logits_output = self.model_runner.forward(forward_batch)
		next_token_ids = self.model_runner.sample(logits_output, model_worker_batch)
		
		return logits_output, next_token_ids
```
