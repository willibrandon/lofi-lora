# Implementation Plan: Lofi LoRA Training Pipeline

**Branch**: `001-lofi-lora-training` | **Date**: 2025-12-22 | **Spec**: [spec.md](./spec.md)
**Input**: Feature specification from `/specs/001-lofi-lora-training/spec.md`

## Summary

Build a comprehensive pipeline for curating, training, and deploying a custom ACE-Step LoRA fine-tuned specifically for lofi music generation. The pipeline includes audio processing tools, prompt generation utilities, training wrapper scripts, validation metrics, and ONNX export with quantization support.

## Technical Context

**Language/Version**: Python 3.11 (ACE-Step compatibility, PyTorch ecosystem)
**Primary Dependencies**:
- torch, diffusers, peft (LoRA training)
- librosa, soundfile (audio processing)
- onnx, onnxruntime (model export)
- datasets (HuggingFace format)
- frechet_audio_distance, laion-clap (validation metrics)

**Storage**: File-based (audio files, JSON metadata, checkpoints)
**Testing**: pytest with audio fixtures
**Target Platform**: Linux (CUDA), local RTX 4090
**Project Type**: Single CLI-based training pipeline
**Performance Goals**:
- Training: 100k steps in ~60 hours on RTX 4090
- Inference: <5 seconds per minute of audio
- Audio processing: >100 files/minute

**Constraints**:
- 24GB VRAM (RTX 4090)
- 64GB system RAM
- 2TB NVMe storage
- All data legally licensed

**Scale/Scope**:
- 500 tracks (~25 hours audio)
- 38 style presets
- 100k training steps

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Requirement | Status |
|-----------|-------------|--------|
| I. Data Integrity | All tracks documented in metadata/sources.json with license info | PASS - FR-027 requires source metadata |
| II. Reproducibility | Version-locked deps, config stored, seeds logged | PASS - FR-014/FR-018 require checkpoints and logging |
| III. Quality Metrics | FAD < 5.0, CLAP > 0.7, human preference tests | PASS - FR-019/FR-020/SC-001/SC-002 specify these |
| IV. Model Optimization | Tiered quantization, audio components at FP16 min | PASS - FR-025/FR-026 specify precision requirements |
| V. Documentation | prompt.txt, lyrics.txt, style mapping per track | PASS - FR-011/FR-012/FR-028 require these files |

**Technical Standards Alignment**:
- Audio Specs: FR-002/FR-003/FR-004/FR-005/FR-006/FR-007 match constitution requirements
- Training Config: Constitution values (r=256, lora_alpha=32, use_rslora=true, lr=1e-4, --precision bf16) align with spec
- Export Requirements: FR-024/FR-026 match ONNX opset and precision rules

**All gates PASS. Proceeding to Phase 0.**

## Project Structure

### Documentation (this feature)

```text
specs/001-lofi-lora-training/
├── plan.md              # This file
├── spec.md              # Feature specification
├── research.md          # Phase 0 output
├── data-model.md        # Phase 1 output
├── quickstart.md        # Phase 1 output
├── contracts/           # Phase 1 output (CLI interface specs)
└── tasks.md             # Phase 2 output (/speckit.tasks)
```

### Source Code (repository root)

```text
scripts/
├── lofi_audio_processor.py    # Audio normalization and quality checks
├── lofi_prompt_generator.py   # Style preset prompt generation
├── convert_dataset.py         # HuggingFace dataset conversion
├── validate_checkpoint.py     # FAD/CLAP metric calculation
├── merge_lora.py              # LoRA weight merging
├── export_onnx.py             # ONNX export with quantization
└── run_training.sh            # Training wrapper script

config/
├── lofi_lora_config.json      # LoRA hyperparameters
├── style_presets.json         # 38 style preset definitions
└── training_defaults.json     # Default training parameters

data/                          # Ignored in git
├── raw/                       # Original source files by tier
├── processed/                 # Cleaned and validated by category
├── final/                     # Training-ready files
├── validation/                # Hold-out set (10%)
└── metadata/
    ├── sources.json           # Licensing info per track
    ├── style_mapping.json     # Style ID assignments
    └── quality_scores.json    # Human evaluation scores

tests/
├── unit/
│   ├── test_audio_processor.py
│   ├── test_prompt_generator.py
│   └── test_style_presets.py
├── integration/
│   ├── test_training_pipeline.py
│   └── test_export_pipeline.py
└── fixtures/
    └── sample_audio/          # Short test audio files
```

**Structure Decision**: Single project structure selected. This is a CLI-based ML pipeline without frontend/backend separation. All scripts are standalone utilities orchestrated via shell scripts.

## Complexity Tracking

> No constitution violations to justify.

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| (none) | N/A | N/A |
