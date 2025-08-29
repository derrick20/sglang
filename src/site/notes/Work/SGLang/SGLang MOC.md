---
{"dg-publish":true,"permalink":"/work/sg-lang/sg-lang-moc/","tags":["gardenEntry"]}
---

Goal 
- Incorporate the batching for the memory allocator.
- This leads us to reap the benefits of the continuous scheduler, and the disaggregation.
	- The paging scheme also is there.


Changes made:
- Modifying ScheduleBatch to respect the spec of the encoder decoder. 
	- This required us to emulate the path of the Mllama model
	- ScheduleBatch
		- prepare_for_extend
			- Extending the fill_ids
		- prepare_encoder_info_extend
			- Allocating
	- Dynamic monkey patching of decoder attention modules:
		- This is safe because SGLang handles requests sequentially (so swapping the method with a particular forward batch is ok)
		- Now, the challenge here 
- Decision:
	- We're using -1 tensor to feed through the (B, N, 1) to indicate that the encoder_hidden_states KV values have been cached already

Future optimizations:
- The Triton kernel for the ragged batching. Basically, the key feature is avoiding padding.
	- This would involve incorporating the position bias back in.

Hacks to revert
- Overall - the concept of self.model.state management
	- In model_runner.py, :1394, the self.model.reset_state()
		- Well just delete that method
- Using the (B, N, 1) shape with the dummy encoder_hidden_state on 2nd or later decodes - match the d_model size instead.
- Disabling radix-cache - that's the confusing bit for now. It requires a lot more thinking probably.
- Batches - 

Structure Generation Language
### Primary Hierarchy
- [[Work/SGLang/Scheduler\|Scheduler]] 
- Scheduler has a [[Work/SGLang/TpModelWorkerClient\|TpModelWorkerClient]]
- TpModelWorkerClient has a [[Work/SGLang/TpModelWorker\|TpModelWorker]]
	- This maintains the input/output queue
- TpModelWorker has a [[Work/SGLang/ModelRunner\|ModelRunner]]
- ModelRunner has a Model, [[Work/SGLang/AttentionBackend\|AttentionBackend]]
	- It loads weights, imports backend
	- Has forward methods
- FlashAttentionBackend is an AttentionBackend
	- Has lower level forward methods: q, k, v

### Project Structure
- entrypoints
	- [[Work/SGLang/Engine\|Engine]]
		- TokenizerManager
		- [[Work/SGLang/Scheduler\|Scheduler]]
		- DetokenizerManager
		
- disaggregation
	- DecodeRequest, Queue, etc.
- layers
	- Attention
		- [[Work/SGLang/AttentionBackend\|AttentionBackend]]
	- LogitsProcessor
	- RadixAttention
- managers
	- [[Work/SGLang/TpModelWorkerClient\|TpModelWorkerClient]]
	- [[Work/SGLang/TpModelWorker\|TpModelWorker]]
	- [[Work/SGLang/Scheduler\|Scheduler]]
	- TokenizerManager
	- DetokenizerManager
- mem_cache
	- RadixCache
	- MemoryPool
		- [[Work/SGLang/ReqToTokenPool\|ReqToTokenPool]]
		- [[Molecules/KVCache\|KVCache]] (base class)
			- [[Work/SGLang/TokenToKVPool\|TokenToKVPool]]
	- Allocator
		- [[Work/SGLang/TokenToKVPoolAllocator\|TokenToKVPoolAllocator]]
- model_executor
	- [[Work/SGLang/ForwardBatch\|ForwardBatch]]
	- [[Work/SGLang/ModelRunner\|ModelRunner]]
Throughout, the  [[Work/SGLang/ForwardBatch\|ForwardBatch]]

---
title: SGLang
---
classDiagram
    Engine *-- Scheduler
    Engine *-- TokenizerManager
    Engine *-- DetokenizerManager
    Scheduler *-- TpModelWorkerClient
    TpModelWorkerClient *-- TpModelWorker
    TpModelWorker *-- ModelRunner
    ModelRunner *-- Model
    ModelRunner *-- AttentionBackend
	Scheduler  ..> ScheduleBatch
	TpModelWorker ..> ModelWorkerBatch
	ModelRunner ..> ForwardBatch
	ModelWorkerBatch ..> ScheduleBatch : Subset of
	ForwardBatch ..> ModelWorkerBatch : 

### Batch Transformation Pathway
[[Work/SGLang/Scheduler\|Scheduler]]:
- :770 event_loop_overlap(self)
	- batch: ScheduleBatch = self.get_next_batch_to_run() 
		- [[Work/SGLang/ScheduleBatch\|ScheduleBatch]]
			- Contains high level scheduling data (on CPU)
	- :783 - calls self.run_batch
- :1703 run_batch
	- :1718 - batch.get_model_worker_batch()
		- Converts ScheduleBatch into ModelWorkerBatch
	- :1726 Calls self.tp_worker.forward_batch_generation(model_worker_batch)
[[Work/SGLang/TpModelWorkerClient\|TpModelWorkerClient]]
- :222 forward_batch_generation(model_worker_batch)
	- ModelWorkerBatch managed by the [[Work/SGLang/TpModelWorker\|TpModelWorker]]
	- :239 Puts the batch onto the queue
- The self.forward_thread will continually pop from the queue
	- forward_thread_func(self):
		- self.worker.forward_batch_generation(model_worker_batch)
[[Work/SGLang/TpModelWorker\|TpModelWorker]]
- def forward_batch_generation(self, model_worker_batch):
	- Constructs [[Work/SGLang/ForwardBatch\|ForwardBatch]] from ModelWorkerBatch
	- Calls self.model_runner.forward
[[Work/SGLang/ModelRunner\|ModelRunner]]
- def forward(self, forward_batch)
	- self.model.forward(forward_batch.input_ids, forward_batch, ...)
[[Work/SGLang/MT5ForConditionalGeneration\|MT5ForConditionalGeneration]]
- Now, we get to our model foward function