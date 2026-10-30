# Ollama Local AI: Portable Local Coding Environment

A technical documentation repository covering the configuration of a locally hosted AI coding environment using Ollama, Qwen2.5-Coder and external USB storage.

The project focuses on running local language models without relying on paid API services, managing hardware resource consumption and establishing a portable model-storage configuration.

**Project Status:** Local inference operational. Resource-conscious model configuration created. GPU troubleshooting, monitoring dashboard and Qwen Code integration pending.

---

## 1. Project Objectives

The primary objectives are:

* Configure Ollama for local model inference.
* Migrate model storage to external USB storage.
* Install and configure a local coding model.
* Establish resource-conscious inference settings.
* Investigate CPU and GPU resource management.
* Develop an independent hardware monitoring dashboard.
* Integrate Qwen Code with the local Ollama instance.
* Establish a reproducible local AI development environment.

## 2. Hardware Specifications

| Component          | Specification                  |
| ------------------ | ------------------------------ |
| Laptop             | HP Victus                      |
| Processor          | AMD Ryzen 5 5600H              |
| CPU Cores          | 6                              |
| Logical Processors | 12                             |
| Dedicated GPU      | NVIDIA GeForce RTX 3050 Laptop |
| GPU VRAM           | 4 GB                           |
| Integrated GPU     | AMD Radeon Graphics            |
| System RAM         | 16 GB (15.3 GB usable)         |
| External Storage   | 64 GB USB Drive                |

### Software Environment

| Software         | Version    |
| ---------------- | ---------- |
| Operating System | Windows 11 |
| Ollama           | 0.40.13    |
| Qwen Code        | 0.26.4     |
| Node.js          | v24.21.0   |
| npm              | 11.19.0    |
| Python           | 3.13.14    |

---

## 3. External Storage Configuration

### 3.1 Motivation

The initial configuration encountered storage limitations while downloading local language models.

The external USB drive was selected to separate model storage from the laptop's internal storage.

### 3.2 Filesystem Configuration

The USB drive was formatted using the exFAT filesystem.

| Property                         | Configuration       |
| -------------------------------- | ------------------- |
| Drive Letter                     | E:                  |
| Capacity                         | Approximately 64 GB |
| Filesystem                       | exFAT               |
| Available Space After Formatting | 57.72 GB            |
| Ollama Model Directory           | `E:\OllamaModels`   |

**Filesystem consideration:** FAT32 does not support individual files larger than 4 GB. exFAT supports larger files and is therefore suitable for storing large model files.

### 3.3 Directory Creation

```powershell
New-Item -ItemType Directory -Force "E:\OllamaModels"
```

### 3.4 Ollama Storage Configuration

The `OLLAMA_MODELS` environment variable was configured at the Windows User level.

```powershell
[Environment]::SetEnvironmentVariable(
    "OLLAMA_MODELS",
    "E:\OllamaModels",
    "User"
)
```

### 3.5 Configuration Verification

```powershell
[Environment]::GetEnvironmentVariable("OLLAMA_MODELS","User")
```

Expected output:

```text
E:\OllamaModels
```

The current PowerShell environment was also verified:

```powershell
echo $env:OLLAMA_MODELS
```

After configuring the environment variable, Ollama was restarted to ensure that its background process could inherit the updated configuration.

**Operational requirement:** The external USB drive must remain connected whenever Ollama accesses models stored on it. Models should be stopped before safely ejecting the drive.

---

## 4. Local Coding Model Installation

### 4.1 Selected Model

**Qwen2.5-Coder 3B**

The model was selected as a relatively lightweight coding model suitable for local inference on the available hardware.

### 4.2 Installation

```powershell
ollama pull qwen2.5-coder:3b
```

### 4.3 Installation Results

| Property            | Value              |
| ------------------- | ------------------ |
| Model               | `qwen2.5-coder:3b` |
| Model ID            | `f72c60cabf62`     |
| Reported Disk Size  | 1.9 GB             |
| Installation Status | Successful         |

Verify the installation:

```powershell
ollama list
```

Run the model:

```powershell
ollama run qwen2.5-coder:3b
```

### 4.4 Initial Inference Test

The model was evaluated using a basic Three.js code-generation task.

**Test Prompt:**

```text
Write a simple Three.js scene with a pink rotating cube.
Explain the code.
```

The model successfully generated:

* HTML document structure.
* Three.js scene initialization.
* Perspective camera configuration.
* WebGL renderer initialization.
* Cube geometry and material.
* Animation loop.
* Code explanation.

**Result:** Local inference and basic code generation were successfully verified.

---

## 5. Resource-Conscious Model Configuration

A separate Ollama model configuration was created to apply inference parameters without modifying the original model.

**Configured Model:** `qwen-coder-safe`

### 5.1 Modelfile

```text
FROM qwen2.5-coder:3b

PARAMETER num_ctx 8192
PARAMETER num_thread 4
PARAMETER num_predict 512
```

### 5.2 Model Creation

```powershell
@'
FROM qwen2.5-coder:3b
PARAMETER num_ctx 8192
PARAMETER num_thread 4
PARAMETER num_predict 512
'@ | Set-Content "$env:TEMP\Modelfile-qwen-safe"

ollama create qwen-coder-safe -f "$env:TEMP\Modelfile-qwen-safe"
```

### 5.3 Configuration Parameters

| Parameter     | Value | Description                         |
| ------------- | ----: | ----------------------------------- |
| `num_ctx`     |  8192 | Configures the context window       |
| `num_thread`  |     4 | Configures CPU inference threads    |
| `num_predict` |   512 | Limits the maximum generated tokens |

Run the configured model:

```powershell
ollama run qwen-coder-safe
```

The original `qwen2.5-coder:3b` model remains unchanged.

**Configuration limitation:** These parameters do not establish a strict CPU utilization percentage or GPU usage cap. They provide inference-level controls but do not guarantee specific hardware temperatures or utilization levels.

---

## 6. Ollama Server-Level Configuration

The following Windows User environment variables were configured:

```powershell
[Environment]::SetEnvironmentVariable(
    "OLLAMA_NUM_PARALLEL","1","User"
)

[Environment]::SetEnvironmentVariable(
    "OLLAMA_MAX_LOADED_MODELS","1","User"
)

[Environment]::SetEnvironmentVariable(
    "OLLAMA_KEEP_ALIVE","5m","User"
)
```

### 6.1 Configuration Description

| Variable                   | Value | Purpose                              |
| -------------------------- | ----- | ------------------------------------ |
| `OLLAMA_NUM_PARALLEL`      | 1     | Configures parallel request handling |
| `OLLAMA_MAX_LOADED_MODELS` | 1     | Limits simultaneously loaded models  |
| `OLLAMA_KEEP_ALIVE`        | 5m    | Configures idle model retention      |

These are server-level settings and are separate from the model-specific Modelfile parameters.

**Verification status:** The environment variables have been configured. Restart and runtime verification of the Ollama background process remain pending.

---

## 7. Runtime Monitoring

Ollama provides a command for inspecting currently loaded models.

```powershell
ollama ps
```

### 7.1 Observed Runtime Information

```text
NAME                      SIZE      PROCESSOR
qwen-coder-safe:latest    2.4 GB    100% CPU
```

Additional runtime information:

| Property             | Observed Value    |
| -------------------- | ----------------- |
| Model                | `qwen-coder-safe` |
| Context Window       | 8192              |
| Processor Allocation | 100% CPU          |
| Runner               | llamacpp          |

**Interpretation:** The model was running entirely on the CPU at the time of inspection.

The `100% CPU` value displayed by `ollama ps` describes processor allocation, not necessarily total CPU utilization.

The reported runtime size is distinct from the model's disk size.

---

## 8. NVIDIA GPU Troubleshooting

### 8.1 Hardware Detection

The laptop contains an NVIDIA GeForce RTX 3050 Laptop GPU with 4 GB of dedicated VRAM.

Initial GPU monitoring was attempted using:

```powershell
nvidia-smi
```

The command initially failed because NVIDIA-SMI could not communicate with the NVIDIA driver.

### 8.2 Device Detection

The following command was used:

```powershell
Get-PnpDevice -Class Display |
Format-Table Status, FriendlyName, InstanceId -AutoSize
```

Observed result:

```text
OK     AMD Radeon(TM) Graphics
Error  NVIDIA GeForce RTX 3050 Laptop GPU
```

### 8.3 Device Status Investigation

Further diagnostics were performed:

```powershell
Get-PnpDevice -Class Display |
Where-Object {$_.FriendlyName -like "*NVIDIA*"} |
Format-List Status, FriendlyName, Problem, ConfigManagerErrorCode
```

Observed result:

```text
Status                 : Error
FriendlyName           : NVIDIA GeForce RTX 3050 Laptop GPU
Problem                : CM_PROB_DISABLED
ConfigManagerErrorCode : CM_PROB_DISABLED
```

The NVIDIA Display Container service was found to be running.

Driver information:

```text
DriverVersion : 32.0.15.9227
DriverDate    : 01-04-2026
Status        : Error
```

### 8.4 Current Assessment

| Component                | Status                            |
| ------------------------ | --------------------------------- |
| AMD Integrated Graphics  | Operational                       |
| NVIDIA RTX 3050          | Reported disabled by Windows      |
| NVIDIA Display Container | Running                           |
| NVIDIA-SMI               | Unable to communicate with driver |
| GPU Telemetry            | Pending                           |
| GPU Acceleration         | Not verified                      |

### 8.5 Pending Troubleshooting

The next diagnostic step is to inspect the NVIDIA device in Windows Device Manager.

1. Open Device Manager.
2. Expand Display adapters.
3. Locate NVIDIA GeForce RTX 3050 Laptop GPU.
4. Inspect its device status.
5. If appropriate, enable the device.
6. Recheck NVIDIA-SMI.

```powershell
nvidia-smi
```

The current device status does not establish hardware failure. Further investigation is required.

---

## 9. Planned Development: Local AI Resource Guardian

An independent hardware monitoring dashboard is planned to monitor system resources while Ollama is running.

### 9.1 Proposed Technology Stack

| Technology           | Purpose                          |
| -------------------- | -------------------------------- |
| Python               | Monitoring and control logic     |
| Streamlit            | Dashboard interface              |
| psutil               | CPU and RAM monitoring           |
| NVIDIA-SMI           | NVIDIA GPU telemetry             |
| LibreHardwareMonitor | Hardware temperature monitoring  |
| Ollama API           | Model status and process control |

### 9.2 Planned Monitoring Metrics

* CPU utilization.
* CPU temperature.
* GPU utilization.
* GPU temperature.
* GPU VRAM consumption.
* System RAM consumption.
* Active Ollama model.
* Model running or idle status.

### 9.3 Proposed Alert Classification

| Level   | Behaviour                                     |
| ------- | --------------------------------------------- |
| SAFE    | Normal operating conditions                   |
| CAUTION | Elevated temperatures or resource consumption |
| DANGER  | Critical temperature or resource conditions   |

Thresholds will be configurable and selected according to hardware specifications.

GPU utilization alone will not determine the danger level. Temperature, memory consumption and other available hardware telemetry will also be considered.

### 9.4 Planned Controls

* Start and pause monitoring.
* Display hardware warnings.
* Stop the active Ollama model.
* Provide an emergency stop control.
* Configure temperature thresholds.
* Optionally terminate the model after a sustained critical-temperature condition.

The monitoring application will operate independently of the language model to avoid relying on the model to monitor its own resource consumption.

**Implementation status:** Planned.

---

## 10. Planned Qwen Code Integration

The next integration objective is to connect Qwen Code to the locally hosted Ollama model.

### Intended Architecture

```text
                  USER
                    |
                    v
                QWEN CODE
                    |
                    v
                  OLLAMA
                    |
                    v
             Qwen2.5-Coder 3B
                    |
                    v
             LOCAL PROJECT FILES
```

The intended coding-agent capabilities include:

* Reading project files.
* Editing multiple files.
* Generating code.
* Debugging errors.
* Executing development commands.
* Assisting with Three.js development.

The integration will use a local model provider rather than a paid OpenAI API.

**Integration status:** Pending.

---

## 11. Overall Implementation Status

| Component                              | Status                                   |
| -------------------------------------- | ---------------------------------------- |
| USB formatting                         | Complete                                 |
| exFAT configuration                    | Complete                                 |
| Ollama model directory                 | Configured                               |
| Environment variable                   | Configured                               |
| Qwen2.5-Coder 3B installation          | Complete                                 |
| Local inference test                   | Successful                               |
| Resource-conscious model configuration | Created                                  |
| CPU inference thread configuration     | Configured                               |
| Context window configuration           | Configured                               |
| Generation token limit                 | Configured                               |
| Server concurrency settings            | Configured; runtime verification pending |
| NVIDIA GPU troubleshooting             | Pending                                  |
| GPU telemetry                          | Pending                                  |
| Resource Guardian dashboard            | Planned                                  |
| Qwen Code integration                  | Pending                                  |

---

## 12. Operational Precautions

1. Keep the USB drive connected while using Ollama.
2. Do not disconnect the drive while a model is running.
3. Stop the model before safely ejecting the USB drive.

```powershell
ollama stop qwen-coder-safe
```

4. Inspect active models:

```powershell
ollama ps
```

5. Inspect installed models:

```powershell
ollama list
```

6. Monitor available USB storage.
7. Do not interpret CPU thread limits as GPU utilization limits.
8. Do not force GPU allocation until NVIDIA driver functionality has been verified.
9. Verify server-level environment settings after restarting Ollama.

---

## 13. Future Development Roadmap

The long-term objective is to establish a self-monitoring local AI coding environment.

Planned development stages:

| Stage | Objective                                           |
| ----- | --------------------------------------------------- |
| 1     | Complete and verify Ollama storage migration        |
| 2     | Finalize resource-conscious inference configuration |
| 3     | Resolve NVIDIA GPU detection                        |
| 4     | Implement the Resource Guardian dashboard           |
| 5     | Integrate Qwen Code with Ollama                     |
| 6     | Test the complete local coding-agent workflow       |

The intended outcome is a portable local AI development environment that provides coding assistance while maintaining visibility into hardware resource consumption.

---

## 14. Project Summary

The Ollama model-storage migration to an external exFAT USB drive has been configured successfully.

Qwen2.5-Coder 3B has been downloaded and tested through local inference.

A separate resource-conscious model configuration has been created, along with preliminary server-level resource settings.

The remaining work involves verifying persistent server settings, resolving NVIDIA GPU detection, implementing independent hardware monitoring and completing Qwen Code integration.

**Current milestone:** Local inference operational; resource monitoring and coding-agent integration pending.

---

*This repository serves as a technical record of the implementation, configuration decisions, troubleshooting procedures and future development of the local AI environment.*
