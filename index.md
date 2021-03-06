---
layout: default
---

Demo for the final project of _[GCT535 Spring 2022 Sound Technology for Multimedia, KAIST](https://mac.kaist.ac.kr/~juhan/gct535/index.html)_.

We constructed our own **Vocoder Effect and related parameters to mimick several iconic vocoder-effected vocal tracks** which are Daftpunk, Zedd and Stevie Wonder.

- First, we implemented a classic channel vocoder using band-pass filters and RMS filters.

- Second, we implemented and compared controllable parameters including F0, Number of Frequency Bands, Frequency Scale, Random Noising, Formant Shifting and Beta between modulator and carrier.
  We also implemented compressor and expander to improve the effector sound.

- Finally, we mimicked the target artists using our implemented vocoder and its parameters.
  We also generated interesting results with various types of carrier sounds.

You can check [implementation detail](#vocoder-implementation) and [result samples](#result-samples) below.

## Vocoder Implementation

<script>
MathJax = {
  tex: {
    inlineMath: [['$', '$'], ['\\(', '\\)']]
  },
  svg: {
    fontCache: 'global'
  }
};
</script>
<script type="text/javascript" id="MathJax-script" async
  src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-svg.js">
</script>

<br>

<p>
    <img src="public/images/vocoder.png" alt>
    <em>Figure.1 Implementation of Vocoder</em>
</p>

<br>

A vocoder takes two different signals, carrier $car$ and modulator $mod$, as inputs and outputs one result signal $output$. Let the number of frequency bands $N$, the number of time samples $T$.

Each bandpass filter $bandpass_i, i\in [1, N]$ corresponding to a single frequency band has passband $[f_{i-1}, f_i]$.
The output signal is retrieved as the following:

$$output_t=ISTFT(\Sigma_{i\in [1, N]}{RMS(bandpass_i(mod_t))\times bandpass_i(STFT(car_t))}), t\in[0, T]$$.

RMS and STFT are calculated in the same hop length.

As a result, we can retrieve a modified carrier signal that has the same envelope as the modulator signal in the frequency domain over time which reflects the formant characteristics of the modulator.

<br>

<p>
    <img src="public/images/compare_stft.png" alt>
    <em>Figure.2 Comparison between original STFT and vocoded STFT</em>
</p>

<br>

## Result Samples

<table>
    <tr>
        <th>Modulator</th>
        <th>Carrier</th>
        <th>Vocoded Result</th>
    </tr>
    <tr>
        <td rowspan="5"><audio src="public/audios/sources/wow.wav" controls></audio></td>
        <td><audio src="public/audios/carriers/sawtooth.wav" controls></audio></td>
        <td><audio src="public/audios/results/wow_basic.wav" controls></audio></td>
    </tr>
    <tr>
        <td><audio src="public/audios/carriers/exp_sawtooth.wav" controls></audio></td>
        <td><audio src="public/audios/results/wow_exp_sawtooth.wav" controls></audio></td>
    </tr>
    <tr>
        <td><audio src="public/audios/carriers/orchestra.wav" controls></audio></td>
        <td><audio src="public/audios/results/wow_orchestra.wav" controls></audio></td>
    </tr>
    <tr>
        <td><audio src="public/audios/carriers/pad_chord.wav" controls></audio></td>
        <td><audio src="public/audios/results/wow_pad_chord.wav" controls></audio></td>
    </tr>
    <tr>
        <td><audio src="public/audios/carriers/wind.wav" controls></audio></td>
        <td><audio src="public/audios/results/wow_wind.wav" controls></audio></td>
    </tr>
</table>

<br>

---

### Controllable Parameters

{% tabs parameters %}

{% tab parameters F0 %}

The **fundamental frequency**, often referred to simply as F0, is the lowest frequency of a periodic signal. In music, the pitch is the fundamental frequency of a note.
Since the carrier sound is solely in charge of the harmonic, the result's pitch is controlled by the F0 value of the carrier signal.

|                                 f0=220                                  |                                 f0=440                                  |                                 f0=880                                  |
| :---------------------------------------------------------------------: | :---------------------------------------------------------------------: | :---------------------------------------------------------------------: |
| <audio src="public/audios/results/f0/suzanne_220.wav" controls></audio> | <audio src="public/audios/results/f0/suzanne_440.wav" controls></audio> | <audio src="public/audios/results/f0/suzanne_880.wav" controls></audio> |

{% endtab %}

{% tab parameters f Bands %}

**F Bands** refers to the number of frequency bands into which the modulator signal is split.
The larger number of F Bands means the smaller range of band-pass filters, vice versa.
You can think of this number as a resolution of the modulator's formant.
Check the samples focusing on the formant(vowels) resolution.

|                                  band_num=10                                  |                                  band_num=60                                  |                                  band_num=120                                  |
| :---------------------------------------------------------------------------: | :---------------------------------------------------------------------------: | :----------------------------------------------------------------------------: |
| <audio src="public/audios/results/freq_band/suzanne_10.wav" controls></audio> | <audio src="public/audios/results/freq_band/suzanne_60.wav" controls></audio> | <audio src="public/audios/results/freq_band/suzanne_120.wav" controls></audio> |

{% endtab %}

{% tab parameters Scale %}

Different **Scales**, such as linear or mel(log), can be applied on frequency axis when dividing equal-width Frequency Bands.
It determines the range of each frequency band, which is important to human pitch perception and formant shift.

|                                    Linear Scale                                    |                                    Mel Scale                                    |
| :--------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------: |
| <audio src="public/audios/results/freq_scale/suzanne_linear.wav" controls></audio> | <audio src="public/audios/results/freq_scale/suzanne_mel.wav" controls></audio> |

{% endtab %}

{% tab parameters Noise %}

To highlight sibilant sounds (e.g. _s, sh, t, ch, chh_, etc.), high-frequency **Noise** can be added to the carrier signal.
As these sounds do not have a particular pitch, adding frequency components around 8kHz to 16kHz may help them to stand out.  
For implementation, we generated a random white noise and applied a bi-quad high-pass filter.
_amp_ is the amplitude of the noise. _Q_ is the Q value of the bi-quad filter.

|                                without Noise                                 |                                   amp=0.5, Q=1                                   |                                   amp=0.7, Q=3                                   |
| :--------------------------------------------------------------------------: | :------------------------------------------------------------------------------: | :------------------------------------------------------------------------------: |
| <audio src="public/audios/results/noise/suzanne_basic.wav" controls></audio> | <audio src="public/audios/results/noise/suzanne_amp0.5_Q1.wav" controls></audio> | <audio src="public/audios/results/noise/suzanne_amp0.7_Q3.wav" controls></audio> |

{% endtab %}

{% tab parameters Formant %}

**Formant-shifting** is achieved by shifting the multiplication between the frequency band channels from modulator and carrier.
If the shift step is 1, the modulator channel n would be mapped to the carrier channel n+1.
Note that formant does not relate to the pitch or frequency of the signal, but rather the timbre and spectrum peaks.
Each formant corresponds to a resonance in the vocal tract and is prominent in vowels.

|                                     shift=0                                      |                                     shift=-1                                      |                                     shift=2                                      |
| :------------------------------------------------------------------------------: | :-------------------------------------------------------------------------------: | :------------------------------------------------------------------------------: |
| <audio src="public/audios/results/formant_shift/suzanne_0.wav" controls></audio> | <audio src="public/audios/results/formant_shift/suzanne_-1.wav" controls></audio> | <audio src="public/audios/results/formant_shift/suzanne_2.wav" controls></audio> |

{% endtab %}

{% tab parameters Compressor %}

**Compressor** attenuates the given sound above threshold (dB) with a certain ratio (>1) while maintaining the maximum amplitude level (dB).
It reduces the dynamic range without clipping.  
Applying the effector before vocoder softens the dynamic change in formant.

|                                without Compressor                                |                                 with Compressor                                 |
| :------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------: |
| <audio src="public/audios/results/comp&expd/suzanne_basic.wav" controls></audio> | <audio src="public/audios/results/comp&expd/suzanne_comp.wav" controls></audio> |

{% endtab %}

{% tab parameters Expander %}

**Expander**, in contrast to compressor, attenuates the given sound below threshold (dB) with a certain ratio (>1).
It can reduce unnecessary noise.

|                                 without Expander                                 |                                  with Expander                                  |
| :------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------: |
| <audio src="public/audios/results/comp&expd/suzanne_basic.wav" controls></audio> | <audio src="public/audios/results/comp&expd/suzanne_expd.wav" controls></audio> |

{% endtab %}

{% tab parameters Beta %}

**Beta** refers to the non-linear alternation of modulator signal as follows: ${modulator}^{beta} * carrier$
This parameter was devised to control the artifact of modulator signal.

|                                 ratio=0.3                                 |                                 ratio=0.7                                 |                                 ratio=1.2                                 |
| :-----------------------------------------------------------------------: | :-----------------------------------------------------------------------: | :-----------------------------------------------------------------------: |
| <audio src="public/audios/results/beta/suzanne_0.3.wav" controls></audio> | <audio src="public/audios/results/beta/suzanne_0.7.wav" controls></audio> | <audio src="public/audios/results/beta/suzanne_1.2.wav" controls></audio> |

{% endtab %}

{% endtabs %}

<br>

---

### Various Carrier Samples

We applied various types of carrier sounds to the two modulator, _suzanne_ and _dog sound_.
Check the below samples focusing on how the sound is modulated following the carrier's texture or harmonic structure.

#### 1. Suzanne

<audio src="public/audios/sources/suzanne.wav" controls></audio>

<table>
    <tr>
        <th></th>
        <th>Carrier</th>
        <th>Vocoded Result</th>
    </tr>
    <tr>
        <td>Incremental Sawtooth</td>
        <td><audio src="public/audios/carriers/exp_sawtooth.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/suzanne_exp_sawtooth.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Orchestra</td>
        <td><audio src="public/audios/carriers/orchestra.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/suzanne_orchestra.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Pad Chord</td>
        <td><audio src="public/audios/carriers/pad_chord.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/suzanne_pad_chord.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Wind</td>
        <td><audio src="public/audios/carriers/wind.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/suzanne_wind.wav" controls></audio></td>
    </tr>
</table>

#### 2. Dog Sound

<audio src="public/audios/sources/dog_sound.wav" controls></audio>

<table>
    <tr>
        <th></th>
        <th>Carrier</th>
        <th>Vocoded Result</th>
    </tr>
    <tr>
        <td>Incremental Sawtooth</td>
        <td><audio src="public/audios/carriers/exp_sawtooth.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_exp_sawtooth.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Harp</td>
        <td><audio src="public/audios/carriers/harp.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_harp.wav" controls></audio></td>
    </tr>
    <tr>
        <td>School Bell</td>
        <td><audio src="public/audios/carriers/school_bell.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_school_bell.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Synth</td>
        <td><audio src="public/audios/carriers/synth.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_synth.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Whale</td>
        <td><audio src="public/audios/carriers/whale.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_whale.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Meow</td>
        <td><audio src="public/audios/carriers/meow.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_meow.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Starwars</td>
        <td><audio src="public/audios/carriers/starwars.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_starwars.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Trumpet</td>
        <td><audio src="public/audios/carriers/trumpet.wav" controls></audio></td>
        <td><audio src="public/audios/results/carriers/dog_sound_trumpet.wav" controls></audio></td>
    </tr>
</table>

<br>

## Let's Mimick Some Artists!

We used sawtooth signals with static or dynamic fundamental frequency over time as a carrier sound.
By manipulating the controllable parameters of the vocoder, we finally got the resulting sound tracks.

#### 1. Daftpunk - Harder, Better, Faster, Stronger

<table>
    <tr>
        <td rowspan="3"><img src="public/images/daftpunk.jpeg" width="250px"/></td>
        <td>Target</td>
        <td><audio src="public/audios/results/artist/daftpunk.mp3" controls></audio></td>
    </tr>
    <tr>
        <td>Our Voice</td>
        <td><audio src="public/audios/results/artist/daftpunk_vocal.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Mimicked Result</td>
        <td><audio src="public/audios/results/artist/daftpunk_result+mr.wav" controls></audio></td>
    </tr>
</table>

#### 2. Zedd - Stay

<table>
    <tr>
        <td rowspan="3"><img src="public/images/zedd.jpg" width="250px"/></td>
        <td>Target</td>
        <td><audio src="public/audios/results/artist/zedd.mp3" controls></audio></td>
    </tr>
    <tr>
        <td>Our Voice</td>
        <td><audio src="public/audios/results/artist/zedd_vocal.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Mimicked Result</td>
        <td><audio src="public/audios/results/artist/zedd_result+mr.wav" controls></audio></td>
    </tr>
</table>

#### 3. Stevie Wonder - Close to you

<table>
    <tr>
        <td rowspan="3"><img src="public/images/closetoyou.jpeg" width="250px"/></td>
        <td>Target</td>
        <td><audio src="public/audios/results/artist/Closetoyou.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Our Voice</td>
        <td><audio src="public/audios/results/artist/Closetoyou_vocal.wav" controls></audio></td>
    </tr>
    <tr>
        <td>Mimicked Result</td>
        <td><audio src="public/audios/results/artist/Closetoyou_result+mr.wav" controls></audio></td>
    </tr>
</table>

<br>
