<p align="center">
    <img src="https://i.imgur.com/oK6d604.png">
</p>

# MMS Project: Music Recognition Software
> did you know that shazam uses your location data? crazy right!?
> just use shazam 2! 

A Java-based music recognition software much like “Shazam”, which detects the sources (or song names) of small audio clips recorded through an input device.

# Index
- [Vision](https://github.com/nur1popcorn/MMSGroupProject#vision)
- [Build instructions](https://github.com/nur1popcorn/MMSGroupProject#build-instructions)
- [Usage](https://github.com/nur1popcorn/MMSGroupProject#usage)
- [Approach](https://github.com/nur1popcorn/MMSGroupProject#approach)
    - [Problems](https://github.com/nur1popcorn/MMSGroupProject#problems)
    - [Getting the fingerprint](https://github.com/nur1popcorn/MMSGroupProject#getting-the-fingerprint)
    - [Finding the song](https://github.com/nur1popcorn/MMSGroupProject#finding-the-song)
    - [Web-UI](https://github.com/nur1popcorn/MMSGroupProject#web-ui)
- [Technologies](https://github.com/nur1popcorn/MMSGroupProject#technologies)
    - [Programs](https://github.com/nur1popcorn/MMSGroupProject#programs)
- [Features](https://github.com/nur1popcorn/MMSGroupProject#features)
- [Tests](https://github.com/nur1popcorn/MMSGroupProject#tests)
- [Conclusion](https://github.com/nur1popcorn/MMSGroupProject#conclusion)
    - [Shortcomings](https://github.com/nur1popcorn/MMSGroupProject#shortcomings)
    - [Strengths](https://github.com/nur1popcorn/MMSGroupProject#strengths)
    - [Thoughts](https://github.com/nur1popcorn/MMSGroupProject#thoughts)
- [Resources](https://github.com/nur1popcorn/MMSGroupProject#resources)
- [Authors](https://github.com/nur1popcorn/MMSGroupProject#authors)

# Vision
Our goal was to create a web-based platform where you can record short audio clips that will be sent to a server through an interface. The server will then check if a matching song is in the library and hopefully respond with the corresponding name of the song. 

We did not intend to make it as pretty or as functional as Shazam, but rather, to prove that it can be done and it isn't earth-shattering alien technology. Plus, it's not such a typical problem to solve and we thought it might be fun.

Note: There will be **no** neural networks or deep learning in this project. That's too much.

# Build instructions
Designed to be built with [Maven](https://maven.apache.org/). Install maven and instruct it to enter the package lifecycle

```ssh=
sudo apt install maven

mvn package
```

# Usage

There are already some songs of our choice included in the project, however, if you want to add your own songs just follow these steps:

After building, open up [core.src.main.java.com.jkmt.mmsproject.preprocessing](https://github.com/nur1popcorn/MMSGroupProject/tree/master/core/src/main/java/com/jkmt/mmsproject) and run **MP3Converter.java**

This will create the "mp3" and "wav" folders you need for using this program.

Then, place the songs you want to fill the database with into the "mp3" folder. 
Those songs are then supposed to be converted to wav, and then used to generate the hashes we keep in the database.

Run in the following order:
* MP3Converter.java - Converts your files to wav
* HashGenerator.java - Processes and hashes the wav files

Then, open up [web.src.main.java.com.jkmt.mmsProjectWeb](https://github.com/nur1popcorn/MMSGroupProject/tree/master/web/src/main/java/com/jkmt/mmsProjectWeb) and run **MmsProjectWebApplication.java**

This starts the server which can be then reached by visiting **localhost:8080** in your web browser of choice (except if you want the fancy animations to work, we don't recommend IE or Safari).

Done! Click the big red button to start recording, wait a few seconds, and click it again. Your song should show up!

<p align ="center">
    <img src="https://i.imgur.com/5WbCXiZ.png">
</p>

# Approach 
The way Shazam works is based on creating a **fingerprint** for each audio clip. It uses your microphone to gather a short sample of the audio you hear. It creates an *acoustic fingerprint* based on that recording and compares it against their database of already-created song fingerprints for a match. 

### Problems
Our program needs to solve the following problems:
* being Noise/Fault tolerant:
    * because the music recorded by a phone in a bar/outdoor has a **bad quality**
    * because of the cheap microphone inside a phone that produces **noise/distortion**
* fingerprints needs to be **time invariant**; the fingerprint of a full song must be able to match with just a 10-second record of the song
* fingerprint matching need to be somewhat **efficient**. We don't wanna be waiting for hours.
* minimize the number of **false results** - making each fingerprint as recognizable as possible.

### Getting the fingerprint 
At the core of this problem lies the creation of this "acoustic fingerprint". Once we have recorded our song, we are left with an array of float values, representing all of our samples. The typical representation of audio over time is a spectrogram, just like this:

<p align="center">
    <img src="https://i.imgur.com/zyqW2K1.png">
</p>

Although it is very easy to identify the peaks of the audio, it would be naive to try to use our microphone recording raw data as a fingerprint, since there are a lot of noise-filled frequencies. Not only that, but there are way too many samples and a lot of redundant frequencies that don't show us a clear signature of the song we've just heard.

Instead, we aim to **simplify** our data to only significant frequencies, and focus on reducing the number of samples in total because we really don't care about each single one. In an ideal world, we would be left with something that would look more or less like this:

<p align="center">
    <img src="https://i.imgur.com/E0Yb6kE.png">
</p>

So that's not gonna be too easy. But with a little research we've come up with something.

There are four major steps to creating a fingerprint:
* **Converting the signal to mono**
    * Not all devices have monophonic microphones. That can lead to the left channel being pointed to the speaker, capturing the audio we want, but the right channel being quiet and filled with noise instead. We can't simply select one of the channels to use, rather we combine those stereo channels into one mono channel, which will then more reliable when comparing.
* **Cutting off unnecessary frequencies with a filter**
    * We are not concerned with all frequencies contained in our recording. The most noticeable part of a song are actually the mid and low frequencies, those tell you about the rhythm of a song, which is very easy to see when looking at the peaks. Therefore, we cut off all frequencies above 5000Hz.
* **Sub-sampling** 
    * Reducing our array of samples so that we are only left with the louder, more noticeable ones. The so-called "peaks". To do this, every 4 samples, we sum those 4 samples up. There are definitely more sophisticated methods to do this. 
* **Fourier-transform**
    * This is where Fast-Fourier-Transform comes in. Fourier transformation applies to discrete signals and gives a discrete spectrum, in other words, the frequencies inside the signal. But fourier transformation is a huge, complicated process. Since we want efficiency, we're gonna use Fast-Fourier-Transform, which is an efficient implementation of Discrete-Fourier-Transform!  This is what transforms our simplified samples into a simplified spectrogram, which is what we store as our "fingerprint!"

In the end, our simplified spectrogram is (again, in an ideal world) going to look something like this:
<p align="center">
    <img src="https://i.imgur.com/6E5cvHL.png">
</p>

### Finding the song    
Now that we have our fingerprints, we've arrived at our last step. We compare different points from our recorded spectrogram with our stored spectrogram database, and we see which corresponding song returns the strongest score, and if it is satisfiable (above a certain tolerance threshold).

Then, we return the matching song and voila!

### Web-UI
Concept: a large recording button. Vector graphics of a microphone, turns into a stop symbol once clicked. Fancy rotating animation when hovering over the button.

Realization: HTML + Javascript + CSS

# Technologies
* Java (backend code - fingerprint creation and matching)
* HTML (frontend code - web API)
* CSS (frontend code - web API)
* JavaScript (frontend code - web API)
* [Maven](https://maven.apache.org/) (Automation tool - dependency management)
* [SpringBoot](https://spring.io/projects/spring-boot) (Java Framework - dependency management, micro service architecture)
* [Markdown](https://en.wikipedia.org/wiki/Markdown) (Markup language - creating documentation)

### Programs
* IntelliJ (Java)
* WebStorm (HTML, CSS, JS)
* Adobe Illustrator (SVG)
* Paint (Wonderful cover art)


# Features
Successful implementations:

[`core:`](https://github.com/nur1popcorn/MMSGroupProject/tree/master/core)
* MP3 -> WAV automatic conversion, to prepare for processing 
* Stereo -> Mono conversion by combining the left and the right channels
* Lowpass filtering, able to cut off any frequencies. Currently set to 5000Hz, can be modified freely in the backend
* Sub-sampling algorithm, by summing up every four samples
* Fast-Fourier-Transform successfully implemented
* Hash generation that takes tiny amounts of memory
* Fingerprint matching 

[`web:`](https://github.com/nur1popcorn/MMSGroupProject/tree/master/web)
* A simple and effective interface that *"just works"*. Better than writing silly commands, people like buttons!
* Getting microphone sound through web-api and processing it in JavaScript
* Recording is done through the web client, and then sent to the server for fingerprint matching
* Returns the name of the corresponding song
* Pretty and animated (CSS!)


# Tests
Converting songs and generating hashes takes about 1-2 seconds per song. Once we have the hashes, we do not need to keep the mp3 or the wav files anymore, saving up some significant space.

In fact, after hashing, our fingerprints take up about ~1/1000 of the space that the mp3 files would (assumming stereo, 256kbps quality)

**Test 1: 10 songs | same genre | 5 second recordings | quiet environment**

Number of test runs: 30
Result: 83% success (5 wrong, 25 correct)
Notes: Struggled with a specific song, which had a similar drum beat to another one in the database. 

**Test 2: 5 songs | different genres | 5 second recordings | quiet environment**

Number of test runs: 20
Result: 95% success (1 wrong, 19 correct)
Notes: The only wrong result was at the intro of a song, which was just a single drum beat. Not very fair to the algorithm 
Seems like the algorithm performs very well with a small database of distinct genres

**Test 3: 50 songs | different genres | 5 second recordings | quiet environment**

Number of test runs: 30
Result: 90% success (3 wrong, 27 correct)
Notes: One of the wrong results was a recording of a speech that was part of a song, with no instruments. Perhaps that is not very clear to our algorithm
Performed very well even with a larger database.
Size of hashes: only 420KB
Size of mp3s: 393MB

**Test 4: 250 songs | completely mixed | 5 second recordings | noisy environment**

*The ultimate test.*

Number of test runs: 50
Result: 30% success (35 wrong, 15 correct)
Notes: Time spent converting and hashing: ~5 minutes and 15 seconds. Ouch.
Background noise was significant (window open, playing video of coffee shop background noise from youtube) but the song was still very loud and clear to me in person.
Alternatively the problem might be the very large number of songs in the database. There's a lot of blurry lines and seems like our current method just isn't accurate enough to handle this.
There was no noticeable pattern to the errors or the correct results. It was just struggling.



# Conclusion
### Shortcomings
There are some cases that our program is not well-equiped to deal with. This may be due to our simplistic approach to processing the microphone recording. A noisy microphone that does not capture enough low-frequencies is one such problem. And we're not well equipped to deal with a large database of songs.

A possible way to fix this would be to implement [volume normalization](https://en.wikipedia.org/wiki/Audio_normalization), [noise reduction](https://en.wikipedia.org/wiki/Noise_reduction), and a [multiband compressor](https://en.wikipedia.org/wiki/Dynamic_range_compression#Multiband_compression) to balance out the frequencies. 

However, these are far from trivial to implement, and would require a lot more time than the scope of our team project allows.

Another issue is that our program is **over-confident**. Even if the recording is short, or even if you give it silence, it's going to name a song. It would be better to implement a method that can identify "poor" recordings and disregard them.

Lastly, it just takes quite a long while to convert and hash. In the meantime we're also using a lot of disk space because of the wav files. Not ideal if our database were to have thousands of songs..

### Strengths
Given the right conditions (distinct songs, good microphone, no background noise), our program produces very good results with high success rates. Although our approach is quite simplistic, seems like we've done something right.

Although we were initially frightened at the complexity of creating a fingerprint which solves the aforementioned [problems](https://github.com/nur1popcorn/MMSGroupProject#problems), we were able to do so quite well, at least on a rudimentary level. 
Some might say it is a wonder that this works at all. 

The Web-UI created quite a few struggles during our journey, but in the end it is actually quite pretty and definitely more interesting than a Java Swing application. The animations are smooth and provide satisfying feedback to user input.

### Thoughts

Overall this was a fun project, but I don't think we could have foreseen how difficult working with sound in this kind of application would be. There are a lot of problems to think about when you have samplerate, audio format, amplitude, frequencies, sound quality and other real-life physics problems to keep in mind. 

The Java sound API isn't amazing, and we chose to not rely on other people's code for our processing. In the end, we had a lot of weird bugs that we could only understand by pure force of trial and error. 

But it feels like we touched on some base concepts of Multi Media systems. For one, this is a practical project, because Shazam is such a widely used and praised service. Most of this project is closely related to Audio (quite obviously) but the fun part is that we got to do some HTML with CSS animations too. 

In the end, it feels very satisfying to stop recording and see the name of your song show up on the screen.

# Resources
* https://www.youtube.com/watch?v=kMNSAhsyiDg
* http://coding-geek.com/how-shazam-works/
* https://en.wikipedia.org/wiki/Fast_Fourier_transform
* http://audio.rightmark.org/lukin/pub/aes_adapt/images/KobylaLin.png
* https://willdrevo.com/public/img/posts/dejavu-post/spectrogram_peaks.png
* http://coding-geek.com/wp-content/uploads/2015/05/shazam_filtered_spectrogram-min.png
* https://www.w3schools.com/css/css3_animations.asp
* https://www.markdowntopdf.com/
* Special thanks to StackOverflow 🤠

# Authors 
- **Joachim Waldl**
- **Keanu Pöschko**	
- **Marcel Brunnbauer**
- **Teodor Dumitrache**
