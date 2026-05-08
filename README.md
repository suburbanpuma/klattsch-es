# klattsch
## THIS IS VERY UNFINISHED. IT WONT BUILD.
A fork of the one and only primitive parallel-formant speech synthesizer in the browser, klattsch. Late-70s / early-80s tier (Votrax, SAM), now with Spanish!

The name is a portmanteau of *Klatt* (Dennis Klatt, the formant-synth pioneer) and *Klatsch* (German for gossip / casual chat).

[**Live demo**](https://tgies.github.io/klattsch/)

[**Commercial support**](#commercial-support) - integration consulting from the author

## What it does

You type a phoneme string in [ARPABET](https://en.wikipedia.org/wiki/ARPABET), Romaji, or with the [YYA Spanish Phonetic Alphabet](https://github.com/YYAlliance/spanish-phoneset/) with optional directives, and the computer says it.

```
HH AH L OW                        hello, default voice
b140 HH AH L OW                   higher voice
bA3 HH AH L OW                    higher voice (note name)
AY+15 D IH D                      "I did" with a rising contour
D IH D DH AE(+40) T               "did THAT" with a transient pitch ornament on AE
r200 bC#4 ( HH AH ) ( L OW )      sung syllables, one note per group
```

See the in-app `syntax help` panel for the full directive table.

## Usage

### CLI

Render a phoneme string straight to a WAV file:

```bash
npx klattsch "HH AH L OW" hello.wav
```

### Node / `OfflineAudioContext`

```js
import { compileString, FormantSynth, encodeWav } from 'klattsch';

const sampleRate = 48000;
const { schedule, totalMs } = compileString('HH AH L OW');
const synth = new FormantSynth({ sampleRate, schedule });
const buf = new Float32Array(Math.ceil(totalMs * sampleRate / 1000));
synth.process(buf);

const { bytes } = encodeWav(buf, sampleRate);
// write bytes to a .wav file
```

### Browser with a bundler (Vite, webpack, esbuild, Rollup)

```js
import { compileString } from 'klattsch';
import workletUrl from 'klattsch/formant-worklet.js?url';

const ctx = new AudioContext();
await ctx.audioWorklet.addModule(workletUrl);
const node = new AudioWorkletNode(ctx, 'formant-processor');
node.connect(ctx.destination);

const { schedule } = compileString('HH AH L OW');
node.port.postMessage({ type: 'schedule', schedule });
```

### Browser without a bundler (CDN)

```html
<script type="module">
  import { compileString } from 'https://esm.sh/klattsch';

  const ctx = new AudioContext();
  await ctx.audioWorklet.addModule('https://esm.sh/klattsch/formant-worklet.js');
  const node = new AudioWorkletNode(ctx, 'formant-processor');
  node.connect(ctx.destination);

  const { schedule } = compileString('HH AH L OW');
  node.port.postMessage({ type: 'schedule', schedule });
</script>
```

## How it works

- **Excitation:** voiced source is a Rosenberg-style glottal pulse with a tunable open / closed quotient (`g` / "effort") and unvoiced source is xorshift noise. These are crossfaded by each phoneme's `voicing` parameter, with optional aspiration noise mixed in.
- **Filtering:** three parallel bandpass biquads for each formant.
- **Prosody:** the sequencer compiles phoneme strings into a time-stamped schedule of formant targets.
- **Voice character:** vibrato (depth + rate), aspiration / breathiness, spectral tilt, and glottal effort are all controllable.

## References

- Klatt, D. H. (1980). *Software for a cascade/parallel formant synthesizer.*
- Hillenbrand et al. (1995). *Acoustic characteristics of American English vowels.*
- Rosenberg, A. E. (1971). *Effect of glottal pulse shape on the quality of natural vowels.*
- Robinson, R. Bristow-Johnson. *Audio EQ Cookbook.*
- Mokhtari, P. & Tanaka, K. (2000). *A Corpus of Japanese Vowel Formant Patterns.* Bulletin of the Electrotechnical Laboratory (ETL), Vol. 64, Special Issue, 57-66. ([project page](https://isd.pu-toyama.ac.jp/~parham/sp_FormantDataETL.html), [data file](https://web.archive.org/web/20240811224814/https://isd.pu-toyama.ac.jp/~parham/documents/formantsETL/MokhtariTanaka2000_ETLformantdata.txt)) - source of the Japanese vowel formants in the `ja-mokhtari-2000` phoneme bank.

## See also

- [**libadlmidi-js**](https://github.com/libadlmidi-js/libadlmidi-js) - WebAssembly build of libADLMIDI, an OPL3 FM synthesis library with AudioWorklet integration. Where klattsch does parallel-formant *vocal-tract* synthesis, libadlmidi-js does FM-operator synthesis: the sound of early-80s arcade boards and AdLib cards. Includes [oplsfxr](https://libadlmidi-js.github.io/examples/oplsfxr.html), a sfxr-style sound effect generator.

## Built with klattsch

- [**klattsch-sing**](https://sing.wasthatzero.net/) - a piano-roll sequencer for speech-based singing synthesis. Draw notes, type words or phonemes, render to WAV. Supports MIDI import, quantization, tempo + time signature, demo songs, and a custom KSP project file format. Built on the klattsch npm package.

## Commercial Support

`klattsch` is built and maintained by [Tony Gies](https://github.com/tgies). For studios, indie developers, and agencies integrating klattsch into a shipped product, consulting is available through his consultancy, Crash United, LLC.

### Sponsorship

To support ongoing development without a formal contract, [GitHub Sponsors](https://github.com/sponsors/tgies) or [Ko-fi](https://ko-fi.com/crashunited) are the simplest paths. SUPPORT TGIES, THIS PROJECT IS SUPER COOL!

## License

MIT &copy; Tony Gies
