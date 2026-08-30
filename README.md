<div align="center">

```text
███╗   ██╗ ██████╗ ███████╗███╗   ███╗ █████╗
████╗  ██║██╔═══██╗██╔════╝████╗ ████║██╔══██╗
██╔██╗ ██║██║   ██║█████╗  ██╔████╔██║███████║
██║╚██╗██║██║   ██║██╔══╝  ██║╚██╔╝██║██╔══██║
██║ ╚████║╚██████╔╝███████╗██║ ╚═╝ ██║██║  ██║
╚═╝  ╚═══╝ ╚═════╝ ╚══════╝╚═╝     ╚═╝╚═╝  ╚═╝
```

**Thought, made legible.**

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)
[![Status](https://img.shields.io/badge/status-research%20plan-8a2be2.svg)](#project-status)

</div>

Noema is an open-source biosignal-to-text engine. It combines signals such as EEG, EMG, EOG, and motion data to turn deliberate human intent into text and commands—privately, transparently, and with hardware people can actually own.

The name comes from philosophy: a *noema* is the content or object of thought.

Noema begins with practical communication: trained commands, character selection, silent articulation, and user-specific decoding. It grows toward richer attempted- and imagined-speech research without pretending that today's consumer sensors can read arbitrary private thoughts.

> [!IMPORTANT]
> Noema is not a mind reader, medical device, diagnostic system, or safety-critical communication aid. It is an early open research project. Reliable decoding requires deliberate participation, calibration, controlled evaluation, and an explicit way to abstain when the signal is uncertain.

## Why Noema

Speech and typing are only two ways to express intent. For people who cannot reliably use them—and for anyone exploring new forms of human-computer interaction—the body still produces rich electrical signals before, during, and around an intended action.

Noema treats those signals as a new input modality:

- **Communication** — spell text or express a small vocabulary without relying on a keyboard or audible speech.
- **Accessible control** — map deliberate, trained patterns to commands while preserving a reliable pause and exit path.
- **Multimodal decoding** — combine weak evidence across brain, muscle, eye, and motion sensors instead of demanding that one device do everything.
- **A reusable SDK** — let applications consume text hypotheses and intent events without owning the acquisition and decoding pipeline.
- **Open research** — make calibration, datasets, metrics, failure cases, and hardware support inspectable and reproducible.

## What “thought to text” means here

Noema uses the phrase as a direction, not as permission to overclaim. The project separates several increasingly difficult interaction modes:

1. **Intent commands** — trained states such as yes/no, select/cancel, left/right, or rest/active.
2. **Stimulus-assisted spelling** — deliberate selection through paradigms such as event-related potentials or steady-state visual responses.
3. **Silent articulation** — text or phoneme inference aided by subtle facial, jaw, throat, or forearm EMG.
4. **Attempted speech** — decoding speech a participant intentionally tries to produce.
5. **Imagined speech** — experimental inference from internally rehearsed language with no overt movement.
6. **Open-ended thought** — outside the project's present claims; Noema does not promise unrestricted access to a person's inner monologue.

Each output carries provenance, confidence, timing, and decoder state. Applications can distinguish an EEG-assisted selection from an EMG-assisted phoneme or a language-model-ranked hypothesis.

## Signals, together

Noema is hardware-flexible and modality-aware.

| Signal | Role in Noema |
| --- | --- |
| EEG | Event-related responses, oscillatory patterns, motor imagery, and experimental speech representations |
| EMG | Deliberate muscle activity, silent articulation, gesture confirmation, and artifact context |
| EOG | Eye movement and blink intent, plus ocular-artifact awareness for EEG |
| IMU | Motion context, sensor displacement, and rejection of movement-corrupted windows |
| Markers | Precisely timed prompts, targets, actions, and ground-truth labels during calibration |

Signals are not interchangeable. Noema keeps their channel maps, sampling rates, timestamps, preprocessing, and confidence separate until an explicit fusion stage.

## Interaction model

The safest output is not always a word. Noema can emit a ranked hypothesis, request confirmation, or abstain.

```text
listening → evidence accumulating → candidate → confirmed
    │                 │                │
    └──── uncertain ──┴──── timeout ───┴──► abstain
```

A deliberate activation gate arms decoding. A pause control remains available through another modality. Destructive or security-sensitive commands require confirmation outside the decoder itself.

Language models can rank plausible candidates, but they must not turn weak sensor evidence into false certainty. Raw decoder confidence, uncorrected output, and the reason for abstention remain available to the user and consuming application.

## Architecture

```text
EEG devices     EMG sensors     EOG / eye channels     IMUs     task markers
     └──────────────┴──────────────────┴─────────────────┴──────────┘
                                  │
                    device adapters + clock sync
                                  │
                 typed, timestamped multimodal stream
                                  │
             channel validation + signal-quality monitor
                                  │
        causal preprocessing + artifact annotation / rejection
                                  │
        ┌─────────────────────────┴────────────────────────┐
        │                                                  │
 classical baselines                              learned encoders
 ERP / SSVEP / spectral /                   temporal, spatial, and
 Riemannian features                         subject-adapted models
        └─────────────────────────┬────────────────────────┘
                                  │
                   confidence-aware sensor fusion
                                  │
                constrained sequence / text decoder
                                  │
            hypotheses + commands + quality + provenance
                       ┌──────────┴──────────┐
                       ▼                     ▼
                 reference UI          Noema SDK / apps
```

The boundaries matter. Acquisition records what the hardware produced. Preprocessing records every transformation. Models estimate a trained target. The decoder proposes an output. The application decides what that output is allowed to do.

An initial platform-neutral event vocabulary includes:

```text
SignalWindow(modality, channels, samples, timestamps)
SignalQuality(channel, status, metrics)
Marker(kind, value, timestamp)
TextHypothesis(text, confidence, provenance, latency)
Intent(name, confidence, provenance, timestamp)
DecoderState(calibrating | listening | uncertain | paused | lost)
```

These shapes define the intended boundary, not a committed API yet.

## Hardware philosophy

Noema does not belong to one headset. Device-specific details live behind adapters, while downstream pipelines consume a common timestamped stream.

The first hardware work focuses on the sensors available to contributors, including Athena hardware, EEG boards, and standalone muscle sensors. Every adapter documents:

- signal modalities, channel names, units, sampling rates, and reference scheme
- connection and synchronization behavior
- packet loss, buffering, latency, and clock drift
- vendor SDK and firmware requirements
- whether raw data is actually available
- protocol, dependency, and redistribution licenses

A device appears in the supported-hardware matrix only after those properties are tested. Marketing claims are not treated as specifications.

## Technical direction

The research layer uses Python because the biosignal ecosystem is strongest there. The first experiments center on:

- **BrainFlow and device SDKs** for hardware acquisition where supported
- **Lab Streaming Layer** for synchronized multimodal streams and experiment markers
- **MNE-Python, NumPy, and SciPy** for signal inspection and preprocessing
- **pyRiemann and scikit-learn** for strong, interpretable classical baselines
- **PyTorch and Braindecode** for learned temporal and spatial encoders
- **XDF and a documented Noema session schema** for reproducible recordings
- **A typed event API** that keeps apps independent of model and hardware choices

Real-time code uses causal transforms and streaming evaluation. Offline notebooks never silently become production pipelines. Any faster native core or mobile binding preserves the same event and provenance model.

## Dataset contract

A model is only as trustworthy as its recording protocol. Each Noema session pairs signal data with:

- pseudonymous participant and session identifiers
- device, firmware, montage, reference, and sampling metadata
- electrode or sensor placement and contact-quality observations
- synchronized task prompts, responses, and timing markers
- calibration state and preprocessing history
- consent scope, retention policy, and permitted uses
- environmental notes and known failure conditions

Public releases are separate, explicit decisions. Recording data for local experiments never implies consent to upload it, train a shared model, or publish it.

## Evaluation

Noema evaluates the full interaction, not only a model's best offline score:

- character error rate and word error rate
- command accuracy, false activations, and information-transfer rate
- confidence calibration and abstention quality
- time to first useful output and end-to-end latency
- setup and calibration time
- performance across sessions, days, devices, and sensor placement changes
- robustness to eye, jaw, head, and cable movement
- performance before and after language-model assistance
- per-participant results rather than only pooled averages

Train/test splits are grouped by session or participant where appropriate. Streaming benchmarks preserve temporal order and prevent preprocessing leakage from future samples.

## Privacy, consent, and agency

Neural and muscular recordings are intimate biometric data. Noema adopts strict defaults:

- process locally and work offline for core acquisition and decoding
- record only after an unmistakable opt-in action
- show when sensors are live, recording, calibrating, or transmitting
- encrypt saved sessions and keep raw data out of logs
- make export, retention, and deletion understandable and user-controlled
- never use signals for identity, emotion, truthfulness, health, or hidden mental-state inference
- never upload recordings or train shared models without separate informed consent
- preserve a physical or independent way to pause and disconnect

Noema is built to express what a person deliberately chooses to communicate—not to infer what they did not consent to share.

## Roadmap

### 0. Foundation

- define the session schema, event vocabulary, consent model, and benchmark protocol
- inventory available hardware and verify access to raw streams
- build a replayable synthetic signal source before depending on physical devices

### 1. Record and inspect

- acquire synchronized EEG, EMG, EOG, IMU, and task markers
- visualize channels, contact quality, latency, packet loss, and artifacts
- save and deterministically replay complete sessions

### 2. Deliberate intent

- establish rest-versus-active and small command-vocabulary baselines
- compare classical features with compact learned models
- implement confidence calibration, activation gates, and abstention

### 3. Text selection

- prototype stimulus-assisted character selection
- measure typing rate, error correction cost, fatigue, and false activation
- add an accessible reference speller with multimodal confirmation

### 4. Multimodal speech

- align EEG with facial, jaw, throat, or forearm EMG where available
- investigate phoneme, syllable, and constrained-vocabulary decoding
- quantify what each modality contributes through ablation studies

### 5. Sequence decoding

- explore attempted- and imagined-speech protocols with subject-specific training
- separate sensor evidence from optional language-model reranking
- publish negative results and generalization limits alongside successful experiments

### 6. SDK and applications

- stabilize text, intent, quality, and provenance events
- provide a reference desktop application and documented integrations
- connect Noema with other Wega input modalities such as gaze, gesture, and voice

## Project status

Noema is at the research-plan stage. This repository contains the product thesis, technical boundaries, ethical commitments, and roadmap. It does not contain a working decoder, and no code from the projects below has been copied.

## References and upstream ecosystem

Noema starts fresh and learns from established open-source tools and research:

- [BrainFlow](https://github.com/brainflow-dev/brainflow) — a hardware-agnostic acquisition and signal-processing API for EEG, EMG, ECG, and other biosensors.
- [Lab Streaming Layer](https://github.com/sccn/labstreaminglayer) — synchronized collection of multimodal time series and experiment markers.
- [MNE-Python](https://github.com/mne-tools/mne-python) — analysis and visualization for EEG, MEG, and other neurophysiological data.
- [Braindecode](https://github.com/braindecode/braindecode) — deep-learning models and tooling for electrophysiological signals.
- [pyRiemann](https://github.com/pyRiemann/pyRiemann) — classical learning methods for multichannel biosignals and BCI paradigms.
- [MOABB](https://github.com/NeuroTechX/moabb) — reproducible benchmarking across public BCI datasets.
- [EEG-ExPy](https://github.com/NeuroTechX/EEG-ExPy) — accessible EEG experiment examples from NeuroTechX.
- [Brain2Qwerty](https://arxiv.org/abs/2502.17480) — non-invasive brain-to-text research recorded while participants typed memorized sentences; useful context, not evidence of unrestricted thought reading.

Before adding any dependency, code, model, dataset, or asset, contributors must verify its license and use restrictions, preserve attribution, and document provenance. Improvements that belong in an upstream project are contributed back whenever practical.

## Contributing

Noema welcomes signal-processing researchers, accessibility experts, neuroscientists, hardware hackers, ML engineers, product designers, and people with lived experience of communication access needs.

Early contributions can focus on hardware protocol research, session schemas, experiment design, privacy review, synthetic streams, evaluation harnesses, and literature notes. Hardware recordings and participant studies require an explicit consent and data-handling plan before collection begins.

## License

Noema is licensed under the [Apache License 2.0](LICENSE).
