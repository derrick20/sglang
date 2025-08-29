---
{"dg-publish":true,"permalink":"/work/sg-lang/scheduler/"}
---




self.tp_worker: [[Work/SGLang/TpModelWorker\|TpModelWorker]]

// Init running status
self.waiting_queue: List[Req] = []


// The running decoding batch for continuous batching
self.running_batch: ScheduleBatch = ScheduleBatch(reqs=[], batch_is_full=False)

// The current forward batch
self.cur_batch: Optional[ScheduleBatch] = None

// The last forward batch
self.last_batch: Optional[ScheduleBatch] = None

self.tree_cache: [[Work/Articles/RadixCache\|RadixCache]]

Possess properties to create a disaggregated queue via [[SchedulerDisaggregationPrefillMixin\|SchedulerDisaggregationPrefillMixin]], and [[SchedulerDisaggregationDecodeMixin\|SchedulerDisaggregationDecodeMixin]].
In `disaggregation/prefill.py` and `disaggregation/decode.py`

For now, ignore.


Algorithm for [[continuous batching\|continuous batching]]:
- It handle one prefill chunk at a time
	- Weird
- But, when it finishes, it sets it aside
- We have staging areas:
	- chunk_req
		- The prefill chunk just being handled
	- running_batch
		- Defined as the collection of ongoing decoding requests
	- last_batch
		- What was just finished. 
		- It was either a prefill or a decode
		- If prefill, and finished all chunks, we may promote to a decode
		- If decode, and stop token/capacity reached, remove from the batch


```python title:managers/scheduler.py

def run_scheduler_process(server_args: ServerArgs, ...):
	scheduler = Scheduler(server_args, ...)
	if disaggregation_mode == NULL:
	elif == PREFILL:
	elif == DECODE:

'''
Separates into structures across files
'''
class Scheduler(
	SchedulerOutputProcessorMixin,
    SchedulerUpdateWeightsMixin,
    SchedulerProfilerMixin,
    SchedulerMetricsMixin,
    SchedulerDisaggregationDecodeMixin,
    SchedulerDisaggregationPrefillMixin
):

	def init_memory_pool_and_cache(self):
		self.req_to_token_pool, self.token_to_kv_pool_allocator = (self.tp_worker.get_memory_pool())
		# Sets some radix cache logic
		self.tree_cache = RadixCache(...)

	def get_next_batch_to_run(self) -> ScheduleBatch:
		# Combines 
		# TODO - chunking logic to merge batches 
		# Updates running batch.
		# May have retractions
		filter_batch()
	def filter_batch(self):
		keep_indices = [i if not self.reqs[i].finished()]
		
		 
	def event_loop_normal(self):
		# Original input loop

	def event_loop_overlap(self):
		while True:
			# Receive requests
			recv_reqs = self.recv_requests()
			
			# Process requests
			self.process_input_requests(recv_reqs)
			
			# Get next batch
			batch: ScheduleBatch = self.get_next_batch_to_run()
			
			# Run batch
			batch.launch_done = threading.Event() # Setup alert?
			result = self.run_batch(batch)
			
			# Process result
			if self.last_batch:
				self.process_batch_result()
		
```

```python title:scheduler_output_processor_mixin.py

class SchedulerOutputProcessorMixin:
	def process_batch_result_prefill(self: Scheduler, # tf?
	 batch: ScheduleBatch, result: GenerationBatchResult):
		 
```

![Pasted image 20250715111610.png](/img/user/Work/SGLang/attachments/Pasted%20image%2020250715111610.png)
![Pasted image 20250715111852.png](/img/user/Work/SGLang/attachments/Pasted%20image%2020250715111852.png)
