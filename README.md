# Stable Music Video

A live diffusion based music video system that generates real-time visuals in sync with whatever is currently playing on Spotify. It reads the current track: title, artist, lyrics, album art, and audio features ect, and uses this to continuously prompt a Stable Diffusion model, producing an ever-evolving visual stream that responds to the music.

## How It Works

1. **Spotify** provides the current track's metadata, lyrics, and audio analysis (tempo, energy, valence, etc.)
2. A **prompt engine** translates this information into Stable Diffusion prompts and generation parameters, blending in characterised noise derived from audio features
3. **StreamDiffusion** runs real-time img2img inference, taking a noise input and producing AI-generated frames at up to 60fps
4. **TouchDesigner** receives these frames and handles visual composition and output

```
Spotify API
    │
    ├── Track title, artist, album art
    ├── Lyrics
    └── Audio features (BPM, energy, valence, danceability...)
            │
            ▼
    Prompt Engine
    (builds SD prompt + maps audio features → generation params)
            │
            ▼
    StreamDiffusion (Python)  ◄──── Characterised noise input
    (real-time img2img, sd-turbo)
            │
     Shared memory / Syphon
            │
            ▼
    TouchDesigner
    (visual composition & output)
```

## Components

### Spotify Integration
Connects to the Spotify Web API to poll the currently playing track. Extracts:
- **Track metadata** — title, artist, album, album art
- **Lyrics** — sourced and aligned to the current playback position
- **Audio features** — BPM, energy, valence, danceability, acousticness, and more from Spotify's audio analysis endpoint

These are used to construct a dynamic prompt and to modulate generation parameters in real time.

### Prompt Engine
Translates Spotify data into StreamDiffusion inputs:
- Builds a text prompt from track title, artist, genre, and lyrics
- Maps audio features to generation parameters (e.g. energy → `guidance_scale`, BPM → noise intensity, valence → colour temperature)
- Generates characterised noise as the img2img input, shaped by the audio analysis

### StreamDiffusion
Handles real-time Stable Diffusion inference. Runs `sd-turbo` in img2img mode with TensorRT acceleration (Windows/Linux) or MPS (macOS), producing frames at up to 60fps.

Parameters are updated live via OSC — prompts, seeds, guidance scale, delta, t-index list, LoRA weights, and more can all be changed without restarting the pipeline.

- **Config:** `StreamDiffusion/StreamDiffusionTD/stream_config.json`
- **Main process:** `StreamDiffusion/StreamDiffusionTD/main_sdtd.py`
- **Model wrapper:** `StreamDiffusion/StreamDiffusionTD/wrapper_td.py`

### TouchDesigner
Receives the generated frames from StreamDiffusion via shared memory (Windows) or Syphon (macOS) and handles final visual composition, display, and any further post-processing.

The TouchDesigner component file is `StreamDiffusionTD-0.2.3.tox`

## Credits

- **[StreamDiffusion](https://github.com/cumulo-autumn/streamdiffusion)** by cumulo-autumn et al. -> the real-time Stable Diffusion pipeline that makes high-framerate inference possible
- **[StreamDiffusionTD](https://dotsimulate.com/tools)** by dotsimulate -> the TouchDesigner integration layer, OSC control interface, shared memory frame transport, and V2V attention caching on top of StreamDiffusion