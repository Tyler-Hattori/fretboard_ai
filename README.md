# Teaching AI to recognize chord progressions based on their geometric representation on a guitar fretboard

## Overview

In this project, I utilize the encoder-only transformer model to classify the key of a given four-chord progression. From what I found online, most music analysis algorithms utilize frequency content to classify or generate music. This makes sense---I suppose in the simplest terms music can be compactly described as interesting combinations of frequencies that sound pleasing to the human ear. After all, music elements such as pitch, tone, harmony, timbre, melody, rhythm, and tempo are all described by frequency relations. However, this seems very unintuitive to me at a very fundamental level. When people learn how to play the guitar, or any intrument for that matter, they almost never bother to learn the frequencies of notes. Learning music is often geometric; we say that 'this' finger pattern sounds especially good next to 'that' finger pattern. For guitar specifically, we learn songs by reading guitar tablature (an example of guitar tab is shown in guitar_tab_example.png). A guitar chord can be represented by a vector of 6 numbers, where each number represents the fret held on each of the guitar's 6 strings.


## The Dataset

My dataset consists of 144,000 sequences of chords represented by their guitar tablature. Each sequence consists of N=4 chords, which I convert to a sequence of 4 tokens before feeding the input into the encoder. I use 132,800 sequences for training and 14,784 sequences for validation. After 20 epochs using a batch size of 32, the model can identify 4-chord sequences with 98% accuracy.

On the piano, musical scales are visually obvious because note frequencies gradually increase from left to right, where every 12th step doubles the frequency of the note (in music, two notes with 2:1 frequency ratio sound so pure together that we call them the same note. Because of this, for nomenclature we defined a 12 note C-C#-D-D#-E-F-F#-G-G#-A-A#-B-C pattern and repeated it continuously as frqeuencies exponentially increase. We use 12 steps because this coincidentally allows the 7th step to align with the frequency exactly halfway along the scale, the 5th step to align with the frequency exactly a third of the way along the scale, the 3rd step a quarter of the way, etc. The sharps exist because only 8 steps align with nice ratios---these are C-D-E-F-G-A-B-C. These are the common tones in music. When we change key, we change the starting note of the scale. There are 12 keys, but we often speak only in terms of the 8 main tones. Hence the term 'octave.' Chords consist of three notes in a given key with nice ratios). On the guitar, the unique tuning pattern of the strings allows for several different finger patterns to represent the same chord. This can be seen in the provided image 'fretboard_key_of_c.png.' For example, notice that the chord C-E-G can be played in many different ways. Since I want my model to learn these different representations, to create my dataset I first defined the 6 common ways to represent each of the 7 common chords in each of the 24 common keys of music (12 major and 12 minor. I should note here that the minor scale is the same as the major scale started on the 6th tone. I define them differently in my dataset because chord progressions are defined as major or minor). Then, I defined 39 of the most common chord progressions in the major scale and 43 of the most common chord progressions in the minor scale. For each of these progressions in a given key, I randomly created 150 different ways to play that progression on the guitar by randomly selecting different voicings for each chord. I used 150 because this would allow my dataset to be of length 147,600 (150 voicings * (39+43) progressions * 12 keys). I liked this number because it is analogous to the size of the CIFAR dataset where there are 6,000 instances of each class (6,000 * 24 possible keys = 144,000).

There are five main chord shapes on the guitar: C, A, G, E, and D. Each of these shapes can be made major or minor by changing one fret. Moreover, each of these shapes can be shifted to any fret on the guitar to represent the same chord in a different key. For now, I am only concerning myself with progressions that contain only major or minor chords. Therefore, I am able to define a counting system of 5*2*12 = 120 tokens that contain all common ways of playing any major of minor chord on the fretboard. In the project, I also include a sixth shape for chord representations beyond halfway down the guitar string. This brings the vocabulary length of the tokens to 150, and helps the model understand that chord shapes repeat after fret 12.

## The Model

The internal dimension of the model is 512 and the encoder loops 6 times. I used sparse categorical cross entropy loss, as my labels are numbers from 1 to 12. The self attention uses 8 heads and the dimension of every hidden feed-forward layer is 2048.

## Comments

Moving forward, I would like to add chord shapes with 7ths, 6ths, 9ths, sus, and more. This would simply require more training and a few more manually-defined chord progressions. Moreover, I would like to add a decoder that can generate tokenized melodies from the chord progression recieved by the encoder. However, to do this I would want the model to be able to understand timing and (possibly) instrumentation. I would therefore need to implement the compound word transformer, which I attempt in the next draft of this project. In the fourth draft I will add the decoder and in the fifth draft I will add instrumentation.

## References

Link to 'fretboard_key_of_c.png' screenshot source: https://upload.wikimedia.org/wikipedia/commons/1/1b/C_Major_Scale_on_fretboard.svg
Link to 'guitar_tab_example.png' screenshot source: https://s3.amazonaws.com/halleonard-pagepreviews/HL_DDS_0000000000093646.png

I took inspiration from the following papers:
- Dosovitski et al. An Image is Worth 16x16 Words: Transformers Image Recognition at Scale. (2021).
- Liu et al. Swin Transformer: Hierarchical Vision Transformer using Shifted Windows. (2021).
- Vaswani et al. Attention Is All You Need. (2017).
- Huang et al. Shuffle Transformer: Rethinking Spatial Shuffle for Vision Transformer. (2021).
