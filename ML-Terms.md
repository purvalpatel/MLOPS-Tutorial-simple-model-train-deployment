RAG
-----
RAG (Retrival Augmented Generation) <br>
R (Retrival)  - Fetch the information from your data source. (PDF, Websites, docs etc. ) <br>
A (Augmentation) - Insert (Augment) the retrived data into LLM's Prompt. <br>
G (Generation) - LLM uses the retrived context to generate accurate answer. <br>

employee-handbook.pdf <br>
You ask the LLM: <br>
```
“How many casual leaves per year do employees get?”
```
**Without RAG** → LLM hallucinates. <br>
**With RAG** → LLM pulls the answer from YOUR PDF. <br>
Provides answers from your data.

Neural Network:
---------
- Computer system inspred by humna brain.
- It recognize patterns and make decissions from data.

CNNs ( Convolutional Neural Netowkrs)
------------------------------
- Best for images, videos, Visual pattern recognition
- Automatically detect features like edges, shapes and textures.

Diffusion models:
--------------
- Image generation, Audio synthesis, denoising, creative generation tasks

Transformers ( Non LLM Context )
-----------------
- Sequential data like text, time series, image patches.
Use case :
- Time series forecasting ( Predict stock prices or sensor reading )
- Audio processing - speach recognition
- Protein folding

Weights:
-----
- learned values

Model Optimizations ( TensorRT Model Optimizer )
---------------------
1. Quantization -
 - Shrink the numbers
 - AI Model uses 32-bit floating numbers to store weights ( Their learned values )
 - Reduce that to 16,8,4-bit integrers
 - Make the model smaller in memory.

2. Distillation
   - Teach smaller students.
   - make smaller model almost same as big.
  
3. Sparsity
  - Remove unneccesary connections
  - turning of or removing connections that dont matter much.
  - Make model faster

4. Pruning
- Removing parts of neaural network that dont contribute much to the final result.

5. Speculative decoding:
- Guess now , check later.

Ollama:
------
- Model runtime manager for LLM
- Pull -> Manage -> run models
- Handles - Model download, GPU/CPU runtime setup, Chat/Inference Interface, API server for integration

TensorRT
------
- NVIDIA's high performance inference optimizer and runtime
- run models much faster on GPU

ONNX(Open neural network exchange)
---------------------
- its like universal file format for AI models
- So models trained in one framework (pytorchm, tensorflow) can be used anywhere else without retraining.

Optimizing GPU execution using:
- FP16
- Layer Fusion
- Kernel Auto-tuning

TensorRT-LLM
-------------
- Library  for optimizing LLM inference on NVIDIA GPU.
- built on the top of TensorRT Stack, but specialized for LLM


Safetensor
------------
- File format for storing model weights like .bin or .pt
- designed to : safe, Fast and portable.
- Older formats are (.bin, .pt) are pickle-based, meaning they can execute arbirary pytohn code when loaded. which creates security risk.

- Pytorch model save weights like .safetensorts.
- Its just data not executable code.
- like zip file which stores only data.

  Safetensor -> TensorRT-LLM <br>
  1. Get a model
  2. Convert using TensorRT-LLM ( trtllm-build )
  3. Run it with TensorRT-LLM ( trtllm-serv ./enginge)

 This is not production grade, this is for testing only. <br>

 Realworld example: <br>
 1. Download model from huggingface. ( model.safetensor, config.json, tokenizer.json)
 2. Convert to tensorRT-LLM (trtllm build)
 3. Run model (trtllm-serv)
 4. Now you can query model
