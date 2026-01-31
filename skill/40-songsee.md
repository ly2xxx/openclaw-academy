# Songsee - Audio Visualization

**Location:** ` \openclaw\skills\songsee\`  
**Emoji:** 🌊  
**Requirements:** `songsee` CLI  
**Homepage:** https://github.com/steipete/songsee

## Purpose
Generate spectrograms and feature-panel visualizations from audio files.

## Installation

```bash
brew install steipete/tap/songsee
```

## Quick Start

**Basic spectrogram:**
```bash
songsee track.mp3
```

**Multi-panel visualization:**
```bash
songsee track.mp3 --viz spectrogram,mel,chroma,hpss,selfsim,loudness,tempogram,mfcc,flux
```

**Time slice:**
```bash
songsee track.mp3 --start 12.5 --duration 8 -o slice.jpg
```

**From stdin:**
```bash
cat track.mp3 | songsee - --format png -o out.png
```

## Visualization Types

| Type | Description |
|------|-------------|
| `spectrogram` | Frequency spectrum over time |
| `mel` | Mel-scale spectrogram |
| `chroma` | Chromagram (pitch classes) |
| `hpss` | Harmonic-Percussive Source Separation |
| `selfsim` | Self-similarity matrix |
| `loudness` | Loudness over time |
| `tempogram` | Tempo estimation |
| `mfcc` | Mel-Frequency Cepstral Coefficients |
| `flux` | Spectral flux |

## Common Flags

### Visualization

```bash
--viz <type>           # Repeatable or comma-separated
--style <palette>      # classic, magma, inferno, viridis, gray
```

### Output

```bash
--width <pixels>       # Output width
--height <pixels>      # Output height
--format jpg|png       # Output format
-o <path>              # Output file
```

### Analysis

```bash
--window <size>        # FFT window size
--hop <size>           # Hop length
--min-freq <hz>        # Minimum frequency
--max-freq <hz>        # Maximum frequency
```

### Time Slice

```bash
--start <seconds>      # Start time
--duration <seconds>   # Duration to analyze
```

## Examples

**High-quality spectrogram:**
```bash
songsee track.mp3 --style magma --width 2400 --height 800 --format png -o spec.png
```

**Multi-panel grid:**
```bash
songsee track.mp3 --viz spectrogram,mel,chroma,loudness --style viridis -o analysis.jpg
```

**Specific time range:**
```bash
songsee track.mp3 --start 30 --duration 15 --viz mel,chroma -o chorus.png
```

**Frequency-focused:**
```bash
songsee track.mp3 --min-freq 80 --max-freq 8000 --style inferno -o mid-range.png
```

## Supported Formats

**Native decode:**
- WAV
- MP3

**Via ffmpeg** (if available):
- FLAC, OGG, M4A, etc.

## Best Practices

1. **Multiple visualizations** for comprehensive analysis
2. **Time slicing** for specific sections
3. **PNG format** for quality (JPG for size)
4. **Style palettes** for clarity (magma/viridis often best)
5. **Frequency range** to focus on specific bands

## Use Cases

**Music analysis:**
```bash
songsee song.mp3 --viz spectrogram,chroma,tempogram --style magma
```

**Vocal isolation check:**
```bash
songsee vocal.wav --viz hpss --style viridis
```

**Rhythm analysis:**
```bash
songsee beat.mp3 --viz tempogram,flux,loudness
```

**Sound comparison:**
```bash
songsee before.wav --start 10 --duration 5 -o before.png
songsee after.wav --start 10 --duration 5 -o after.png
```

## Source
SKILL.md: ` \openclaw\skills\songsee\SKILL.md`
