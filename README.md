# On-Device LLM Deployment for Android

A mobile app that runs Large Language Models directly on Android devices. Built it to demonstrate mobile ML optimization and deployment skills.

## What it does

This Android app loads quantized LLM models (like Qwen2.5-1.5B) and runs them locally on your phone. No internet required - everything happens on-device. The app includes performance monitoring to track inference speed, memory usage, and response quality.

## Demo

| Q4 Model Demo | Q3 Model Demo |
|---|---|
| <video width="400" height="800" controls><source src="demos/Q4_K_M.mp4" type="video/mp4">Your browser does not support the video tag.</video> | <video width="400" height="800" controls><source src="demos/Q3_K_M.mp4" type="video/mp4">Your browser does not support the video tag.</video> |
| *Q4 quantized model - Higher quality responses (2.21 TPS)* | *Q3 quantized model - Faster loading, lower memory (2.13 TPS)* |

---------

## Performance Results

### Model Comparison

| Model | Size | TTFT | Tokens | TPS | Peak Memory | Avg Memory | Response Length |
|-------|------|------|--------|-----|-------------|------------|-----------------|
| Qwen2.5-1.5B Q4 | 986MB | 18,791ms | 236 | 2.21 | 2,428MB | 2,097MB | 1,172 chars |
| Qwen2.5-1.5B Q3 | 824MB | 18,536ms | 93 | 2.13 | 2,119MB | 2,040MB | 492 chars |

- TTFT: timestamp of first emitted token minus request start.
- TPS: tokens count generated divided by total time taken.
Sampling settings: temperature = 0.8, top_p = 0.9, min_p = 0.05, repeat_penalty = 1.1.

### Observations
- **Quality vs Efficiency**: Q4 generated 2.5x more tokens (236 vs 93) with significantly longer, more detailed responses
- **Performance**: Similar TPS (~2.1-2.2) and TTFT (~18.5-18.8s) despite different response lengths
- **Memory**: Q4 uses ~15% more memory (309MB difference) for substantially more content  
- **Model Size**: Q3 is ~16% smaller (162MB difference) than Q4

### Device Specifications
- **Device**: Redmi Note 7 (RAM: 4GB)
- **Prompt Length**: 48 characters

## How it works

The project has three main parts:

1. **Python quantization script** - Downloads models from HuggingFace, converts to GGUF format, and quantizes them
2. **Android app** - Kotlin/Compose UI with JNI bindings to llama.cpp  
3. **Performance monitoring** - Logs inference metrics to CSV files

### Technical details

- Uses llama.cpp for the inference engine
- ChatML format for conversation handling
- Real-time memory monitoring during inference
- Configurable context window (512-4096 tokens)
- Automatic benchmark logging.

## Building and running

### Prerequisites
- Python 3.10 
- cmake 3.28.3 
- gcc 13.3.0
- openjdk 21.0.8
- Android Studio with NDK
- Android device with USB debugging

### Model preparation
#### Steps:
1. **Clone this repository**
   ```bash
   git clone [URL will be inserted later]
   ```

2. **Build llama.cpp**
   ```bash
   cd llm_edge_deployment/llama.cpp
   cmake -B build && cmake --build build --config Release
   ```

3. **Verify required binaries exist:**
   - `build/bin/llama-cli`
   - `build/bin/llama-quantize`

4. **Install Python dependencies**
   ```bash
   cd ../
   pip install -r requirements.txt
   ```

5. **Run quantization script**
   ```bash
   python quantize.py
   ```
This downloads Qwen2.5-1.5B from HuggingFace, converts to GGUF, and quantizes to Q4_K_M format. The quantized model is automatically placed in the Android app's assets folder at llama.cpp/examples/llama.android/app/src/main/assets/models/.

Note: Process requires ~3GB temporary disk space and takes 5-10 minutes depending on internet speed.

### Android build
```bash
cd llama.cpp/examples/llama.android
./gradlew clean installDebug
```
This builds the APK with the quantized model embedded and installs it on your connected device. The model files are automatically copied from APK assets to device storage on first run. 

Build time: ~3-5 minutes depending on hardware.

### Monitor performance of your phone during inferences:
```bash
# Watch logs
adb logcat -s MainViewModel LLamaAndroid llama-android.cpp

# Pull benchmark data
adb pull /sdcard/Android/data/com.example.llama/files/llm_benchmark.log ./
```

## Architecture

```
1. Model Prep: HF → llama.cpp convert → GGUF → llama.cpp quantize → Optimized GGUF

2. Android Build: Kotlin App + JNI Bridge + llama.cpp (embedded) → APK

3. Deployment: Gradle build → APK installation (Takes ~3 minutes)

4. Runtime: Model loading → llama.cpp inference + Performance monitoring → Results
```

Key optimizations:
- Conversation context caching
- Real-time performance monitoring
- Automatic stop token detection for Qwen models

## What I learned

Working on this project taught me:
- Mobile ML deployment challenges (memory constraints)
- Model quantization trade-offs between size and quality
- Real-time performance monitoring

The biggest challenge was memory management - keeping everything within mobile device limits while maintaining reasonable inference speed.

## Files overview

- `quantize.py` - Model download and quantization pipeline
- `MainActivity.kt` - Main Android activity and UI setup
- `MainViewModel.kt` - Android app logic and inference orchestration  
- `ChatTemplates.kt` - Chat format templates and conversation handling
- `llama-android.cpp` - JNI bridge to llama.cpp inference engine
- `BenchmarkLogger.kt` - Performance monitoring and CSV logging
- `MemoryMonitor.kt` - Real-time memory usage tracking

## Additional Resources

- https://github.com/ggml-org/llama.cpp
- https://huggingface.co/Qwen/Qwen2.5-1.5B-Instruct
- https://developer.android.com/studio
- https://docs.gradle.org/current/userguide/command_line_interface.html


Built as a portfolio project to demonstrate mobile ML engineering skills.

