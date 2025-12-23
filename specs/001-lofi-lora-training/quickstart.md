# Quickstart: Lofi LoRA Training Pipeline

**Branch**: `001-lofi-lora-training` | **Date**: 2025-12-22

## Prerequisites

- Python 3.11+
- CUDA 12.1+ with RTX 4090 (24GB VRAM)
- 64GB+ system RAM
- 2TB+ available storage
- FFmpeg installed
- ACE-Step base model downloaded

## Installation

```bash
# Clone repository
git clone https://github.com/willibrandon/lofi-lora.git
cd lofi-lora

# Create virtual environment
python -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows

# Install dependencies
pip install -r requirements.txt

# Install ACE-Step (editable)
pip install -e /path/to/ACE-Step

# Verify installation
python -c "import torch; print(torch.cuda.is_available())"
```

## Quick Training Run (10 tracks)

### 1. Prepare Sample Data

```bash
# Create directory structure
mkdir -p data/raw/sample data/processed

# Place 10 audio files in data/raw/sample/
# Files should be 30-240 seconds, any format

# Process audio
python scripts/lofi_audio_processor.py process \
  --input-dir data/raw/sample \
  --output-dir data/processed \
  --json
```

### 2. Create Style Mapping

```bash
# Create minimal style mapping
cat > data/metadata/style_mapping.json << 'EOF'
{
  "track_001": ["CORE-01"],
  "track_002": ["CORE-01"],
  "track_003": ["CORE-02"]
}
EOF
```

### 3. Generate Prompts

```bash
python scripts/lofi_prompt_generator.py generate \
  --audio-dir data/processed \
  --style-mapping data/metadata/style_mapping.json \
  --presets config/style_presets.json \
  --include-lyrics
```

### 4. Convert to HuggingFace Dataset

```bash
python scripts/convert_dataset.py \
  --data-dir data/processed \
  --metadata-dir data/metadata \
  --output-dir data/dataset \
  --validation-split 0.1 \
  --repeat-count 10
```

### 5. Run Training (Short Test)

```bash
./scripts/run_training.sh \
  --dataset data/dataset \
  --exp-name test_run \
  --max-steps 1000
```

### 6. Validate Checkpoint

```bash
python scripts/validate_checkpoint.py \
  --checkpoint exps/logs/test_run/checkpoints/epoch=0-step=1000_lora \
  --validation-set data/validation \
  --generate-samples \
  --json
```

## Full Training Run (500 tracks)

### Preparation Time: ~1 week

1. **Acquire Data** (see [Data Sources](./spec.md#1.2-legal-data-sources))
   - Commission or license 500 tracks
   - Document all licenses in `data/metadata/sources.json`

2. **Categorize Tracks**
   - Assign style IDs to each track
   - Aim for balanced distribution across categories

3. **Process Full Dataset**
   ```bash
   python scripts/lofi_audio_processor.py process \
     --input-dir data/raw \
     --output-dir data/processed \
     --workers 8
   ```

4. **Generate All Prompts**
   ```bash
   python scripts/lofi_prompt_generator.py generate \
     --audio-dir data/processed \
     --style-mapping data/metadata/style_mapping.json \
     --presets config/style_presets.json \
     --include-lyrics
   ```

5. **Create Training Dataset**
   ```bash
   python scripts/convert_dataset.py \
     --data-dir data/processed \
     --metadata-dir data/metadata \
     --output-dir data/lofi_dataset \
     --repeat-count 200
   ```

### Training Time: ~60 hours

```bash
./scripts/run_training.sh \
  --dataset data/lofi_dataset \
  --exp-name lofi_beats_v1 \
  --max-steps 100000
```

**Monitoring**:
- Checkpoints saved every 5,000 steps
- Sample audio generated every 5,000 steps
- View logs at `exps/logs/lofi_beats_v1/`

**Resume on failure**:
```bash
./scripts/run_training.sh \
  --dataset data/lofi_dataset \
  --exp-name lofi_beats_v1 \
  --resume
```

### Validation & Export

```bash
# Find best checkpoint
python scripts/validate_checkpoint.py compare \
  --checkpoints exps/logs/lofi_beats_v1/checkpoints/* \
  --validation-set data/validation \
  --json

# Merge LoRA with base model
python scripts/merge_lora.py \
  --base-model ~/.cache/ace-step/checkpoints \
  --lora-checkpoint exps/logs/lofi_beats_v1/checkpoints/epoch=0-step=100000_lora \
  --output-dir models/lofi_merged

# Export to ONNX (INT8)
python scripts/export_onnx.py \
  --model models/lofi_merged \
  --output models/lofi_int8.onnx \
  --precision int8 \
  --verify
```

## Directory Structure After Training

```
lofi-lora/
├── config/
│   ├── lofi_lora_config.json
│   ├── style_presets.json
│   └── training_defaults.json
├── data/
│   ├── raw/                    # Original audio
│   ├── processed/              # Normalized audio + prompts
│   ├── lofi_dataset/           # HuggingFace format
│   ├── validation/             # Hold-out set
│   └── metadata/
│       ├── sources.json
│       ├── style_mapping.json
│       └── quality_scores.json
├── exps/
│   └── logs/
│       └── lofi_beats_v1/
│           ├── checkpoints/
│           ├── samples/
│           └── metrics.json
└── models/
    ├── lofi_merged/
    └── lofi_int8.onnx
```

## Common Issues

### VRAM Exceeded
Reduce LoRA rank:
```json
// config/lofi_lora_config.json
{
  "r": 128,  // Reduced from 256
  "lora_alpha": 16
}
```

### Training Diverges
Check gradient clipping and reduce learning rate:
```bash
./scripts/run_training.sh ... \
  --learning-rate 5e-5 \
  --gradient-clip-val 0.25
```

### ACE-Step Not Found
Ensure model is downloaded:
```bash
python -c "from acestep.pipeline_ace_step import ACEStepPipeline; ACEStepPipeline()"
```

## Next Steps

- Review [Data Model](./data-model.md) for entity schemas
- Review [CLI Interface](./contracts/cli-interface.md) for all command options
- See [Spec](./spec.md) for complete requirements and success criteria
