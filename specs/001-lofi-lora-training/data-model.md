# Data Model: Lofi LoRA Training Pipeline

**Branch**: `001-lofi-lora-training` | **Date**: 2025-12-22

## Entity Overview

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Track     │────▶│ StylePreset │     │  Checkpoint │
└─────────────┘     └─────────────┘     └─────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ SourceRecord│     │  TagCategory│     │QualityMetrics│
└─────────────┘     └─────────────┘     └─────────────┘
```

---

## Track

Represents an individual audio file with all associated training metadata.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| track_id | string | Yes | Unique identifier (e.g., "track_001") |
| filename | string | Yes | Audio filename without path |
| audio_path | string | Yes | Relative path to processed MP3 |
| prompt_path | string | Yes | Relative path to prompt.txt |
| lyrics_path | string | Yes | Relative path to lyrics.txt |
| duration | float | Yes | Duration in seconds (30-240) |
| sample_rate | int | Yes | Always 48000 |
| quality_score | int | Yes | Automated score 0-100 |
| style_ids | string[] | Yes | List of assigned style IDs |
| source_id | string | Yes | Reference to SourceRecord |
| human_verified | boolean | No | Manual review completed |
| split | enum | Yes | "train" or "validation" |

### Validation Rules
- `duration` must be between 30 and 240 seconds
- `quality_score` must be between 0 and 100
- `style_ids` must contain at least one valid style ID
- `split` defaults to "train"; 10% random assigned to "validation"

### State Transitions
```
RAW → PROCESSED → TAGGED → VERIFIED → TRAINING_READY
                              ↑
                         (optional)
```

---

## StylePreset

Predefined tag combinations for consistent prompt generation.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| style_id | string | Yes | Unique ID (e.g., "CORE-01", "SEASON-02") |
| name | string | Yes | Human-readable name |
| category | enum | Yes | "core", "seasonal", "thematic", "instrumental", "vocal" |
| extends | string | No | Parent preset ID for inheritance |
| base_tags | string[] | Yes | Primary genre/style tags |
| instruments | string[] | Yes | Typical instruments |
| production | string[] | Yes | Production characteristics |
| mood_options | string[] | Yes | Applicable mood tags |
| tempo_range | [int, int] | Yes | Min/max BPM |
| atmosphere | string | No | Default atmosphere tag |
| vocal_type | string | No | Default vocal type if applicable |

### Categories and Counts
| Category | Count | ID Pattern |
|----------|-------|------------|
| Core | 7 | CORE-01 to CORE-07 |
| Seasonal | 9 | SEASON-01 to SEASON-09 |
| Thematic | 10 | THEME-01 to THEME-10 |
| Instrumental | 7 | INST-01 to INST-07 |
| Vocal | 5 | VOCAL-01 to VOCAL-05 |

---

## SourceRecord

Licensing and provenance information for legal compliance.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| source_id | string | Yes | Unique identifier |
| track_ids | string[] | Yes | Tracks from this source |
| source_type | enum | Yes | "commissioned", "royalty_free", "creative_commons", "partnership" |
| provider | string | Yes | Platform or artist name |
| license_type | string | Yes | Specific license (e.g., "CC BY 4.0", "AI Training Rights") |
| cost | float | No | Acquisition cost in USD |
| date_acquired | date | Yes | ISO 8601 date |
| verified_by | string | No | Who verified the license |
| notes | string | No | Additional licensing notes |

### Validation Rules
- `license_type` must explicitly permit AI training
- `source_type` "commissioned" requires `cost` field
- `date_acquired` must not be in the future

---

## Checkpoint

Training snapshot with associated quality metrics.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| checkpoint_id | string | Yes | Format: "epoch=0-step={N}_lora" |
| step | int | Yes | Training step number |
| path | string | Yes | Relative path to checkpoint directory |
| timestamp | datetime | Yes | When checkpoint was saved |
| fad_score | float | No | FAD against validation set |
| clap_score | float | No | CLAP alignment score |
| is_valid | boolean | Yes | Checkpoint integrity verified |
| config_hash | string | Yes | Hash of training config |
| dataset_hash | string | Yes | Hash of dataset version |
| samples_generated | boolean | No | Sample audio generated |

### Validation Rules
- `step` must be a multiple of checkpoint interval (default 5000)
- `is_valid` set to false if loading fails
- `fad_score` and `clap_score` populated after validation run

### Retention Policy
- Keep best 3 checkpoints by FAD score
- Keep latest checkpoint always
- Intermediate checkpoints may be pruned

---

## QualityMetrics

Quality evaluation results for a checkpoint.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| checkpoint_id | string | Yes | Reference to Checkpoint |
| evaluation_date | datetime | Yes | When evaluation ran |
| fad_score | float | Yes | Fréchet Audio Distance |
| clap_score | float | Yes | CLAP text-audio alignment |
| style_coverage | object | No | Per-style generation success |
| human_evaluation | object | No | Human preference test results |

### Style Coverage Schema
```json
{
  "CORE-01": {"generated": true, "recognizable": true},
  "CORE-02": {"generated": true, "recognizable": false}
}
```

### Human Evaluation Schema
```json
{
  "evaluator_count": 10,
  "accuracy": 0.52,
  "p_value": 0.08,
  "date": "2025-01-15"
}
```

---

## TagCategory

Controlled vocabulary for prompt construction.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| category | enum | Yes | "genre", "mood", "instruments", "tempo", "production", "atmosphere", "vocal_type" |
| tags | string[] | Yes | Valid tags in this category |
| canonical_forms | object | No | Alias → canonical mapping |

### Canonical Forms Example
```json
{
  "lo-fi": "lofi",
  "rhodes piano": "rhodes",
  "hip hop": "hip hop"
}
```

---

## Dataset

Collection metadata for training dataset.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| dataset_id | string | Yes | Version identifier |
| created | datetime | Yes | Creation timestamp |
| track_count | int | Yes | Total tracks in dataset |
| train_count | int | Yes | Tracks in training split |
| validation_count | int | Yes | Tracks in validation split |
| total_duration | float | Yes | Total audio duration in hours |
| style_distribution | object | Yes | Track count per style category |
| hash | string | Yes | Content hash for reproducibility |

### Style Distribution Schema
```json
{
  "core": 200,
  "seasonal": 100,
  "thematic": 100,
  "instrumental": 50,
  "vocal": 50
}
```

---

## File Storage Layout

```
data/
├── raw/
│   ├── commissioned/
│   ├── royalty-free/
│   ├── creative-commons/
│   └── partnerships/
├── processed/
│   ├── core/
│   ├── seasonal/
│   ├── thematic/
│   ├── instrumental/
│   └── vocal/
├── final/
│   ├── track_001.mp3
│   ├── track_001_prompt.txt
│   ├── track_001_lyrics.txt
│   └── ...
├── validation/
│   └── (same structure as final/)
├── metadata/
│   ├── sources.json         # Array of SourceRecord
│   ├── style_mapping.json   # {track_id: [style_ids]}
│   ├── quality_scores.json  # {track_id: {score, verified, date}}
│   └── dataset_manifest.json # Dataset entity
└── checkpoints/
    ├── manifest.json        # Array of Checkpoint
    └── epoch=0-step=*/      # Checkpoint directories
```

---

## JSON Schema Locations

| Entity | Schema File |
|--------|-------------|
| Track | `config/schemas/track.json` |
| StylePreset | `config/schemas/style_preset.json` |
| SourceRecord | `config/schemas/source_record.json` |
| Checkpoint | `config/schemas/checkpoint.json` |
| Dataset | `config/schemas/dataset.json` |
