# lofi-lora

Custom ACE-Step LoRA training for lofi music generation.

## Overview

This repository contains the training pipeline for a lofi-optimized LoRA adapter for [ACE-Step](https://github.com/ace-step/ACE-Step). The trained model enhances lofi beat generation with authentic vinyl warmth, jazz influences, and genre-specific characteristics across 30+ style variations.

## Related Projects

- [lofi.nvim](https://github.com/willibrandon/lofi.nvim) - Neovim plugin for AI-generated lofi beats
- [willibrandon/lofi-models](https://huggingface.co/willibrandon/lofi-models) - Trained model weights
- [ACE-Step](https://github.com/ace-step/ACE-Step) - Base model and training framework

## Documentation

See [docs/lofi-lora-training.md](docs/lofi-lora-training.md) for the full training specification including:

- Data curation strategy and legal sources
- Style taxonomy (30+ lofi variations)
- Prompt engineering system
- Training configuration
- Model optimization (quantization)
- Quality validation

## Quick Start

```bash
# Clone repos
git clone https://github.com/willibrandon/lofi-lora.git
git clone https://github.com/ace-step/ACE-Step.git

# Install ACE-Step
cd ACE-Step && pip install -e . && cd ..

# Install dependencies
cd lofi-lora && pip install -r requirements.txt

# Prepare data (see docs for data acquisition)
# Place tracks in data/ with corresponding _prompt.txt and _lyrics.txt files

# Convert to HuggingFace dataset
python scripts/convert_dataset.py --data_dir ./data --output_name lofi_lora_dataset

# Train
./train.sh
```

## Hardware Requirements

- GPU: RTX 4090 (24GB VRAM) or equivalent
- RAM: 64GB+
- Storage: 2TB+ NVMe
- Training time: ~60 hours for 100k steps

## License

Apache 2.0
