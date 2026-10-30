# Ollama Local AI: Portable Local Coding Environment

A technical documentation repository covering the configuration of a locally hosted AI coding environment using Ollama, Qwen2.5-Coder, Qwen Code and external USB storage.

The project focuses on running local language models without relying on paid API services, managing hardware resource consumption and establishing a portable model-storage configuration.

**Project Status:** Local inference operational. Qwen Code successfully integrated with Ollama. NVIDIA GPU monitoring operational. Resource Guardian implemented and tested.

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
* Provide visibility into system resource consumption during local inference.

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

| Software             | Version                       |
| -------------------- | ----------------------------- |
| Operating System     | Windows 11                    |
| Ollama               | 0.40.13                       |
| Qwen Code            | 0.26.4                        |
| Node.js              | v24.21.0                      |
| npm                  | 11.19.0                       |
| Python               | 3.13.14                       |
| .NET SDK             | 10.0.103                      |
| Streamlit            | 1.66.1                        |
| psutil               | 7.2.2                         |
| requests             | 2.34.2                        |
| pandas               | 3.1.0                         |
| LibreHardwareMonitor | Source build, net10.0-windows |

---

## 3. External Storage Configuration

### 3.1 Motivation

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

| Parameter     | Value | Description                      |
| ------------- | ----: | -------------------------------- |
| `num_ctx`     |  8192 | Configures the context window    |
| `num_thread`  |     4 | Configures CPU inference threads |
| `num_predict` |   512 | Limits maximum generated tokens  |

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

These are server-level settings and are separate from model-specific Modelfile parameters.

**Status:** Environment variables configured. Ollama model loading and unloading were subsequently verified through runtime testing.

---

## 7. Runtime Monitoring

Ollama provides commands for inspecting installed and currently loaded models.

### 7.1 Installed Models

```powershell
ollama list
```

### 7.2 Currently Loaded Models

```powershell
ollama ps
```

### 7.3 Observed Runtime Information

```text
NAME                      ID              SIZE      PROCESSOR    CONTEXT    RUNNER
qwen-coder-safe:latest    afebe1c77752    2.4 GB    100% CPU     8192       llamacpp
```

| Property             | Observed Value           |
| -------------------- | ------------------------ |
| Model                | `qwen-coder-safe:latest` |
| Model ID             | `afebe1c77752`           |
| Runtime Size         | 2.4 GB                   |
| Processor Allocation | 100% CPU                 |
| Context Window       | 8192                     |
| Runner               | llamacpp                 |

**Interpretation:** The model was running entirely on the CPU at the time of inspection.

The `100% CPU` value displayed by `ollama ps` describes processor allocation, not total CPU utilization.

The reported runtime size is distinct from the model's disk size.

---

## 8. NVIDIA GPU Troubleshooting and Recovery

### 8.1 Initial Hardware Detection

The laptop contains an NVIDIA GeForce RTX 3050 Laptop GPU with 4 GB of dedicated VRAM.

Initial GPU monitoring was attempted using:

```powershell
nvidia-smi
```

The command initially failed because NVIDIA-SMI could not communicate with the NVIDIA driver.

### 8.2 Device Detection

```powershell
Get-PnpDevice -Class Display |
Format-Table Status, FriendlyName, InstanceId -AutoSize
```

The NVIDIA GPU was initially reported with an error status.

### 8.3 Device Status Investigation

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

### 8.4 GPU Recovery

The NVIDIA RTX 3050 was found to be disabled in Windows Device Manager.

The device was enabled through Device Manager.

Subsequent verification:

```powershell
Get-PnpDevice -Class Display
```

Both graphics devices were then reported with an OK status:

* AMD Radeon Graphics
* NVIDIA GeForce RTX 3050 Laptop GPU

### 8.5 NVIDIA-SMI Verification

NVIDIA-SMI subsequently worked from an elevated PowerShell session.

Observed environment:

| Property              | Value           |
| --------------------- | --------------- |
| NVIDIA-SMI            | 592.27          |
| NVIDIA Driver         | 592.27          |
| CUDA Version Reported | 13.1            |
| GPU                   | RTX 3050 Laptop |
| Dedicated VRAM        | 4 GB            |

**Result:** NVIDIA driver communication was restored and GPU telemetry became available.

GPU availability does not imply that Ollama inference is using the GPU. The tested Qwen model continued to use CPU allocation.

---

## 9. Hardware Monitoring with LibreHardwareMonitor

### 9.1 Motivation

The monitoring application required hardware telemetry beyond basic CPU and RAM utilization.

LibreHardwareMonitor was selected to expose sensor information, including temperatures, power consumption and GPU memory usage.

### 9.2 Source Build

The LibreHardwareMonitor source repository was downloaded and built locally using the .NET SDK.

Build command:

```powershell
dotnet build .\LibreHardwareMonitor.Windows.Forms\LibreHardwareMonitor.Windows.Forms.csproj -c Release -p:Platform=x64 -p:PlatformTarget=x64
```

The resulting Windows Forms application was available at:

```text
C:\Users\Hp\LibreHardwareMonitor-master\LibreHardwareMonitor-master\bin\Release\net10.0-windows\LibreHardwareMonitor.Windows.Forms.exe
```

### 9.3 Remote Web Server

LibreHardwareMonitor's Remote Web Server feature was enabled.

| Property         | Configuration |
| ---------------- | ------------- |
| Host             | `127.0.0.1`   |
| Port             | `8085`        |
| Endpoint         | `/data.json`  |
| Authentication   | Disabled      |
| Network Exposure | Loopback only |

The sensor endpoint:

```text
http://127.0.0.1:8085/data.json
```

Authentication was disabled because the local browser request was receiving an HTTP 401 response without a usable authentication challenge.

**Security consideration:** The server remains bound to `127.0.0.1`. It must not be exposed to the local network or public internet without appropriate authentication and security controls.

### 9.4 Sensor Verification

The sensor tree was inspected through PowerShell:

```powershell
$data = Invoke-RestMethod "http://127.0.0.1:8085/data.json"

function Show-Tree($node, $level = 0) {
    $indent = " " * ($level * 2)

    if ($node.Text) {
        Write-Host "$indent$($node.Text) $($node.Value)"
    }

    foreach ($child in $node.Children) {
        Show-Tree $child ($level + 1)
    }
}

Show-Tree $data
```

The following hardware categories were identified:

* AMD Ryzen 5 5600H CPU sensors.
* NVIDIA RTX 3050 GPU sensors.
* AMD integrated GPU sensors.
* Physical memory information.
* SSD temperature and health.
* USB storage information.
* Network hardware information.

### 9.5 Example Sensor Readings

One recorded snapshot showed:

| Metric             | Observed Value |
| ------------------ | -------------: |
| CPU Temperature    |        71.6 °C |
| CPU Utilization    |          35.5% |
| CPU Package Power  |         16.6 W |
| Physical RAM Usage |          92.7% |
| RAM Available      |        1.12 GB |
| GPU Temperature    |          54 °C |
| GPU Hotspot        |          62 °C |
| GPU Utilization    |             0% |
| GPU VRAM Used      |         134 MB |
| GPU VRAM Total     |           4 GB |
| SSD Temperature    |          50 °C |
| SSD Health         |            97% |

These values represent one historical snapshot, not fixed or guaranteed operating values.

---

## 10. Resource Guardian — Hardware Monitoring Dashboard

### 10.1 Overview

Resource Guardian is an independent Python-based monitoring application developed to observe hardware resource consumption while using local AI models.

It operates separately from Qwen Code and the Ollama inference process.

The dashboard retrieves hardware telemetry from LibreHardwareMonitor and uses psutil for system-level resource information.

### 10.2 Technology Stack

| Technology           | Purpose                                   |
| -------------------- | ----------------------------------------- |
| Python               | Monitoring and control logic              |
| Streamlit            | Dashboard interface                       |
| psutil               | CPU and physical RAM monitoring           |
| requests             | Local HTTP communication                  |
| pandas               | Resource history data                     |
| LibreHardwareMonitor | Hardware temperature and sensor telemetry |
| Ollama API           | Model status and unloading                |

### 10.3 Project Structure

```text
ResourceGuardian/
│
├── .venv/
│
├── app.py
│
└── hardware.py
```

The Python virtual environment contains the project's installed dependencies.

### 10.4 Environment Setup

Create the project directory:

```powershell
mkdir "$HOME\ResourceGuardian"
cd "$HOME\ResourceGuardian"
```

Create a virtual environment:

```powershell
python -m venv .venv
```

Activate it:

```powershell
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
python -m pip install --upgrade pip

pip install streamlit psutil requests pandas
```

### 10.5 Hardware Monitoring Module

The `hardware.py` module handles hardware sensor collection.

Its primary responsibilities include:

* Retrieving the LibreHardwareMonitor JSON sensor tree.
* Locating hardware devices by name.
* Finding individual sensor values.
* Converting sensor strings into numerical values.
* Collecting a consolidated hardware snapshot.
* Reading physical memory usage through psutil.

The module exposes:

```python
get_snapshot()
```

This function returns a dictionary containing timestamped CPU, RAM, GPU and SSD information.

If LibreHardwareMonitor is unavailable, the function returns an error rather than attempting to fabricate sensor readings.

### 10.6 Dashboard Implementation

The `app.py` module implements the Streamlit dashboard.

Implemented features:

* Live CPU temperature monitoring.
* CPU utilization monitoring.
* CPU package power monitoring.
* Physical RAM usage and availability.
* NVIDIA GPU temperature.
* GPU hotspot temperature.
* GPU utilization.
* GPU VRAM consumption.
* SSD temperature.
* SSD health percentage.
* Resource history chart.
* Ollama model status detection.
* Manual model unloading.

### 10.7 Dashboard Refresh

The dashboard uses Streamlit's fragment functionality:

```python
@st.fragment(run_every="3s")
```

This refreshes the monitoring section approximately every three seconds without requiring the entire application to be rerun.

Resource history is maintained in Streamlit session state and limited to the latest 60 observations.

### 10.8 Status Classification

The initial dashboard uses configurable rule-based thresholds.

| Resource        | CAUTION |  DANGER |
| --------------- | ------: | ------: |
| RAM Usage       |   ≥ 85% |   ≥ 95% |
| CPU Temperature | ≥ 80 °C | ≥ 90 °C |
| GPU Temperature | ≥ 80 °C | ≥ 90 °C |

The highest detected severity determines the displayed system status.

These thresholds are initial monitoring rules, not manufacturer-certified safety limits. They should be adjusted according to hardware specifications and operating conditions.

The current implementation does not automatically shut down the laptop or terminate arbitrary applications.

### 10.9 Dashboard Launch

From the activated virtual environment:

```powershell
python -m streamlit run app.py --server.address 127.0.0.1
```

The dashboard is available locally at:

```text
http://127.0.0.1:8501
```

### 10.10 LibreHardwareMonitor Dependency

Resource Guardian depends on the LibreHardwareMonitor Web Server being active.

If the sensor server is unavailable, hardware readings cannot be retrieved.

Start LibreHardwareMonitor before launching the dashboard.

---

## 11. Ollama Integration with Resource Guardian

### 11.1 Model Detection

Resource Guardian communicates with Ollama through its local API.

The model status endpoint is:

```text
http://127.0.0.1:11434/api/ps
```

The dashboard checks whether `qwen-coder-safe` is currently loaded.

### 11.2 Model Unloading

The application provides a manual unload button.

The unload request uses the Ollama generate API:

```python
requests.post(
    "http://127.0.0.1:11434/api/generate",
    json={
        "model": "qwen-coder-safe",
        "prompt": "",
        "keep_alive": 0,
        "stream": False
    },
    timeout=15
)
```

Setting `keep_alive` to zero requests immediate unloading of the model after the request.

### 11.3 Verification Procedure

Load the model:

```powershell
ollama run qwen-coder-safe
```

Submit a prompt to ensure that inference occurs.

Inspect the running model:

```powershell
ollama ps
```

Return to Resource Guardian and click:

**Unload Ollama Model**

Verify the result:

```powershell
ollama ps
```

Expected result when unloading succeeds:

```text
NAME    ID    SIZE    PROCESSOR    CONTEXT    RUNNER    UNTIL
```

### 11.4 Test Results

| Test                     | Result     |
| ------------------------ | ---------- |
| Ollama server detection  | Successful |
| Loaded model detection   | Successful |
| Model status display     | Successful |
| Manual unload request    | Successful |
| Post-unload verification | Successful |

**Implementation status:** Model detection and manual unloading are implemented and verified.

**Limitation:** The current dashboard does not automatically unload models based on temperature or RAM thresholds. Model unloading requires user action.

---

## 12. Qwen Code Integration

### 12.1 Objective

The objective was to connect Qwen Code to the locally hosted Ollama model instead of relying on a paid API provider.

### 12.2 Software Versions

| Component | Version  |
| --------- | -------- |
| Qwen Code | 0.26.4   |
| Ollama    | 0.40.13  |
| Node.js   | v24.21.0 |
| npm       | 11.19.0  |

### 12.3 Local Model Provider

Qwen Code was configured to use the local Ollama OpenAI-compatible API endpoint.

```text
http://127.0.0.1:11434/v1
```

The explicit IPv4 address is intentional.

During troubleshooting, two local Ollama server endpoints were identified:

* `127.0.0.1:11434`
* `[::1]:11434`

They exposed different model registries during the investigation.

The configuration therefore uses the explicit IPv4 endpoint to avoid accidentally connecting to the other local server instance.

### 12.4 Model Configuration

The local model used for coding assistance:

```text
qwen-coder-safe:latest
```

The Qwen Code settings were configured through:

```text
C:\Users\Hp\.qwen\settings.json
```

An existing settings backup was preserved at:

```text
C:\Users\Hp\.qwen\settings.backup.json
```

The custom provider configuration was tested by launching Qwen Code and verifying local model inference.

### 12.5 Integration Verification

| Test                                    | Result       |
| --------------------------------------- | ------------ |
| Qwen Code installation                  | Successful   |
| Local Ollama provider configuration     | Successful   |
| IPv4 endpoint connectivity              | Successful   |
| Local model inference through Qwen Code | Successful   |
| Paid API dependency                     | Not required |

**Result:** Qwen Code successfully communicates with the local Ollama model.

---

## 13. Resource Management and System Optimization

### 13.1 OpenClaw Background Process

During resource investigation, an automatically running OpenClaw gateway was identified.

The gateway was not required for the current local AI workflow.

It was stopped using:

```powershell
openclaw gateway stop
```

The Windows login startup entry was also disabled.

The installation and configuration were preserved.

### 13.2 RAM Observations

Before stopping the background gateway, available physical memory was approximately 0.36 GB.

After stopping it, available memory increased to approximately 2.3–2.4 GB during subsequent observations.

Memory availability continued to vary depending on active processes and system activity.

### 13.3 Pagefile

The Windows pagefile was observed at:

```text
D:\pagefile.sys
```

Its allocated size was approximately 24.6 GB.

The pagefile was left unchanged.

### 13.4 Resource Considerations

The laptop has 16 GB of physical RAM, which remains a significant constraint when running several development applications simultaneously.

Recommended operating practices:

* Avoid keeping unnecessary applications open.
* Unload Ollama models when they are not required.
* Avoid running multiple large local models simultaneously.
* Keep the Resource Guardian dashboard lightweight.
* Monitor physical RAM availability during inference.
* Keep the external model-storage drive connected.

---

## 14. Operational Commands

### Ollama

List installed models:

```powershell
ollama list
```

Inspect loaded models:

```powershell
ollama ps
```

Run the coding model:

```powershell
ollama run qwen-coder-safe
```

Stop the coding model:

```powershell
ollama stop qwen-coder-safe
```

### Resource Guardian

Activate the virtual environment:

```powershell
cd "$HOME\ResourceGuardian"
.\.venv\Scripts\Activate.ps1
```

Launch the dashboard:

```powershell
python -m streamlit run app.py --server.address 127.0.0.1
```

### Hardware Sensor Endpoint

```powershell
Invoke-RestMethod "http://127.0.0.1:8085/data.json"
```

### NVIDIA GPU

Run from an elevated PowerShell session if required:

```powershell
nvidia-smi
```

---

## 15. Testing and Verification

The complete environment was tested in stages.

### 15.1 Hardware Monitoring

Verified:

* CPU sensor detection.
* CPU temperature readings.
* CPU utilization readings.
* CPU package power.
* Physical RAM monitoring.
* NVIDIA GPU sensor detection.
* GPU temperature and hotspot readings.
* GPU utilization.
* GPU VRAM readings.
* SSD temperature.
* SSD health.

### 15.2 Resource Guardian

Verified:

* Python environment activation.
* Hardware module execution.
* LibreHardwareMonitor connectivity.
* Dashboard launch.
* Live dashboard refresh.
* Resource history updates.
* Ollama server detection.
* Loaded model detection.
* Manual model unloading.

### 15.3 Ollama Integration

Verified:

* Model loading.
* Local inference.
* CPU allocation.
* Runtime inspection using `ollama ps`.
* Model unloading through Resource Guardian.
* Empty model list after successful unloading.

### 15.4 Stability Test

Resource Guardian was left running for approximately five minutes.

The dashboard continued refreshing without reported errors.

**Stability test result:** Passed.

This verifies the tested monitoring session, not indefinite reliability under every possible workload.

---

## 16. Overall Implementation Status

| Component                              | Status     |
| -------------------------------------- | ---------- |
| USB formatting                         | Complete   |
| exFAT configuration                    | Complete   |
| Ollama model directory                 | Configured |
| Ollama environment variables           | Configured |
| Qwen2.5-Coder 3B installation          | Complete   |
| Local inference test                   | Successful |
| Resource-conscious model configuration | Complete   |
| CPU inference thread configuration     | Configured |
| Context window configuration           | Configured |
| Server concurrency settings            | Configured |
| NVIDIA GPU recovery                    | Complete   |
| NVIDIA-SMI verification                | Successful |
| LibreHardwareMonitor build             | Complete   |
| Hardware sensor integration            | Complete   |
| Python virtual environment             | Complete   |
| Resource Guardian dashboard            | Complete   |
| CPU/RAM/GPU/SSD monitoring             | Working    |
| Resource history                       | Working    |
| Ollama model detection                 | Verified   |
| Manual Ollama unloading                | Verified   |
| Qwen Code integration                  | Complete   |
| Five-minute stability test             | Passed     |
| Three.js project setup                 | Pending    |

---

## 17. Future Development Roadmap

With the local AI environment operational, the next development phase is focused on building a 3D project using Three.js.

### Planned Project: Barbie Dreamhouse

The intended project is an interactive 3D Barbie Dreamhouse experience.

Proposed technology stack:

| Technology | Purpose                                       |
| ---------- | --------------------------------------------- |
| Three.js   | 3D rendering and scene management             |
| Vite       | Frontend development server and build tooling |
| JavaScript | Application logic                             |
| HTML/CSS   | Interface and presentation                    |
| Qwen Code  | Local coding assistance                       |
| Ollama     | Local model inference                         |
| Git/GitHub | Version control and project hosting           |

### Development Stages

| Stage | Objective                                         |
| ----- | ------------------------------------------------- |
| 1     | Create the Three.js project                       |
| 2     | Configure the development environment             |
| 3     | Establish the 3D scene and camera                 |
| 4     | Design the Dreamhouse architecture                |
| 5     | Implement rooms, furniture and decorative objects |
| 6     | Add lighting, materials and animations            |
| 7     | Implement interactive navigation                  |
| 8     | Optimize rendering and resource consumption       |
| 9     | Deploy the completed project                      |

The Three.js project will remain separate from Resource Guardian and the Ollama model-storage directory.

---

## 18. Project Summary

The project began as an attempt to establish a portable local AI coding environment using Ollama and Qwen2.5-Coder.

Model storage was migrated to an external exFAT USB drive, and a separate resource-conscious model configuration was created.

The NVIDIA RTX 3050 was initially disabled by Windows. The device was subsequently enabled, and NVIDIA-SMI communication was restored.

LibreHardwareMonitor was built locally and configured to expose hardware sensor information through a loopback-only web endpoint.

An independent Python-based Resource Guardian dashboard was then implemented using Streamlit, psutil, requests and pandas.

The dashboard now provides live hardware monitoring, resource history, Ollama model detection and manual model unloading.

Qwen Code was successfully integrated with the local Ollama instance through its OpenAI-compatible API endpoint.

The five-minute dashboard stability test and the model detection/unloading tests were completed successfully.

**Current milestone:** Local AI coding environment operational, hardware monitoring implemented, and Qwen Code integrated.

**Next milestone:** Begin development of the Barbie Dreamhouse Three.js project.

---

*This repository serves as a technical record of the implementation, configuration decisions, troubleshooting procedures, verification results and future development of the local AI environment.*
