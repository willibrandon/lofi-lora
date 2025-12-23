# Feature Specification: Lofi LoRA Training Pipeline

**Feature Branch**: `001-lofi-lora-training`
**Created**: 2025-12-22
**Status**: Draft
**Input**: User description: "Comprehensive pipeline for curating, training, and deploying a custom ACE-Step LoRA fine-tuned specifically for lofi music generation"

## Clarifications

### Session 2025-12-22

- Q: What is the training failure recovery strategy? → A: Automatic resume + manual override - auto-resume from last good checkpoint, allow manual selection
- Q: How should the system handle ACE-Step base model unavailability? → A: Fail fast with clear diagnostic - abort startup with specific error and remediation steps

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Train Custom Lofi LoRA Model (Priority: P1)

As a developer, I want to train a custom LoRA fine-tuned on lofi music so that I can generate high-quality lofi beats that match specific styles, moods, and themes.

**Why this priority**: This is the core value proposition - without a trained model, no other features matter. The entire pipeline exists to produce a quality lofi LoRA model.

**Independent Test**: Can be fully tested by running the training pipeline on a small dataset (10-20 tracks) and verifying the model produces lofi-characteristic audio when given prompts.

**Acceptance Scenarios**:

1. **Given** a prepared dataset with audio files, prompts, and lyrics files, **When** I run the training pipeline with default configuration, **Then** the system produces checkpoint files at regular intervals that can be used for inference.

2. **Given** a completed training run of 100,000 steps, **When** I evaluate the model against a held-out validation set, **Then** the Fréchet Audio Distance (FAD) score is less than 5.0.

3. **Given** a trained model, **When** I generate audio with a prompt like "lofi hip hop, chill, piano, vinyl crackle, 80 bpm", **Then** the output exhibits authentic lofi characteristics (vinyl texture, warm compression, appropriate instrumentation).

---

### User Story 2 - Prepare Training Data (Priority: P1)

As a developer, I want to process and prepare audio files for training so that my dataset meets ACE-Step's requirements and produces optimal training results.

**Why this priority**: Without properly prepared data, training cannot succeed. This is a prerequisite for P1 training and equally critical.

**Independent Test**: Can be tested by processing a batch of raw audio files and verifying all outputs meet format requirements (duration, sample rate, normalization).

**Acceptance Scenarios**:

1. **Given** raw audio files in various formats (MP3, WAV, FLAC), **When** I run the audio processing pipeline, **Then** all files are converted to 320kbps MP3 at 48kHz sample rate.

2. **Given** an audio file shorter than 30 seconds, **When** processed, **Then** the file is skipped with a clear status message indicating "too_short".

3. **Given** an audio file longer than 240 seconds, **When** processed, **Then** the file is trimmed to 240 seconds with audio quality preserved.

4. **Given** an audio file with clipping (>0.1% of samples), **When** quality assessment runs, **Then** the file is rejected with reason "excessive_clipping".

5. **Given** a processed audio file, **When** I generate prompt and lyrics files, **Then** accompanying `{name}_prompt.txt` and `{name}_lyrics.txt` files are created with correct formatting.

---

### User Story 3 - Generate Style-Consistent Prompts (Priority: P2)

As a developer, I want to generate consistent and comprehensive prompt tags for each track so that the model learns the correct associations between text descriptions and audio characteristics.

**Why this priority**: Prompt quality directly impacts model quality, but can be refined iteratively. Initial training can proceed with basic prompts.

**Independent Test**: Can be tested by generating prompts for a set of categorized tracks and verifying tag consistency across the style taxonomy.

**Acceptance Scenarios**:

1. **Given** a track categorized as "CORE-01" (Classic Lofi Hip Hop), **When** I generate a prompt using the style preset, **Then** the output includes base tags like "lofi hip hop, chill, boom bap drums, vinyl crackle".

2. **Given** a Christmas lofi track (SEASON-02), **When** I generate a prompt, **Then** the output includes seasonal tags like "christmas lofi, cozy, festive, holiday".

3. **Given** an instrumental track, **When** I create the lyrics file, **Then** it contains only `[Instrumental]`.

4. **Given** a vocal track with lyrics, **When** I create the lyrics file, **Then** it follows ACE-Step's section structure ([Intro], [Verse], [Chorus], [Outro]).

---

### User Story 4 - Export Model for Deployment (Priority: P2)

As a developer, I want to export the trained LoRA model to ONNX format so that it can be integrated with the lofi.nvim daemon and distributed via HuggingFace.

**Why this priority**: Export is required for practical use but only valuable after successful training. Can be developed in parallel with training.

**Independent Test**: Can be tested by merging LoRA weights and exporting to ONNX, then running inference on the exported model.

**Acceptance Scenarios**:

1. **Given** a trained LoRA checkpoint, **When** I run the merge process, **Then** the LoRA weights are successfully merged into the base ACE-Step model.

2. **Given** a merged model, **When** I export to ONNX format, **Then** the exported model can be loaded and produces valid audio output.

3. **Given** an ONNX model, **When** configured in lofi.nvim, **Then** the daemon successfully loads and uses the model for generation.

---

### User Story 5 - Optimize Model Size (Priority: P3)

As a developer, I want to reduce the model size through quantization so that users can deploy the model on consumer hardware with reasonable storage requirements.

**Why this priority**: Size optimization improves accessibility but is not required for core functionality. A working full-precision model delivers value.

**Independent Test**: Can be tested by quantizing a model and comparing quality metrics (FAD, CLAP) against the original.

**Acceptance Scenarios**:

1. **Given** an FP32 model (~7.7GB), **When** I convert to FP16, **Then** the model size is approximately 3.9GB with no quality degradation.

2. **Given** an FP16 model, **When** I apply INT8 mixed-precision quantization, **Then** the model size is approximately 2.1GB with less than 5% FAD increase.

3. **Given** a quantized model, **When** I run A/B comparison tests, **Then** the quantized model wins or ties at least 40% of blind listening comparisons.

---

### User Story 6 - Validate Model Quality (Priority: P2)

As a developer, I want to systematically evaluate model quality so that I can select the best checkpoint and ensure the model meets quality standards before release.

**Why this priority**: Quality validation is essential for a successful release but can be developed alongside training.

**Independent Test**: Can be tested by running validation suite on any checkpoint and receiving quality metrics.

**Acceptance Scenarios**:

1. **Given** a trained checkpoint, **When** I run the validation suite, **Then** I receive FAD score, CLAP alignment score, and generated sample files.

2. **Given** multiple checkpoints, **When** I compare quality metrics, **Then** I can identify the best-performing checkpoint.

3. **Given** the best checkpoint, **When** generating samples across all 30+ style categories, **Then** each style produces recognizably distinct output matching the style taxonomy.

---

### User Story 7 - Organize Training Data by Style Category (Priority: P3)

As a developer, I want to organize my training data by lofi style category so that I can ensure balanced representation across all target styles.

**Why this priority**: Organization improves data management but manual categorization can work for initial training.

**Independent Test**: Can be tested by organizing a batch of tracks and verifying correct categorization and metadata tracking.

**Acceptance Scenarios**:

1. **Given** a track with style metadata, **When** I run the organization script, **Then** the track is placed in the correct category directory (core, seasonal, thematic, instrumental, vocal).

2. **Given** a metadata file for tracking, **When** I add a new track, **Then** the source licensing information is recorded with track ID, provider, license type, and cost.

3. **Given** a completed dataset, **When** I query the style distribution, **Then** I can see track counts per category to identify underrepresented styles.

---

### Edge Cases

- What happens when a track has no clear style category? (Default to CORE-01 with manual review flag)
- How does the system handle corrupted audio files? (Fail gracefully with clear error message, skip file)
- What happens if training diverges (loss explodes)? (Automatic early stopping; auto-resume from last good checkpoint with manual override option to select specific checkpoint)
- How does the system handle insufficient VRAM? (Reduce LoRA rank, enable gradient checkpointing)
- What happens if ONNX export fails mid-process? (Checkpoint merged weights, allow resume)
- How are duplicate tracks detected? (Audio fingerprinting with warning, not automatic rejection)
- What happens if ACE-Step base model is unavailable? (Fail fast with specific error message and remediation steps; do not attempt partial operation)

## Requirements *(mandatory)*

### Functional Requirements

#### Data Processing
- **FR-001**: System MUST process audio files in MP3, WAV, FLAC, M4A, and OGG formats
- **FR-002**: System MUST convert all audio to 48kHz sample rate stereo at 320kbps MP3
- **FR-003**: System MUST reject audio files shorter than 30 seconds with status message
- **FR-004**: System MUST trim audio files longer than 240 seconds to 240 seconds
- **FR-005**: System MUST normalize audio peaks to -1dB
- **FR-006**: System MUST detect and reject files with excessive clipping (>0.1% of samples)
- **FR-007**: System MUST detect and reject files with excessive silence (>50% of track)
- **FR-008**: System MUST generate a quality score (0-100) for each processed file

#### Prompt Generation
- **FR-009**: System MUST support 38 style presets across 5 categories (core, seasonal, thematic, instrumental, vocal)
- **FR-010**: System MUST generate prompts following the tag structure: genre, mood, instruments, tempo, production, atmosphere, vocal_type
- **FR-011**: System MUST create `{name}_prompt.txt` files with comma-separated tags
- **FR-012**: System MUST create `{name}_lyrics.txt` files with `[Instrumental]` or structured lyrics

#### Training
- **FR-012a**: System MUST validate ACE-Step base model availability at startup and fail fast with diagnostic error if unavailable
- **FR-013**: System MUST support LoRA training with configurable rank (r) up to 256, lora_alpha=32, and use_rslora=true
- **FR-014**: System MUST save checkpoints at configurable intervals (default: every 2,000 steps per ACE-Step trainer)
- **FR-015**: System MUST generate sample audio at configurable intervals for quality monitoring
- **FR-016**: System MUST support mixed precision training via `--precision bf16` flag (requires explicit configuration; ACE-Step defaults to fp32)
- **FR-017**: System MUST support gradient clipping to prevent training divergence
- **FR-018**: System MUST log training metrics for monitoring
- **FR-018a**: System MUST automatically resume training from last valid checkpoint after failure, with option for manual checkpoint selection

#### Validation
- **FR-019**: System MUST calculate Fréchet Audio Distance (FAD) against validation set
- **FR-020**: System MUST calculate CLAP alignment score for prompt-to-audio similarity
- **FR-021**: System MUST generate test samples across all style categories per checkpoint
- **FR-022**: System MUST track best checkpoint by combined quality score

#### Export
- **FR-023**: System MUST merge LoRA weights into base ACE-Step model
- **FR-024**: System MUST export merged model to ONNX format with opset version 17+
- **FR-025**: System MUST support FP16, INT8, and INT4 precision exports
- **FR-026**: System MUST preserve lyric encoder and speaker embedder at INT8 minimum precision

#### Data Management
- **FR-027**: System MUST maintain source metadata (provider, license, cost, date) per track
- **FR-028**: System MUST support style category assignment per track
- **FR-029**: System MUST track quality scores from human evaluation
- **FR-030**: System MUST separate validation holdout set (10% of data)

### Key Entities

- **Track**: Individual audio file with associated prompt, lyrics, style categories, source metadata, and quality scores
- **Style Preset**: Predefined tag combinations for each of 38 style categories, including base tags, instruments, and production characteristics
- **Checkpoint**: Training snapshot containing LoRA weights, training step, and associated quality metrics
- **Dataset**: Collection of processed tracks organized by style category with validation split
- **Source Record**: Licensing and provenance information including provider, license type, cost, and acquisition date

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Trained model achieves FAD score less than 5.0 against held-out lofi validation set
- **SC-002**: CLAP alignment score exceeds 0.7 for prompt-to-audio similarity
- **SC-003**: Human evaluators cannot distinguish AI-generated tracks from real lofi tracks at better than 55% accuracy (p > 0.05)
- **SC-004**: All 30+ style categories produce recognizably distinct outputs when prompted
- **SC-005**: Vocal/lyric alignment score exceeds 0.8 for tracks with lyrics
- **SC-006**: Audio generation completes within 5 seconds per minute of output (on reference hardware)
- **SC-007**: Exported ONNX model loads and runs successfully in lofi.nvim daemon
- **SC-008**: INT8 quantized model size is 2.5GB or less (73%+ reduction from baseline)
- **SC-009**: INT8 quantized model maintains 95% or better of baseline FAD score
- **SC-010**: Training completes 100,000 steps without divergence on a 500-track dataset

## Assumptions

- ACE-Step base model and training infrastructure are available and functional
- ACE-Step SSL alignment models are available: MERT (`m-a-p/MERT-v1-330M`) and mHuBERT (`utter-project/mHuBERT-147`) - downloaded automatically by trainer.py
- RTX 4090 (24GB VRAM) or equivalent is available for training
- 64GB+ system RAM is available for dataset preprocessing
- 2TB+ NVMe storage is available for raw audio and processed datasets
- All training data will be legally licensed for AI training purposes
- FFmpeg is available for audio format conversion
- The lofi.nvim daemon architecture supports custom LoRA loading
- HuggingFace Hub is available for model distribution

## Out of Scope

- Automatic music composition or arrangement
- Real-time audio generation during training
- Automatic licensing verification or copyright detection
- Web-based UI for training management
- Multi-GPU distributed training
- Automatic data acquisition or web scraping
- Subjective quality judgment (human review required for style accuracy)
