# Interest Profile

Interests are tiered. When ranking, weight Primary Focus papers highest, Core
Interests moderately, and Secondary Interests lowest — a Secondary-area paper
should only make the digest if it is a standout.

## Primary Focus (highest priority)
- **ASR / Speech Recognition**: End-to-end models (conformer, transducer, CTC, attention-based), streaming/online ASR, robustness, long-form ASR, Whisper-based work, ASR error correction
- **Multilingual & Low-Resource ASR**: Massively multilingual models, cross-lingual transfer, code-switching, under-resourced languages, language ID
- **Speech LLMs for ASR & Understanding**: LLM-based ASR (audio-conditioned LLMs, LLM decoders/rescoring), speech-in LLMs, instruction-following speech models, contextual biasing and prompting for recognition
- **Speech–Text Modality Alignment**: How to align the speech modality with text so the backbone LLM loses the least knowledge — knowledge retention / catastrophic forgetting in modality adaptation, representation alignment between speech and text, entity-aware recognition (person names, product names, rare words), named entity recognition from speech, spoken language understanding, leveraging LLM world knowledge in speech tasks
- **Speech Translation**: End-to-end and cascaded speech-to-text translation, simultaneous/streaming translation, multilingual ST

## Core Interests (still tracked)
- **Speech Language Models (broad)**: Codec LMs, GPT-style audio models, speech-to-speech models, full-duplex/conversational models
- **Audio Codec / Tokenization**: Neural audio codecs, discrete speech representations, speech tokenizers (especially as they affect recognition/understanding quality)
- **Self-Supervised Speech**: wav2vec, HuBERT, data2vec and successors; representation learning for recognition
- **Evaluation & Benchmarks**: ASR/Speech-LLM benchmarks, entity-heavy or knowledge-heavy test sets, multilingual benchmarks

## Secondary Interests (lower priority — surface only if standout)
- **Speech Synthesis / TTS**: Neural TTS, zero-shot voice cloning, prosody, expressiveness
- **Voice Conversion**: Zero-shot VC, accent conversion
- **Speech Enhancement**: Noise suppression, dereverberation, source separation
- **Speaker Diarization / Verification**: Who-spoke-when, anti-spoofing, speaker embeddings
- **Music Generation / General Audio Generation**
- **Audio Captioning / Understanding**: Sound event detection, audio QA, audio-visual learning
- **Diffusion / Flow Matching for Audio** (generation-side; alignment/recognition uses remain Primary-adjacent)

## Ranking Signals (High Priority)
- Directly addresses speech–text alignment, knowledge retention in Speech LLMs, or entity/rare-word recognition
- Novel architecture or training paradigm for ASR or Speech LLMs
- Strong multilingual or low-resource results
- State-of-the-art benchmark results with credible evaluation
- Real-time or streaming capability
- Zero-shot/few-shot generalization
- From a top lab or well-known author

## Ranking Signals (Deprioritize)
- Pure TTS/voice-cloning or music-generation papers without a standout contribution
- Pure survey papers (unless very comprehensive)
- Narrow domain-specific applications unrelated to core speech/audio
- Papers dominated by radar, medical ultrasound, seismic, sonar signal processing
