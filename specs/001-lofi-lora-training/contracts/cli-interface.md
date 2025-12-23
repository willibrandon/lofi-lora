# CLI Interface Contracts

**Branch**: `001-lofi-lora-training` | **Date**: 2025-12-22

This document defines the command-line interface for all pipeline scripts.

---

## Audio Processor

**Script**: `scripts/lofi_audio_processor.py`

### Process Directory
```bash
python scripts/lofi_audio_processor.py process \
  --input-dir <path> \
  --output-dir <path> \
  [--sample-rate 48000] \
  [--target-format mp3] \
  [--bitrate 320] \
  [--min-duration 30] \
  [--max-duration 240] \
  [--normalize-peak -1] \
  [--workers 4]
```

**Input**: Directory containing audio files (MP3, WAV, FLAC, M4A, OGG)
**Output**: Processed MP3 files in output directory

**Exit Codes**:
| Code | Meaning |
|------|---------|
| 0 | Success (all files processed or skipped) |
| 1 | Fatal error (invalid paths, missing dependencies) |
| 2 | Partial success (some files failed quality checks) |

**JSON Output** (with `--json` flag):
```json
{
  "processed": 450,
  "skipped": 30,
  "failed": 20,
  "results": [
    {
      "source": "raw/track.wav",
      "status": "success",
      "output_path": "processed/track.mp3",
      "duration": 92.5,
      "quality_score": 87
    }
  ]
}
```

### Validate Single File
```bash
python scripts/lofi_audio_processor.py validate \
  --file <path> \
  [--json]
```

**Output**: Quality assessment for single file

---

## Prompt Generator

**Script**: `scripts/lofi_prompt_generator.py`

### Generate Prompts
```bash
python scripts/lofi_prompt_generator.py generate \
  --audio-dir <path> \
  --style-mapping <path> \
  --presets <path> \
  [--output-dir <path>] \
  [--include-lyrics]
```

**Input**:
- `audio-dir`: Directory with processed audio files
- `style-mapping`: JSON file mapping track_id → style_ids
- `presets`: Style presets JSON file

**Output**: `{name}_prompt.txt` and `{name}_lyrics.txt` files

### Interactive Mode
```bash
python scripts/lofi_prompt_generator.py interactive \
  --audio-file <path> \
  --presets <path>
```

**Output**: Interactive prompt generation for single file

---

## Dataset Converter

**Script**: `scripts/convert_dataset.py`

### Convert to HuggingFace
```bash
python scripts/convert_dataset.py \
  --data-dir <path> \
  --metadata-dir <path> \
  --output-dir <path> \
  [--validation-split 0.1] \
  [--repeat-count 200] \
  [--seed 42]
```

**Input**:
- `data-dir`: Directory with audio, prompt, lyrics files
- `metadata-dir`: Directory with sources.json, style_mapping.json

**Output**: HuggingFace dataset in Arrow format

**JSON Output**:
```json
{
  "train_samples": 90000,
  "validation_samples": 10000,
  "total_duration_hours": 25.3,
  "dataset_hash": "abc123...",
  "output_path": "/path/to/dataset"
}
```

---

## Training Runner

**Script**: `scripts/run_training.sh`

### Start Training
```bash
./scripts/run_training.sh \
  --dataset <path> \
  --exp-name <name> \
  [--config config/lofi_lora_config.json] \
  [--checkpoint-dir ~/.cache/ace-step/checkpoints] \
  [--max-steps 100000] \
  [--resume] \
  [--resume-from <step>]
```

**Input**:
- `dataset`: Path to HuggingFace dataset directory
- `exp-name`: Experiment name for logging

**Output**: Checkpoints in `exps/logs/{exp-name}/checkpoints/`

### Resume Training
```bash
./scripts/run_training.sh \
  --dataset <path> \
  --exp-name <name> \
  --resume
```

Automatically resumes from last valid checkpoint.

```bash
./scripts/run_training.sh \
  --dataset <path> \
  --exp-name <name> \
  --resume-from 50000
```

Resumes from specific step (manual override).

---

## Checkpoint Validator

**Script**: `scripts/validate_checkpoint.py`

### Validate Checkpoint
```bash
python scripts/validate_checkpoint.py \
  --checkpoint <path> \
  --validation-set <path> \
  [--generate-samples] \
  [--sample-output-dir <path>] \
  [--prompts-file <path>] \
  [--json]
```

**Input**:
- `checkpoint`: Path to LoRA checkpoint directory
- `validation-set`: Path to validation audio files

**Output**:
```json
{
  "checkpoint": "epoch=0-step=50000_lora",
  "fad_score": 4.2,
  "clap_score": 0.73,
  "style_coverage": {
    "CORE-01": true,
    "CORE-02": true
  },
  "samples_generated": 38
}
```

### Compare Checkpoints
```bash
python scripts/validate_checkpoint.py compare \
  --checkpoints <path1> <path2> ... \
  --validation-set <path> \
  [--json]
```

**Output**: Comparison table of all checkpoints

---

## LoRA Merger

**Script**: `scripts/merge_lora.py`

### Merge Weights
```bash
python scripts/merge_lora.py \
  --base-model <path> \
  --lora-checkpoint <path> \
  --output-dir <path> \
  [--verify]
```

**Input**:
- `base-model`: Path to ACE-Step base model
- `lora-checkpoint`: Path to trained LoRA checkpoint

**Output**: Merged model in output directory

**Exit Codes**:
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Base model not found |
| 2 | LoRA checkpoint invalid |
| 3 | Merge failed |

---

## ONNX Exporter

**Script**: `scripts/export_onnx.py`

### Export Model
```bash
python scripts/export_onnx.py \
  --model <path> \
  --output <path> \
  --precision <fp16|int8|int4> \
  [--opset 17] \
  [--calibration-data <path>] \
  [--verify]
```

**Input**:
- `model`: Path to merged PyTorch model
- `precision`: Target precision tier

**Output**: ONNX model file(s)

**Calibration Data** (required for int4):
```json
{
  "prompts": [
    "lofi hip hop, chill, piano, 80 bpm",
    "jazz lofi, saxophone, rhodes, coffee shop"
  ]
}
```

**Exit Codes**:
| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Model load failed |
| 2 | Export failed |
| 3 | Verification failed |

---

## Common Flags

All scripts support these common flags:

| Flag | Description |
|------|-------------|
| `--help`, `-h` | Show help message |
| `--version` | Show script version |
| `--verbose`, `-v` | Increase output verbosity |
| `--quiet`, `-q` | Suppress non-error output |
| `--json` | Output in JSON format |
| `--dry-run` | Show what would be done without executing |

---

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `LOFI_DATA_DIR` | Base data directory | `./data` |
| `LOFI_CONFIG_DIR` | Configuration directory | `./config` |
| `LOFI_CHECKPOINT_DIR` | Checkpoint storage | `./exps/logs` |
| `ACE_STEP_DIR` | ACE-Step installation | `~/.cache/ace-step` |
| `CUDA_VISIBLE_DEVICES` | GPU selection | `0` |
