---
layout: post
title: "Limit GPU memory growth in tensorflow 2.4.x by setting environment variable"
description: "fix memory problem while fitting a model"
date: 2021-03-11
tags: [ML, tensorflow]
categories: [ML, tensorflow]
comments: true
---

## Simplest way(TensorFlow 2.2+)
```python
import tensorflow as tf
gpus = tf.config.experimental.list_physical_devices('GPU')
for gpu in gpus:
  tf.config.experimental.set_memory_growth(gpu, True)
```

## Or set environment variable
set `TF_FORCE_GPU_ALLOW_GROWTH` to `true`.

## if TensorFlow 2.0 and 2.1
```python
import tensorflow as tf
tf.config.gpu.set_per_process_memory_growth(True)
```

Source:  
1. https://www.tensorflow.org/guide/gpu#limiting_gpu_memory_growth 
2. https://stackoverflow.com/questions/34199233/how-to-prevent-tensorflow-from-allocating-the-totality-of-a-gpu-memory/34200194#34200194 
