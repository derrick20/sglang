---
{"dg-publish":true,"permalink":"/work/sg-lang/tp-model-worker-client/"}
---



+ self.worker = [[Work/SGLang/TpModelWorker\|TpModelWorker]]

```python title:managers/tp_worker_overlap_thread.py
def forward_thread_func_(self, model_worker_batch: ModelWorkerBatch, launch_done: Optional[threading.Event]):
    # Gets a batch from self.inputs_queue 
	model_worker_batch, future_token_ids_ct, sync_event = self.input_queue.get()
	if not model_worker_batch:
		break
    # Calls self.worker.forward_batch 
	logits_output, next_token_ids, can_run_cuda_graph = (
		self.worker.forward_batch_generation(
		model_worker_batch, model_worker_batch.launch_done
		)
	)
    # Places this output
    self.future_token_ids_map[
		future_token_ids_ct + 1 : future_token_ids_ct + bs + 1
	] = next_token_ids[-1] # HACK, let's see if that fixes it? Get the last token

	# Moves tokens to cpu
	# Puts onto output_queue
```
  
