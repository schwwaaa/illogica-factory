# Illogica Factory
## Local-First Hybrid Reconstruction Milestone Specification

**Version:** 1.0  
**Date:** July 29, 2026  
**Purpose:** Define a modular, local-first reconstruction plan for Illogica Factory, with optional remote APIs and cloud providers available at any stage of the workflow.

---

## 1. Product Definition

Illogica Factory is a modular system for generating complete vertical audiovisual works from a prompt, seed, vocabulary set, or autonomous workflow.

A completed generation may include:

- A concept or subject
- An intentionally unusual dialect
- A narration script
- A title
- A description
- Keywords and tags
- Selected or generated visual material
- Speech synthesis
- Timed captions
- Music or sound
- A rendered 9:16 video
- A complete generation manifest
- Optional publication metadata

The system supports two primary operating modes.

### One Shot

The user provides a subject, prompt, or source material and requests one completed video.

### Factory Run

The user creates multiple related outputs from an initial prompt, recipe, word set, or workflow.

A Factory Run must be finite by default:

- Number of variations
- Maximum runtime
- Maximum storage
- Optional stopping conditions

Continuous unattended generation can eventually exist as a separate operating mode, but it should not be the foundation of the initial reconstruction.

---

## 2. Core Architectural Principle

The application must separate three concepts that were intertwined in the original codebase:

1. **Workflow**
2. **Provider**
3. **Job**

### Workflow

A workflow defines what happens and in what order.

Example:

```text
Select Words
    ↓
Construct Contradictory Concept
    ↓
Mutate Dialect
    ↓
Write Script
    ↓
Generate Search Terms
    ↓
Select Local Media
    ↓
Generate Voice
    ↓
Create Captions
    ↓
Render Video
```

### Provider

A provider defines which implementation performs a workflow step.

Examples:

```text
Script Provider:
- Local llama.cpp
- Local Ollama
- OpenAI API
- Anthropic API
- Cloud-hosted open model
- Deterministic grammar engine

Speech Provider:
- Local Piper
- Operating-system voice
- ElevenLabs
- Cloud TTS service

Media Provider:
- Local media library
- Pexels
- User-selected files
- Generated images
- Generated video
```

### Job

A job is one execution of a workflow using a specific configuration, seed, provider selection, and set of source files.

Every job must be independently reproducible, inspectable, restartable, and cancelable.

---

## 3. Recommended System Structure

```text
Desktop Interface
       │
       ▼
Job and Project Manager
       │
       ▼
Workflow Runtime
       │
       ├── Seed and Concept Nodes
       ├── Language Nodes
       ├── Media Nodes
       ├── Audio Nodes
       ├── Caption Nodes
       └── Render Nodes
       │
       ▼
Provider Registry
       │
       ├── Local Providers
       ├── Remote API Providers
       ├── Cloud Model Providers
       └── Deterministic Algorithms
       │
       ▼
Artifact Store and Job Manifest
```

### Suggested implementation split

#### Desktop shell and supervisor

A Tauri desktop application is appropriate for:

- Project management
- Job controls
- Workflow configuration
- Provider selection
- Progress monitoring
- File management
- Secure credential storage
- Application packaging

#### Processing runtime

The processing runtime should operate independently from the interface.

It should support:

- Headless execution
- Restarting failed jobs
- Queue management
- Separate worker processes
- Cancellation
- Progress events
- Logging
- Local and remote providers

A practical initial division would be:

```text
Rust or native supervisor
    ├── Job database
    ├── Queue
    ├── File management
    ├── Provider configuration
    └── Worker process control

Python processing worker
    ├── Local model integrations
    ├── NLP
    ├── Speech tools
    ├── Caption alignment
    ├── Media analysis
    └── Workflow-node execution

FFmpeg renderer
    └── Final deterministic media assembly
```

This avoids forcing all machine-learning integrations into Rust while still providing a stable native application foundation.

The worker protocol should be language-neutral so Python components can later be replaced without changing the application.

---

## 4. Provider Contract

Every replaceable processing stage should implement a common provider contract.

Conceptually:

```text
Provider
├── provider_id
├── provider_type
├── capabilities
├── configuration_schema
├── availability_check
├── estimated_resource_use
├── execute
├── cancel
└── health_status
```

Each execution receives:

```text
ProviderRequest
├── job_id
├── node_id
├── input_artifacts
├── parameters
├── random_seed
├── timeout
└── privacy_policy
```

Each provider returns:

```text
ProviderResult
├── status
├── output_artifacts
├── structured_data
├── logs
├── model_information
├── usage_information
├── warnings
└── reproducibility_information
```

The workflow should not care whether the provider is:

- A local executable
- A Python function
- A local HTTP service
- A cloud API
- A deterministic algorithm
- Another Illogica process on the network

---

# 5. Milestone Sequence

## Milestone 0 — Archaeological Freeze and Behavioral Specification

### Goal

Preserve the original project’s useful behavior before replacing its implementation.

### Work

- Archive the original repository unchanged.
- Preserve the original example videos.
- Preserve existing JSON generation records.
- Document the original prompt chain.
- Extract the NLTK vocabulary process.
- Record all known caption, voice, media, and render settings.
- Identify representative examples of:
  - Conventional output
  - Strong Illogica dialect
  - Weak Illogica dialect
  - Successful visual mismatch
  - Unsuccessful generation
- Create a behavioral test set from historical outputs.

### Deliverables

```text
research/
├── original-prompts/
├── historical-manifests/
├── example-scripts/
├── example-titles/
├── vocabulary-samples/
├── video-analysis/
└── behavioral-spec.md
```

### Completion test

A new developer can explain exactly how the old language and videos were created without running the old application.

---

## Milestone 1 — Core Data Model and Generation Manifest

### Goal

Define the information exchanged by all future modules.

### Work

Create stable schemas for:

- Project
- Workflow
- Workflow node
- Provider
- Provider profile
- Job
- Variation group
- Artifact
- Prompt revision
- Media source
- Caption track
- Render profile
- Publication metadata

Every job receives a persistent manifest.

### Minimum manifest contents

```text
Job identity
Workflow identity and version
Parent job or variation group
Initial user prompt
Random seed
Derived seeds
Selected words
All intermediate prompts
All model responses
Provider identifiers
Exact model names
Local model hashes
Sampling parameters
Media file hashes
Voice information
Caption information
Render settings
Application version
Start and finish times
Errors and warnings
```

### Completion test

A manually created manifest can describe one of the historical example videos without losing meaningful provenance.

---

## Milestone 2 — Runtime, Queue, and Artifact Store

### Goal

Create a reliable execution foundation before adding generative features.

### Work

Implement:

- Persistent job database
- Sequential queue
- Pause and resume
- Cancel
- Retry individual node
- Restart interrupted job
- Per-job working directory
- Immutable completed artifacts
- Structured logs
- Progress events
- Disk-space checks
- Resource limits
- Temporary-file cleanup
- Crash recovery

### Required isolation

Each job must have its own directory:

```text
projects/<project-id>/jobs/<job-id>/
├── manifest.json
├── inputs/
├── intermediate/
├── media/
├── audio/
├── captions/
├── renders/
└── logs/
```

No stage may depend on a global temporary directory.

### Completion test

Three jobs can be queued, the application can be closed during the second job, and processing can resume without corrupting any job.

---

## Milestone 3 — Workflow Runtime and Built-In Nodes

### Goal

Execute a simple workflow without depending on an AI model.

### Initial nodes

- Static Text Input
- Random Word Sampler
- Template Formatter
- Text File Writer
- Local File Selector
- Audio File Input
- Caption File Input
- FFmpeg Render
- Metadata Writer
- Output Collector

### Workflow requirements

- Directed acyclic graph for initial releases
- Typed inputs and outputs
- Node validation before execution
- Node-level retry
- Node caching
- Clear error propagation
- Workflow versioning
- Workflow duplication

### Completion test

A workflow can take user-provided text, video, audio, and captions and produce a valid 9:16 video without using any AI provider.

This creates a known-good pipeline before generative components are introduced.

---

## Milestone 4 — Local Language Model Foundation

### Goal

Generate concepts, scripts, and metadata entirely locally.

### Initial local provider capabilities

- Text completion
- Chat completion
- Structured JSON generation
- Configurable temperature
- Configurable top-p and top-k
- Seed control where supported
- Context-size detection
- Model availability check
- Model metadata capture
- Timeout and cancellation

### Initial provider types

Support at least one local model runtime first.

The interface should later support:

- Ollama-compatible local servers
- llama.cpp-compatible local servers
- Direct local subprocess execution
- User-defined OpenAI-compatible endpoints

### Initial language nodes

- Concept Generator
- Script Generator
- Title Generator
- Description Generator
- Keyword Generator
- Search-Term Generator
- Structured Response Repair

### Completion test

The application can produce a complete structured script package while disconnected from the internet.

---

## Milestone 5 — Illogica Dialect Engine Version 1

### Goal

Reconstruct the historical language behavior as a named, versioned workflow recipe.

This must not be hidden inside one general-purpose prompt.

### Initial historical recipe

```text
Obscure Word Sampler
    ↓
Semantic Collision Prompt
    ↓
Pseudo-Educational Concept
    ↓
Dialect Preservation Pass
    ↓
Narration Expansion
    ↓
Title and Metadata Generation
```

### Required controls

- Vocabulary corpus
- Number of sampled words
- Word rarity
- Temperature
- Sentence count
- Factuality pressure
- Coherence pressure
- Contradiction pressure
- Academic tone
- Mythological tone
- Taxonomic tone
- Technical tone
- Preservation of uncommon words
- Degree of grammatical damage
- Degree of semantic discontinuity

### Corpus support

- Historical NLTK corpus behavior
- Custom word lists
- User text corpus
- Scientific terminology
- Archaic language
- Regional vocabulary
- Invented vocabulary
- Combined weighted corpora

### Important design rule

The system should preserve intermediate text between passes.

A later model must not silently “correct” the strange language unless the workflow explicitly requests correction.

### Completion test

A blinded comparison identifies the reconstructed outputs as belonging to the same family as the historical Illogica examples.

Exact sentence reproduction is not required. Behavioral resemblance is.

---

## Milestone 6 — Local Media Library

### Goal

Remove stock-media APIs from the required path.

### Library capabilities

- Import local folders
- Watch folders for new content
- Generate thumbnails
- Read duration, resolution, frame rate, and codec
- Detect portrait and landscape orientation
- Add user tags
- Store source and licensing information
- Prevent accidental modification of originals
- Track usage across generated videos

### Selection modes

1. Random
2. Tag based
3. Filename based
4. Duration based
5. Color or motion based
6. Text similarity based
7. Intentional semantic mismatch
8. User-selected pool
9. Avoid recently used material

### Local media analysis

Media analysis should be optional and modular.

Possible later analyzers:

- Image embeddings
- Video embeddings
- Object detection
- Scene detection
- Motion estimation
- Dominant-color extraction
- Optical-flow intensity
- Audio energy

### Completion test

The system can generate ten videos from a local media collection without requesting or downloading external footage.

---

## Milestone 7 — Local Speech and Audio

### Goal

Generate narration without a remote service.

### Provider contract

Speech providers receive:

- Script
- Voice
- Speed
- Pitch where supported
- Language
- Output format
- Optional sentence boundaries
- Optional word timing request

### Initial local support

- At least one offline speech provider
- Imported narration files
- Operating-system voice as an optional fallback
- Local music and sound-effect folders

### Audio processing

- Loudness normalization
- Voice/music ducking
- Fade controls
- Silence trimming
- Optional room tone
- Optional degradation
- Optional pitch or time alteration
- Duration reporting

### Completion test

The system can create intelligible narration, mix it with local music, and export a normalized audio track without internet access.

---

## Milestone 8 — Caption Generation and Timing

### Goal

Create reliable captions from known script and generated speech.

### Caption strategies

- Use timing returned by the speech provider
- Forced alignment against the known script
- Local speech transcription
- Sentence-level estimation fallback
- Manual correction

### Caption presentation

- One word at a time
- Phrase based
- Sentence based
- Karaoke highlighting
- Centered Illogica style
- User-defined typography
- Safe-zone visualization
- Caption position templates
- Animation presets

### Caption artifacts

Store both:

- Semantic caption track
- Rendered caption styling

Example:

```text
captions/
├── narration.words.json
├── narration.srt
├── narration.vtt
└── caption-style.json
```

### Completion test

Captions remain synchronized across several local voices and speech speeds, with no external transcription service.

---

## Milestone 9 — Deterministic 9:16 Renderer

### Goal

Produce reliable finished videos from the pipeline’s artifacts.

### Initial render capabilities

- 1080 × 1920 output
- Configurable frame rate
- H.264 output
- AAC audio
- Media sequencing
- Crop and scale
- Background fill
- Caption rendering
- Music mixing
- Intro and outro controls
- Duration fitting
- Render progress
- Cancellation
- Preview-quality render
- Production-quality render

### Determinism

Given identical:

- Source files
- Manifest
- Seed
- Renderer version
- Workflow version

the renderer should produce the same timeline and materially equivalent output.

### Completion test

The renderer successfully produces the entire historical output format from local assets with no provider dependencies.

---

## Milestone 10 — Complete Local One-Shot Product

### Goal

Join the local modules into the first usable application.

### User flow

```text
Create Project
    ↓
Choose One Shot
    ↓
Enter Prompt
    ↓
Choose Workflow Recipe
    ↓
Choose Local Providers
    ↓
Choose Media Library
    ↓
Preview Plan
    ↓
Generate
    ↓
Review Intermediate Artifacts
    ↓
Render
```

### Required controls

- Prompt
- Workflow
- Model
- Model settings
- Voice
- Media collection
- Caption style
- Music
- Duration target
- Output location
- Random seed
- Privacy summary

### Completion test

A new user can install the application, select local components, and create one complete vertical video while offline.

This is the first major release boundary.

---

## Milestone 11 — Factory Runs and Variation Groups

### Goal

Rebuild loop mode as a controllable production system.

### Factory Run controls

- Number of outputs
- Starting prompt
- Shared seed or independent seeds
- Variation strategy
- Workflow
- Provider profile
- Parallelism limit
- Storage limit
- Failure policy
- Naming pattern
- Stop conditions

### Variation relationships

Every output should identify:

- Parent prompt
- Parent job
- Variation index
- Mutation method
- Mutation strength
- Shared attributes
- Changed attributes

### Initial variation strategies

- Different random words
- Same words, different model seed
- Different dialect intensity
- Different script interpretation
- Different footage
- Different voice
- Different caption style
- Different duration
- Progressive mutation
- Branching mutation

### Completion test

A user can request twelve variations, stop after the fifth, restart later, and understand how every output differs from its siblings.

---

## Milestone 12 — Workflow Recipes

### Goal

Allow workflows to become reusable creative instruments.

### Recipe examples

- Historical Illogica
- Broken Encyclopedia
- Taxonomic Dream
- Technical Mythology
- Semantic Mismatch
- Recursive Translation
- Vocabulary Mutation
- Academic Nonsense
- Conventional Short
- User-Defined Recipe

### Recipe contents

A recipe should store:

- Node graph
- Node settings
- Prompt templates
- Provider capability requirements
- Suggested local models
- Variation rules
- Caption style
- Render profile
- Version
- Author
- Description
- Example outputs

### Completion test

A workflow can be exported, imported, versioned, and used with different compatible providers.

---

## Milestone 13 — Algorithmic Prompt and Workflow Operators

### Goal

Introduce variation without requiring every creative decision to come from a language model.

### Initial algorithmic nodes

#### Vocabulary operators

- Random sampling
- Weighted sampling
- Rare-word preference
- Part-of-speech balancing
- Corpus collision
- Phonetic similarity
- Character mutation
- Syllable recombination

#### Text operators

- Sentence permutation
- Clause replacement
- Controlled deletion
- Controlled repetition
- Grammatical damage
- Punctuation mutation
- Synonym drift
- Antonym collision
- Recursive summarization
- Translation loops

#### Workflow operators

- Choose one of several branches
- Repeat node with altered seed
- Compare two outputs
- Select output by measurable property
- Blend results
- Escalate mutation strength
- Stop when similarity crosses a threshold

### Important restriction

The initial workflow system should remain inspectable.

The application must show:

- What operator ran
- What input it received
- What it changed
- What output it produced

### Completion test

A Factory Run can generate meaningfully different works even when using the same local model throughout.

---

## Milestone 14 — Remote Provider Framework

### Goal

Add cloud capabilities without altering the local workflow model.

Remote support should begin only after the complete local path is reliable.

### Remote provider categories

- Hosted language model
- Hosted speech synthesis
- Hosted transcription
- Stock-media search
- Image generation
- Video generation
- Music generation
- Remote rendering
- Object storage
- Publication platform

### Requirements

- Credentials stored securely
- Provider health check
- Rate-limit handling
- Timeout
- Retry with backoff
- Usage reporting
- Cost reporting where available
- Request and response logging
- Redaction options
- Provider-specific policy warnings
- Offline detection

### Completion test

A local script provider can be replaced with a remote script provider without changing the workflow or downstream nodes.

---

## Milestone 15 — Hybrid Provider Profiles

### Goal

Allow local and remote providers to be mixed at any stage.

### Example profile: Fully Local

```text
Concept: Local LLM
Script: Local LLM
Metadata: Local LLM
Media: Local Library
Voice: Local TTS
Captions: Local Alignment
Render: Local FFmpeg
Publishing: Disabled
```

### Example profile: Local-Private

```text
Concept: Local Algorithm
Script: Local LLM
Metadata: Local LLM
Media: Local Library
Voice: Local TTS
Captions: Local Alignment
Render: Local FFmpeg
```

### Example profile: Hybrid Quality

```text
Concept: Local Illogica Recipe
Script: Cloud LLM
Metadata: Local LLM
Media: Local Library
Voice: Cloud TTS
Captions: Local Alignment
Render: Local FFmpeg
```

### Example profile: Remote Production

```text
Concept: Cloud LLM
Script: Cloud LLM
Metadata: Cloud LLM
Media: Stock API
Voice: Cloud TTS
Captions: Cloud Transcription
Render: Cloud Render Worker
Publishing: YouTube API
```

### Profile controls

- Default provider for each capability
- Approved fallback provider
- Never-send-private-data rule
- Maximum remote cost
- Maximum retries
- Required local-only nodes
- Network availability behavior

### Completion test

A user can select providers independently for every stage and save that combination as a reusable profile.

---

## Milestone 16 — Fallback, Routing, and Capability Negotiation

### Goal

Make hybrid operation resilient rather than merely configurable.

### Routing behaviors

- Local only
- Remote only
- Prefer local
- Prefer remote
- Use remote only after local failure
- Use local only after remote failure
- Ask before switching
- Select cheapest available
- Select fastest available
- Select provider supporting required capability

### Example

```text
Speech Requirement:
- Word timestamps required
- English voice required
- Offline preferred

Routing Result:
1. Local Provider A: available, supports timestamps
2. Local Provider B: available, no timestamps
3. Remote Provider C: available, supports timestamps

Selected:
Local Provider A
```

### Completion test

Disconnecting the internet during a hybrid Factory Run does not corrupt jobs. Eligible nodes fall back locally, while ineligible nodes pause with a clear explanation.

---

## Milestone 17 — Workflow Editor

### Goal

Provide a visual interface for advanced workflow design.

### Editor features

- Node palette
- Typed ports
- Drag-and-drop connections
- Validation
- Provider capability display
- Node preview
- Intermediate artifact viewer
- Run selected node
- Run from selected node
- Branch comparison
- Workflow version history
- Recipe export
- Recipe documentation

### Recommended release strategy

Do not build the graphical editor before the workflow schema and runtime are stable.

Early workflows should be editable through structured configuration and a simpler ordered-stage interface.

### Completion test

A user can duplicate the Historical Illogica recipe, insert a translation-mutation node, change the media-selection strategy, and save it as a new recipe.

---

## Milestone 18 — Review, Curation, and Regeneration

### Goal

Treat generations as editable projects rather than disposable outputs.

### Review capabilities

- Inspect every intermediate result
- Approve or reject nodes
- Edit script
- Replace footage
- Replace one clip
- Regenerate title only
- Regenerate voice only
- Re-time captions
- Change style without regenerating content
- Render alternate versions
- Mark favorite generations
- Compare variation siblings

### Completion test

A user can modify one caption style and rerender without repeating language generation, speech synthesis, or media selection.

---

## Milestone 19 — Model and Recipe Evaluation

### Goal

Measure whether new models preserve the desired creative behavior.

### Evaluation dimensions

- Vocabulary preservation
- Semantic discontinuity
- Syntactic stability
- Phrase novelty
- Unintended conventionalization
- Repetition
- Narration suitability
- Pronounceability
- Caption suitability
- Human artistic preference

### Evaluation tools

- Historical prompt suite
- Side-by-side comparison
- Blind voting
- Similarity metrics
- Vocabulary-retention score
- Repeated-generation analysis
- Model and settings report

### Completion test

A new local or cloud model can be tested against historical Illogica behavior before it is added to a recommended provider profile.

---

## Milestone 20 — Packaging and Deployment

### Goal

Make the local system installable and understandable.

### Packaging targets

- macOS
- Windows
- Linux

### Installation responsibilities

- Application
- Runtime dependencies
- FFmpeg discovery or packaging
- Worker environment
- Local model configuration
- Media-library configuration
- Disk-space estimate
- Hardware capability detection
- Diagnostic report

### Installation levels

#### Core installation

- Application
- Workflow runtime
- Renderer
- Manual media and audio input

#### Local AI installation

- Local model runtime
- Selected language model
- Local speech model
- Optional alignment model

#### Hybrid installation

- Local core
- Remote provider configuration
- Credential management

### Completion test

A clean machine can install the application and successfully run a documented offline example project.

---

## Milestone 21 — Optional Publishing and Automation

### Goal

Add distribution only after generation is dependable.

### Publishing providers

- Local export folder
- YouTube
- Other short-form platforms where supported
- Network storage
- Cloud object storage
- Watched publication folder

### Automation controls

- Never publish automatically
- Save as draft
- Require approval
- Publish approved jobs
- Schedule publishing
- Rate limits
- Duplicate detection
- Metadata preview
- Platform-specific formatting

### Completion test

The system can prepare a complete publication package without requiring publication, and publishing cannot occur without an explicit user-controlled policy.

---

# 6. Recommended Release Boundaries

## Release 0.1 — Pipeline Skeleton

Includes:

- Milestones 0–3
- Job queue
- Manifest
- Static workflow
- Manual source files
- Basic renderer

This proves the architecture.

## Release 0.2 — Local Language

Includes:

- Milestones 4–5
- Local model provider
- Historical Illogica recipe
- Script and metadata output

This proves the dialect can be recreated.

## Release 0.3 — Complete Offline Video

Includes:

- Milestones 6–10
- Local media
- Local speech
- Local captions
- Deterministic rendering
- One Shot interface

This is the first usable local product.

## Release 0.4 — Factory Mode

Includes:

- Milestones 11–13
- Finite Factory Runs
- Variation relationships
- Workflow recipes
- Algorithmic operators

This restores and expands the original autonomous concept.

## Release 0.5 — Hybrid Providers

Includes:

- Milestones 14–16
- Remote adapters
- Mixed provider profiles
- Fallback and routing

This creates the local/cloud hybrid product.

## Release 0.6 — Creative Workbench

Includes:

- Milestones 17–19
- Workflow editor
- Partial regeneration
- Evaluation framework

This converts the generator into a modular artistic system.

## Release 1.0 — Distributable Application

Includes:

- Milestones 20–21
- Cross-platform packaging
- Diagnostics
- Optional publication
- Documentation
- Stable workflow and provider SDK

---

# 7. Initial Provider Matrix

| Capability | Required Local Provider | Later Remote Provider |
|---|---|---|
| Word selection | Built-in corpus engine | Not required |
| Prompt mutation | Built-in algorithms | Hosted model optional |
| Concept generation | Local LLM | Cloud LLM |
| Script generation | Local LLM | Cloud LLM |
| Metadata | Local LLM or templates | Cloud LLM |
| Media | Local library | Stock-media API |
| Image generation | Optional local model | Cloud image provider |
| Video generation | Optional local model | Cloud video provider |
| Speech | Local TTS | Cloud TTS |
| Captions | Local alignment | Cloud transcription |
| Rendering | Local FFmpeg | Remote render worker |
| Storage | Local filesystem | Object storage |
| Publishing | Local export | Platform API |

---

# 8. Configuration Model

Provider selection should exist at three levels.

## Application defaults

The user’s normal providers.

## Project profile

Providers selected for a particular body of work.

## Workflow-node override

A specific node uses a different provider.

Example:

```yaml
profile:
  default_language_provider: local-ollama
  default_speech_provider: local-piper
  default_media_provider: local-library
  default_render_provider: local-ffmpeg

workflow_overrides:
  title_generator:
    provider: cloud-language-provider

  final_voice:
    provider: premium-cloud-tts

privacy:
  allow_remote_text: true
  allow_remote_audio: false
  allow_remote_media: false
```

This allows one remote step without turning the entire job into a cloud workflow.

---

# 9. Modular Workflow Evolution

The project should eventually distinguish between three kinds of workflow intelligence.

## Generative intelligence

A language or media model produces something new.

## Algorithmic intelligence

A deterministic or seeded algorithm modifies, selects, scores, or routes material.

## Curatorial intelligence

A person approves, rejects, combines, or redirects results.

A strong Illogica workflow may combine all three:

```text
Human provides a theme
    ↓
Algorithm samples obscure vocabulary
    ↓
Local model invents a concept
    ↓
Algorithm damages grammar
    ↓
Cloud model expands narration
    ↓
Local selector finds mismatched footage
    ↓
Human approves the script
    ↓
Local speech and renderer complete the video
```

The workflow system should never assume that an AI model must control every creative stage.

---

# 10. Features That Should Not Be Built First

The following would create unnecessary risk during the local reconstruction:

- A public plugin marketplace
- Dynamically loaded native binary plugins
- Multi-user cloud accounts
- Distributed rendering
- Automatic social publication
- Complex timeline editing
- Real-time collaboration
- Unrestricted cyclic workflow graphs
- Automatic model downloading without storage controls
- Parallel generation before job isolation is proven
- A visual node editor before workflow contracts stabilize

Modularity should initially mean:

- Stable interfaces
- Replaceable modules
- Configuration-based selection
- Separate worker processes
- Versioned schemas

It does not initially need to mean arbitrary third-party binary plugins.

---

# 11. Non-Negotiable Engineering Requirements

## Local operation

A complete video must be possible without internet access.

## No global temporary state

Every job owns its own files.

## Reproducibility

Every meaningful generation parameter is recorded.

## Provider independence

No workflow contains provider-specific API logic.

## Failure isolation

One failed node or provider does not corrupt the complete project.

## Partial regeneration

Changing one stage does not require rerunning everything.

## Explicit privacy

The interface shows which material will leave the machine.

## Finite Factory Runs

Autonomous generation has clear limits and stopping controls.

## Historical preservation

The original Illogica recipe remains versioned and available even as new workflows are created.

## Exact model identification

The system records model identifiers, local file hashes, and provider responses whenever possible.

---

# 12. Recommended First Development Track

The most efficient initial track is:

```text
1. Freeze historical behavior
2. Define manifests and provider contracts
3. Build queue and per-job storage
4. Build workflow runtime
5. Build FFmpeg renderer using manual inputs
6. Add one local language provider
7. Reconstruct the Illogica dialect recipe
8. Add local media selection
9. Add local speech
10. Add local caption alignment
11. Release complete offline One Shot
12. Add finite Factory Runs
13. Add algorithmic variation nodes
14. Add remote providers
15. Add hybrid fallback and routing
16. Add graphical workflow editing
```

This order protects the part that matters most: a reliable local artistic instrument.

The cloud layer then becomes an expansion of a working system rather than a dependency holding the system together.
