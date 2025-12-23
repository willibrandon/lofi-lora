# Tasks: Lofi LoRA Training Pipeline

**Input**: Design documents from `/specs/001-lofi-lora-training/`
**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Can run in parallel (different files, no dependencies)
- **[Story]**: Which user story this task belongs to (e.g., US1, US2, US3)
- Include exact file paths in descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization and basic structure

- [ ] T001 Create project directory structure per plan.md layout (scripts/, config/, data/, tests/)
- [ ] T002 Create requirements.txt with pinned dependencies from research.md
- [ ] T003 [P] Create config/lofi_lora_config.json with LoRA hyperparameters (r=256, lora_alpha=32, use_rslora=true, target_modules per ACE-Step)
- [ ] T004 [P] Create config/training_defaults.json with default training parameters
- [ ] T005 [P] Create pytest.ini and tests/__init__.py for test infrastructure
- [ ] T006 [P] Create tests/fixtures/sample_audio/ with short test audio files

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**CRITICAL**: No user story work can begin until this phase is complete

- [ ] T007 Create config/schemas/track.json with Track entity JSON schema from data-model.md
- [ ] T008 [P] Create config/schemas/style_preset.json with StylePreset entity JSON schema
- [ ] T009 [P] Create config/schemas/source_record.json with SourceRecord entity JSON schema
- [ ] T010 [P] Create config/schemas/checkpoint.json with Checkpoint entity JSON schema
- [ ] T011 [P] Create config/schemas/dataset.json with Dataset entity JSON schema
- [ ] T012 Create config/style_presets.json with all 38 style presets across 5 categories (core, seasonal, thematic, instrumental, vocal)
- [ ] T013 Create data/metadata/sources.json skeleton with source tracking structure
- [ ] T014 [P] Create data/metadata/style_mapping.json skeleton for track-to-style assignments
- [ ] T015 [P] Create data/metadata/quality_scores.json skeleton for human evaluation scores

**Checkpoint**: Foundation ready - user story implementation can now begin in parallel

---

## Phase 3: User Story 2 - Prepare Training Data (Priority: P1) - MVP

**Goal**: Process and prepare audio files for training so dataset meets ACE-Step requirements

**Independent Test**: Process a batch of raw audio files and verify all outputs meet format requirements (duration, sample rate, normalization)

### Implementation for User Story 2

- [ ] T016 [P] [US2] Create scripts/lofi_audio_processor.py with CLI argument parsing per contracts/cli-interface.md
- [ ] T017 [US2] Implement audio loading function supporting MP3, WAV, FLAC, M4A, OGG formats in scripts/lofi_audio_processor.py (FR-001)
- [ ] T018 [US2] Implement audio resampling to 48kHz stereo in scripts/lofi_audio_processor.py (FR-002)
- [ ] T019 [US2] Implement duration validation (30-240 seconds) with trim/skip logic in scripts/lofi_audio_processor.py (FR-003, FR-004)
- [ ] T020 [US2] Implement peak normalization to -1dB in scripts/lofi_audio_processor.py (FR-005)
- [ ] T021 [US2] Implement clipping detection (>0.1% samples) in scripts/lofi_audio_processor.py (FR-006)
- [ ] T022 [US2] Implement silence detection (>50% track) in scripts/lofi_audio_processor.py (FR-007)
- [ ] T023 [US2] Implement quality score calculation (0-100) in scripts/lofi_audio_processor.py (FR-008)
- [ ] T024 [US2] Implement FFmpeg MP3 export (320kbps) in scripts/lofi_audio_processor.py
- [ ] T025 [US2] Implement batch processing with worker parallelism in scripts/lofi_audio_processor.py
- [ ] T026 [US2] Implement JSON output format and exit codes per contracts/cli-interface.md in scripts/lofi_audio_processor.py
- [ ] T027 [US2] Implement validate subcommand for single file quality assessment in scripts/lofi_audio_processor.py

**Checkpoint**: Audio processing complete - raw audio can be converted to training-ready format

---

## Phase 4: User Story 3 - Generate Style-Consistent Prompts (Priority: P2)

**Goal**: Generate consistent and comprehensive prompt tags for each track based on style taxonomy

**Independent Test**: Generate prompts for a set of categorized tracks and verify tag consistency across the style taxonomy

### Implementation for User Story 3

- [ ] T028 [P] [US3] Create scripts/lofi_prompt_generator.py with CLI argument parsing per contracts/cli-interface.md
- [ ] T029 [US3] Implement style preset loading with inheritance (extends field) in scripts/lofi_prompt_generator.py
- [ ] T030 [US3] Implement tag structure generation (genre, mood, instruments, tempo, production, atmosphere, vocal_type) in scripts/lofi_prompt_generator.py (FR-010)
- [ ] T031 [US3] Implement {name}_prompt.txt file generation with comma-separated tags in scripts/lofi_prompt_generator.py (FR-011)
- [ ] T032 [US3] Implement {name}_lyrics.txt file generation with [Instrumental] or structured lyrics in scripts/lofi_prompt_generator.py (FR-012)
- [ ] T033 [US3] Implement batch prompt generation for audio directory in scripts/lofi_prompt_generator.py
- [ ] T034 [US3] Implement interactive mode for single file prompt creation in scripts/lofi_prompt_generator.py

**Checkpoint**: Prompt generation complete - audio files have associated training metadata

---

## Phase 5: User Story 1 - Train Custom Lofi LoRA Model (Priority: P1) - MVP

**Goal**: Train a custom LoRA fine-tuned on lofi music to generate high-quality lofi beats

**Independent Test**: Run training pipeline on small dataset (10-20 tracks) and verify model produces lofi-characteristic audio

### Implementation for User Story 1

- [ ] T035 [P] [US1] Create scripts/convert_dataset.py with CLI argument parsing per contracts/cli-interface.md
- [ ] T036 [US1] Implement HuggingFace dataset generation from audio/prompt/lyrics files in scripts/convert_dataset.py
- [ ] T037 [US1] Implement validation split (10%) with seed control in scripts/convert_dataset.py (FR-030)
- [ ] T038 [US1] Implement repeat count for data augmentation in scripts/convert_dataset.py
- [ ] T039 [US1] Implement dataset hash generation for reproducibility in scripts/convert_dataset.py
- [ ] T040 [US1] Implement JSON output with train/validation sample counts in scripts/convert_dataset.py
- [ ] T041 [P] [US1] Create scripts/run_training.sh wrapper script that invokes ACE-Step's trainer.py with lofi-specific config per contracts/cli-interface.md
- [ ] T042 [US1] Implement ACE-Step base model validation with fail-fast diagnostic before invoking trainer.py (FR-012a)
- [ ] T043 [US1] Implement LoRA configuration loading (r=256, lora_alpha=32, use_rslora=true, target modules) via --lora_config_path flag (FR-013)
- [ ] T044 [US1] Configure checkpoint saving interval via --every_n_train_steps (default: 2000) (FR-014)
- [ ] T045 [US1] Configure sample audio generation interval via --every_plot_step (FR-015)
- [ ] T046 [US1] Configure mixed precision training via --precision bf16 flag (FR-016)
- [ ] T047 [US1] Configure gradient clipping via --gradient_clip_val 0.5 flag (FR-017)
- [ ] T048 [US1] Configure training metrics logging via --logger_dir (FR-018)
- [ ] T049 [US1] Implement automatic resume from last valid checkpoint via --ckpt_path flag (FR-018a)
- [ ] T050 [US1] Implement manual checkpoint selection via --ckpt_path pointing to specific checkpoint (FR-018a)

**Checkpoint**: Training pipeline complete - can train LoRA models and save checkpoints

---

## Phase 6: User Story 6 - Validate Model Quality (Priority: P2)

**Goal**: Systematically evaluate model quality to select best checkpoint before release

**Independent Test**: Run validation suite on any checkpoint and receive quality metrics

### Implementation for User Story 6

- [ ] T051 [P] [US6] Create scripts/validate_checkpoint.py with CLI argument parsing per contracts/cli-interface.md
- [ ] T052 [US6] Implement checkpoint loading and integrity verification in scripts/validate_checkpoint.py
- [ ] T053 [US6] Implement FAD score calculation against validation set in scripts/validate_checkpoint.py (FR-019)
- [ ] T054 [US6] Implement CLAP alignment score calculation in scripts/validate_checkpoint.py (FR-020)
- [ ] T055 [US6] Implement test sample generation across all style categories in scripts/validate_checkpoint.py (FR-021)
- [ ] T056 [US6] Implement checkpoint comparison for best selection in scripts/validate_checkpoint.py (FR-022)
- [ ] T057 [US6] Implement JSON output with FAD, CLAP, style coverage metrics in scripts/validate_checkpoint.py
- [ ] T058 [US6] Implement compare subcommand for multi-checkpoint evaluation in scripts/validate_checkpoint.py

**Checkpoint**: Validation complete - can evaluate and compare checkpoint quality

---

## Phase 7: User Story 4 - Export Model for Deployment (Priority: P2)

**Goal**: Export trained LoRA model to ONNX format for lofi.nvim integration

**Independent Test**: Merge LoRA weights, export to ONNX, run inference on exported model

### Implementation for User Story 4

- [ ] T059 [P] [US4] Create scripts/merge_lora.py with CLI argument parsing per contracts/cli-interface.md
- [ ] T060 [US4] Implement LoRA weight merging into base ACE-Step model in scripts/merge_lora.py (FR-023)
- [ ] T061 [US4] Implement merge verification with inference test in scripts/merge_lora.py
- [ ] T062 [US4] Implement exit codes per contracts/cli-interface.md in scripts/merge_lora.py
- [ ] T063 [P] [US4] Create scripts/export_onnx.py with CLI argument parsing per contracts/cli-interface.md
- [ ] T064 [US4] Implement ONNX export with opset 17+ in scripts/export_onnx.py (FR-024)
- [ ] T065 [US4] Implement dynamic axes for variable sequence length in scripts/export_onnx.py
- [ ] T066 [US4] Implement export verification with inference test in scripts/export_onnx.py

**Checkpoint**: Export complete - model can be deployed to lofi.nvim daemon

---

## Phase 8: User Story 5 - Optimize Model Size (Priority: P3)

**Goal**: Reduce model size through quantization for consumer hardware deployment

**Independent Test**: Quantize model and compare FAD/CLAP metrics against original

### Implementation for User Story 5

- [ ] T067 [US5] Implement FP16 precision export in scripts/export_onnx.py (FR-025)
- [ ] T068 [US5] Implement INT8 mixed-precision quantization in scripts/export_onnx.py (FR-025)
- [ ] T069 [US5] Implement INT4 quantization with calibration data in scripts/export_onnx.py (FR-025)
- [ ] T070 [US5] Implement lyric encoder/speaker embedder INT8 minimum precision preservation in scripts/export_onnx.py (FR-026)
- [ ] T071 [US5] Implement calibration data loading for static quantization in scripts/export_onnx.py
- [ ] T072 [US5] Implement size and quality reporting after quantization in scripts/export_onnx.py

**Checkpoint**: Quantization complete - model can be exported at various precision tiers

---

## Phase 9: User Story 7 - Organize Training Data by Style Category (Priority: P3)

**Goal**: Organize training data by lofi style category for balanced representation

**Independent Test**: Organize batch of tracks and verify correct categorization and metadata tracking

### Implementation for User Story 7

- [ ] T073 [US7] Implement track style assignment in data/metadata/style_mapping.json updates (FR-028)
- [ ] T074 [US7] Implement source metadata recording (provider, license, cost, date) in data/metadata/sources.json updates (FR-027)
- [ ] T075 [US7] Implement quality score tracking from human evaluation in data/metadata/quality_scores.json updates (FR-029)
- [ ] T076 [US7] Implement style distribution query showing track counts per category
- [ ] T077 [US7] Implement directory organization by category (core, seasonal, thematic, instrumental, vocal)

**Checkpoint**: Data organization complete - dataset has balanced style representation with full provenance

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Improvements that affect multiple user stories

- [ ] T078 [P] Add --help documentation to all scripts
- [ ] T079 [P] Add --version flag to all scripts
- [ ] T080 [P] Add --verbose and --quiet flags to all scripts per contracts/cli-interface.md
- [ ] T081 [P] Add --dry-run flag to all scripts per contracts/cli-interface.md
- [ ] T082 Validate environment variables (LOFI_DATA_DIR, ACE_STEP_DIR, etc.) across scripts per contracts/cli-interface.md
- [ ] T083 Run quickstart.md validation with 10-track sample dataset
- [ ] T084 Validate checkpoint manifest (checkpoints/manifest.json) tracking across training and validation

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies - can start immediately
- **Foundational (Phase 2)**: Depends on Setup completion - BLOCKS all user stories
- **User Stories (Phase 3+)**: All depend on Foundational phase completion
  - User stories can then proceed in parallel (if staffed)
  - Or sequentially in priority order (P1 -> P2 -> P3)
- **Polish (Final Phase)**: Depends on all desired user stories being complete

### User Story Dependencies

- **User Story 2 (P1)**: Can start after Foundational - Prerequisite for US1
- **User Story 3 (P2)**: Can start after Foundational - Enhances US2 output
- **User Story 1 (P1)**: Depends on US2 (needs processed audio) - Core training
- **User Story 6 (P2)**: Depends on US1 (needs checkpoints) - Validation
- **User Story 4 (P2)**: Depends on US1 (needs trained model) - Export
- **User Story 5 (P3)**: Depends on US4 (needs ONNX model) - Quantization
- **User Story 7 (P3)**: Can start after Foundational - Independent organization

### Critical Path

```
Setup -> Foundational -> US2 (Audio Processing) -> US1 (Training) -> US6 (Validation) -> US4 (Export) -> US5 (Quantization)
```

### Parallel Opportunities

**Within Setup (Phase 1)**:
- T003, T004, T005, T006 can run in parallel

**Within Foundational (Phase 2)**:
- T008, T009, T010, T011 can run in parallel (JSON schemas)
- T014, T015 can run in parallel (metadata files)

**Within User Story Phases**:
- T016 and T028 can run in parallel (different scripts)
- T035 and T041 can run in parallel (dataset converter and training script)
- T051 and T059, T063 can run in parallel (validation and export scripts)
- T078, T079, T080, T081 can run in parallel (documentation across different scripts)

---

## Parallel Example: Phase 2 (Foundational)

```bash
# Launch all JSON schema creation tasks in parallel:
Task: "Create config/schemas/style_preset.json with StylePreset entity JSON schema"
Task: "Create config/schemas/source_record.json with SourceRecord entity JSON schema"
Task: "Create config/schemas/checkpoint.json with Checkpoint entity JSON schema"
Task: "Create config/schemas/dataset.json with Dataset entity JSON schema"
```

## Parallel Example: User Story 2 + 3 Start

```bash
# After Foundational completes, launch both script skeletons:
Task: "Create scripts/lofi_audio_processor.py with CLI argument parsing"
Task: "Create scripts/lofi_prompt_generator.py with CLI argument parsing"
```

---

## Implementation Strategy

### MVP First (User Story 2 + 1 Only)

1. Complete Phase 1: Setup
2. Complete Phase 2: Foundational (CRITICAL - blocks all stories)
3. Complete Phase 3: User Story 2 (Audio Processing)
4. Complete Phase 5: User Story 1 (Training)
5. **STOP and VALIDATE**: Test with 10-track sample dataset per quickstart.md
6. If checkpoints generate, MVP is functional

### Incremental Delivery

1. Complete Setup + Foundational -> Foundation ready
2. Add US2 (Audio Processing) -> Can process raw audio
3. Add US3 (Prompt Generation) -> Can create training metadata
4. Add US1 (Training) -> Can train LoRA models (MVP!)
5. Add US6 (Validation) -> Can evaluate quality
6. Add US4 (Export) -> Can deploy to lofi.nvim
7. Add US5 (Quantization) -> Can optimize for consumer hardware
8. Add US7 (Organization) -> Can manage large datasets

### Suggested MVP Scope

**MVP = Phase 1 + Phase 2 + Phase 3 (US2) + Phase 5 (US1)**

This delivers:
- Audio processing pipeline (converts raw files to training-ready format)
- Training pipeline (produces LoRA checkpoints)
- Basic validation (checkpoint files exist and load)

Total MVP tasks: ~35 tasks (T001-T015 setup/foundational + T016-T027 US2 + T035-T050 US1)

---

## Notes

- [P] tasks = different files, no dependencies
- [Story] label maps task to specific user story for traceability
- Each user story should be independently completable and testable
- Commit after each task or logical group
- Stop at any checkpoint to validate story independently
- US2 must precede US1 (training needs processed audio)
- US1 must precede US6, US4, US5 (need trained model)
