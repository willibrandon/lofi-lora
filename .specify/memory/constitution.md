<!--
Sync Impact Report
==================
Version change: N/A (initial) → 1.0.0
Added sections:
  - 5 Core Principles (Data Integrity, Reproducibility, Quality Metrics, Model Optimization, Documentation)
  - Technical Standards (audio specs, training config, export requirements)
  - Development Workflow (pipeline stages, checkpoint management)
  - Governance (amendment process, versioning policy)
Removed sections: N/A (template placeholders replaced)
Templates requiring updates:
  - .specify/templates/plan-template.md ✅ (Constitution Check section compatible)
  - .specify/templates/spec-template.md ✅ (Requirements format compatible)
  - .specify/templates/tasks-template.md ✅ (Phase structure compatible)
Follow-up TODOs: None
==================
-->

# Lofi LoRA Training Constitution

## Core Principles

### I. Data Integrity & Licensing

All training data MUST have verifiable legal rights for AI training purposes. Each track MUST be documented in `metadata/sources.json` with:
- License type and acquisition date
- Source tier (commissioned, royalty-free, CC, partnership)
- Cost and provider information
- Explicit confirmation of AI training rights

No track may enter the training pipeline without license verification. Ambiguous licensing MUST be treated as non-permissive until clarified with the rights holder.

### II. Reproducibility

Every training run MUST be reproducible from recorded configuration. This requires:
- Version-locked dependencies in `requirements.txt` with exact versions
- LoRA configuration stored in `config/lofi_lora_config.json`
- Random seeds logged for each checkpoint
- Training command and hyperparameters recorded in experiment logs
- Dataset version hash stored with each checkpoint

Checkpoints MUST be saved every 5,000 steps. Each checkpoint MUST include the configuration state that produced it.

### III. Quality Metrics

Model quality MUST be validated against objective metrics before release:
- FAD (Fréchet Audio Distance) < 5.0 vs reference lofi validation set
- CLAP prompt alignment score > 0.7
- Human preference tests: AI tracks MUST be indistinguishable from real lofi (p > 0.05)

Quantized models MUST maintain quality within defined tolerances:
- Tier 2 (INT8): FAD within 5% of FP32 baseline
- Tier 3 (INT4): FAD within 15% of FP32 baseline

No model may be published to HuggingFace Hub without passing all quality gates.

### IV. Model Optimization

Model size reduction MUST follow the tiered approach:
- Tier 1 (FP16): Default for quality-first deployments
- Tier 2 (INT8 Mixed): Recommended balance of size/quality
- Tier 3 (INT4 Mixed): Experimental, edge deployment only

Audio-critical components (DCAE decoder, Vocoder) MUST remain at FP16 minimum. Lyric encoder and speaker embedder MUST remain at INT8 minimum to preserve vocal capability.

### V. Documentation & Traceability

Every track MUST have accompanying metadata files:
- `{name}_prompt.txt`: Comma-separated style tags following the taxonomy
- `{name}_lyrics.txt`: Song structure or `[Instrumental]` marker
- Style ID assignment in `metadata/style_mapping.json`

Prompts MUST use the standardized tag categories: genre, mood, instruments, tempo, production, atmosphere, vocal_type. Custom tags MUST be documented in the style taxonomy before use.

## Technical Standards

### Audio Specifications

| Attribute | Requirement |
|-----------|-------------|
| Sample Rate | 48kHz (ACE-Step native) |
| Format | 320kbps MP3 or lossless source |
| Duration | 30s minimum, 180s maximum, 60-120s optimal |
| Channels | Stereo (mono sources MUST be converted) |
| Peak Level | Normalized to -1dB |
| Clipping | < 0.1% of samples above threshold |
| Silence | < 50% of track duration |

### Training Configuration

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| LoRA rank (r) | 256 | High rank for stylistic nuance |
| Learning rate | 1e-4 | Proven stable for LoRA |
| Max steps | 100,000 | 500 tracks × 200 repeats |
| Checkpoint interval | 5,000 steps | Recovery and comparison |
| Precision | bf16-mixed | RTX 4090 optimized |
| Gradient clip | 0.5 | Prevent explosion |

### Export Requirements

- ONNX export MUST use opset 17 for INT8, opset 21+ for INT4
- LoRA weights MUST be mergeable with base ACE-Step model
- All exported models MUST include a model card with usage instructions

## Development Workflow

### Pipeline Stages

1. **Data Acquisition**: Source tracks with verified licensing
2. **Audio Processing**: Validate and normalize with `lofi_audio_processor.py`
3. **Prompt Tagging**: Generate prompts using style taxonomy
4. **Dataset Creation**: Convert to HuggingFace format with `convert_dataset.py`
5. **Training**: Execute training with logged configuration
6. **Validation**: Run quality metrics against validation set
7. **Quantization**: Apply tiered optimization if size reduction needed
8. **Export**: Produce ONNX artifacts for deployment
9. **Release**: Publish to HuggingFace Hub with model card

### Checkpoint Management

- Checkpoints MUST be retained for the best 3 models by FAD score
- Intermediate checkpoints MAY be pruned after validation
- Final release checkpoint MUST be stored with full configuration
- Checkpoint naming: `epoch=0-step={N}_lora` where N is step count

### Quality Checkpoints

| Stage | Gate |
|-------|------|
| Post-training | FAD < 5.0, CLAP > 0.7 |
| Post-quantization | FAD within tier tolerance |
| Pre-release | Human evaluation complete |
| Release | Model card approved |

## Governance

This constitution supersedes all other development practices for the lofi-lora repository. Amendments require:

1. Documented rationale for the change
2. Version increment following semantic versioning:
   - MAJOR: Backward-incompatible changes to principles or quality gates
   - MINOR: New principles, sections, or materially expanded guidance
   - PATCH: Clarifications, typos, non-semantic refinements
3. Update to all dependent templates if principles change
4. Commit message format: `docs: amend constitution to vX.Y.Z (change summary)`

All PRs MUST verify compliance with this constitution. The `/speckit.analyze` command MUST flag constitution violations as CRITICAL severity.

**Version**: 1.0.0 | **Ratified**: 2025-12-22 | **Last Amended**: 2025-12-22
