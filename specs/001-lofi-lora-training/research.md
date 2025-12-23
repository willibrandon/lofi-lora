# Research: Lofi LoRA Training Pipeline

**Branch**: `001-lofi-lora-training` | **Date**: 2025-12-22

## Research Summary

This document consolidates technical research for implementing the Lofi LoRA training pipeline. All NEEDS CLARIFICATION items from Technical Context have been resolved.

---

## 1. ACE-Step LoRA Training Best Practices

### Decision
Use the ACE-Step training infrastructure as documented in [TRAIN_INSTRUCTION.md](https://github.com/ace-step/ACE-Step/blob/main/TRAIN_INSTRUCTION.md) with the RapMachine LoRA configuration as reference.

### Rationale
- ACE-Step's trainer.py provides proven LoRA training for music generation
- RapMachine LoRA demonstrates style-specific fine-tuning works well with r=256
- PEFT library integration allows standard LoRA patterns

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| Custom training loop | Reinvents ACE-Step's validated approach |
| Full fine-tuning | Exceeds VRAM limits, loses base capabilities |
| Lower rank LoRA (r=32) | Insufficient for capturing stylistic nuance |

### Implementation Notes
- Install ACE-Step as editable dependency: `pip install -e /path/to/ACE-Step`
- Use `trainer.py` with `--lora_config_path` pointing to our config
- Target modules: `speaker_embedder`, `linear_q/k/v`, `to_q/k/v`, `to_out.0`
- LoRA config: `r=256`, `lora_alpha=32`, `use_rslora=true`
- Training automatically downloads MERT (`m-a-p/MERT-v1-330M`) and mHuBERT (`utter-project/mHuBERT-147`) for SSL alignment
- Precision: Use `--precision bf16` explicitly (ACE-Step defaults to fp32)
- Checkpoint interval: `--every_n_train_steps 2000` (ACE-Step default)

---

## 2. Audio Processing Pipeline

### Decision
Use librosa for loading/analysis, soundfile for writing, FFmpeg via subprocess for MP3 encoding.

### Rationale
- librosa: Industry standard for audio analysis, supports all input formats
- soundfile: Clean interface for WAV I/O, required for intermediate processing
- FFmpeg: Best-in-class MP3 encoding with libmp3lame

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| pydub | Slower, less precise sample manipulation |
| torchaudio | Heavier dependency, FFmpeg still needed for MP3 |
| Direct FFmpeg only | Loses numpy-based quality checks |

### Implementation Notes
- Load at 48kHz (ACE-Step native), maintain stereo
- Quality checks in numpy before export
- Export to temp WAV → FFmpeg MP3 → cleanup

---

## 3. Dataset Format (HuggingFace)

### Decision
Use HuggingFace `datasets` library with custom audio loading, structured as:
```python
{
    "audio": {"path": str, "sampling_rate": 48000},
    "prompt": str,
    "lyrics": str,
    "style_id": str,
    "duration": float
}
```

### Rationale
- ACE-Step trainer expects HuggingFace dataset format
- Streaming support for large datasets
- Efficient disk caching with Arrow format

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| Raw file iteration | No shuffling, caching, or parallelism |
| Custom DataLoader | Incompatible with ACE-Step trainer |
| WebDataset | Overkill for local training |

### Implementation Notes
- Use `Dataset.from_generator()` for lazy loading
- Save to disk with `dataset.save_to_disk()`
- Repeat count handled by dataset slicing

---

## 4. Quality Metrics Implementation

### Decision
- FAD: Use `frechet_audio_distance` package with VGGish embeddings
- CLAP: Use `laion/clap-htsat-unfused` from HuggingFace

### Rationale
- FAD with VGGish is the standard for audio generation evaluation
- CLAP provides text-audio alignment scores matching ACE-Step's training objective

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| Custom embedding model | No established baseline, harder to compare |
| PESQ/ViSQOL | Designed for speech, not music |
| User studies only | Expensive, slow, not automatable |

### Implementation Notes
- Generate samples at each checkpoint with fixed seeds
- Compute FAD against validation set (10% holdout)
- CLAP score: cosine similarity between prompt embedding and audio embedding

---

## 5. ONNX Export Strategy

### Decision
Use `torch.onnx.export()` for base model, `onnxruntime.quantization` for INT8/INT4.

### Rationale
- PyTorch native ONNX export is most reliable for transformer models
- ONNX Runtime quantization supports dynamic INT8 without calibration data
- Static quantization with calibration improves INT4 quality

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| TensorRT only | NVIDIA-specific, no CPU support |
| GGML/llama.cpp style | Not designed for audio diffusion |
| Keep PyTorch only | Larger files, slower inference |

### Implementation Notes
- Export sequence: Merge LoRA → FP32 ONNX → FP16 → INT8 → INT4
- Preserve dynamic axes for variable sequence length
- Test inference after each quantization step

---

## 6. Checkpoint Recovery Strategy

### Decision
Implement automatic resume with manual override via checkpoint manifest.

### Rationale
- Training runs are long (60+ hours), automatic recovery minimizes manual intervention
- Manual override needed for corrupted checkpoints or intentional rollback
- Manifest file tracks valid checkpoints with quality scores

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| Always manual restart | Too error-prone for multi-day runs |
| No checkpoint tracking | Can't identify best checkpoint reliably |
| Cloud-based state | Adds complexity, local training assumed |

### Implementation Notes
- Create `checkpoints/manifest.json` with step, path, FAD score, timestamp
- On resume: read manifest, find latest valid checkpoint, restore
- `--resume-from STEP` flag for manual selection

---

## 7. Style Preset Architecture

### Decision
JSON-based preset system with inheritance for shared attributes.

### Rationale
- 38 presets across 5 categories need consistent structure
- Inheritance reduces duplication (e.g., all CORE styles share vinyl crackle)
- JSON enables programmatic generation and validation

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| Python classes | Harder to edit without code changes |
| YAML | No significant advantage over JSON |
| Database | Overkill for static configuration |

### Implementation Notes
```json
{
  "CORE-01": {
    "extends": ["_base_lofi"],
    "base_tags": ["lofi hip hop", "chill"],
    "instruments": ["piano", "rhodes"],
    "production": ["boom bap drums", "vinyl crackle"]
  },
  "_base_lofi": {
    "production": ["tape saturation", "analog warmth"],
    "tempo_range": [70, 90]
  }
}
```

---

## 8. Metadata Schema

### Decision
Three JSON files in `data/metadata/`:
- `sources.json`: Licensing and provenance
- `style_mapping.json`: Track → style ID assignments
- `quality_scores.json`: Human and automated quality scores

### Rationale
- Separation of concerns: licensing is legal, styling is creative, quality is evaluation
- JSON for easy querying and updates
- Constitution requires all three for compliance

### Alternatives Considered
| Alternative | Rejected Because |
|-------------|------------------|
| SQLite database | More complex for read-heavy, write-light workload |
| Single monolithic file | Harder to update individual aspects |
| Embedded in audio metadata | Not all formats support, harder to query |

### Implementation Notes
- `sources.json` indexed by track_id
- `style_mapping.json` allows multiple style IDs per track
- `quality_scores.json` includes timestamp for score versioning

---

## 9. Dependency Versions

### Decision
Pin exact versions for reproducibility:

```
torch==2.1.2
diffusers==0.25.0
peft==0.7.1
librosa==0.10.1
soundfile==0.12.1
onnx==1.15.0
onnxruntime==1.16.3
datasets==2.16.0
frechet_audio_distance==0.3.0
transformers==4.36.2
```

### Rationale
- ACE-Step tested with specific PyTorch/diffusers versions
- Exact pins prevent silent breakage on dependency updates
- requirements.txt tracks all transitive dependencies

### Implementation Notes
- Generate with `pip freeze > requirements.txt` after validated setup
- Include Python version in setup instructions
- Test with fresh venv before release

---

## Open Questions (Deferred to Implementation)

| Question | Deferred Because |
|----------|------------------|
| Exact CLAP model checkpoint | Can be finalized during validation script development |
| Audio fingerprinting library | Duplicate detection is P3, can evaluate later |
| HuggingFace Hub authentication | Standard `huggingface-cli login` workflow |

---

## References

- [ACE-Step Training Instructions](https://github.com/ace-step/ACE-Step/blob/main/TRAIN_INSTRUCTION.md)
- [RapMachine LoRA Configuration](https://github.com/ace-step/ACE-Step/blob/main/ZH_RAP_LORA.md)
- [PEFT LoRA Documentation](https://huggingface.co/docs/peft/conceptual_guides/lora)
- [ONNX Runtime Quantization](https://onnxruntime.ai/docs/performance/model-optimizations/quantization.html)
- [Fréchet Audio Distance](https://github.com/google-research/google-research/tree/master/frechet_audio_distance)
- [LAION CLAP](https://github.com/LAION-AI/CLAP)
