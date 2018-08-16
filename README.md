# deside COMING-SOON
A tool for side chanel data leakage attacks. Pre-alpha.


## Install

```
$ git clone https://github.com/zadewg/deside/
$ cd deside
$ pip install -r requrements.txt
$ git clone https://github.com/kevinpt/ripyl
$ cd ripyl
$ python setup.py install
```

### Early tests
For a brief overlook at some of the program's key features, run it like so. Note that the program is on a very early development stage.

``python3.6 main.py``

## Program features and flow

* Data source parsing
* Asynchronous source correction.
* Discrete time iFFT/notch based 50-60 Hertz power grid noise reduction. 
* Discrete time FIR based band pass filter to isolate target frequency.
* Signal smoothing. See [SMOTHING](#filters)
* Digital conversion with hystheresis.
* Packet decoding, checksum.
* Statistical collision attack on ps2 packets.
* Statistcal frequency analysis on decoded text messages.


## Attack scenario

*Coming soon.
[Tip](https://www.alibaba.com/product-detail/Vandalproof-16-Keys-Stainless-Steel-Keyboard_60817438401.html?spm=a2700.galleryofferlist.normalList.23.47c676f0u6gqsh)

<a name="filters"></a>
## Smoothing Filters
Animation showing SG smoothing being applied, passing through the data from left to right. The red line represents the local polynomial being used to fit a sub-set of the data. The smoothed values are shown as circles.
![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/89/Lissage_sg3_anim.gif/400px-Lissage_sg3_anim.gif)


* Moving average
* Moving median
* Direct form II transposed standard difference equation IIR/FIR
* Savitzky-Golay

See [FILTERS](https://github.com/zadewg/deside/blob/master/FILTERS.py) for details.

## Decoding

Suported protocols:
* PS/2
* CAN
* ETHERNET
* I2C
* ISO K-line (ISO 9141 and ISO 14230)
* J1850
* LIN
* LM73
* NEC Infrared
* OBD-2
* Philips RC-5 Infrared
* Philips RC-6 Infrared
* Sony SIRC Infrared
* SPI
* UART
* USB 2.0

Built with [Ripyl](https://github.com/kevinpt/ripyl) - Protocol decode and synthesis library.

## Suported input

Deside accepts input as bidimensional matrix in (time:value) format, separate time/value arrays, and csv files in time, value format, as in [oscil.csv](https://github.com/zadewg/deside/blob/master/oscil.csv). This is Hantekś default format.

Rigol, LeCroy, and some Tektronix osciloscopes formats are also supported by using the [CONVERSION]() module.

```
$ python3.6 CONVERSION.py -h

usage: CONVERSION.py [-h] -if INFILE -of OUTFILE -o OSCILLOSCOPE [-c CHANNEL]

Arguments:

  -h, --help            show this help message and exit
  
  -if INFILE, --infile INFILE
                        Input data filename
                        
  -of OUTFILE, --outfile OUTFILE
                        Output data filename
                        
  -o OSCILLOSCOPE, --oscilloscope OSCILLOSCOPE
                        Oscilloscope brand. Supported: Rigol, LeCroix, Tektronix
                        
  -c CHANNEL, --channel CHANNEL
                        Specify channel if neccesary. Default=1

```

*Example output:*

|     Time      |    Voltage    | 
|---------------|---------------| 
| 0.0000000000  | -0.4897959184 | 
| 0.0000100000  | -0.4693877551 | 
| 0.0000200000  | -0.4693877551 | 
| 0.0000300000  | -0.4693877551 | 
| 0.0000400000  | -0.4897959184 | 
| 0.0000500000  | -0.4693877551 | 
| 0.0000600000  | -0.4693877551 | 

## Importing deside
``` python
from deside import ASYNC, CONVERSION, DIMENSIONS, FFT, FILTERS, THRESHOLD, PCA
```

### Acknowledgments

* [Sniff keystrokes with lasers/voltmeters -Defcon 2009](http://www.blackhat.com/presentations/bh-usa-09/BARISANI/BHUSA09-Barisani-Keystrokes-SLIDES.pdf)

---

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

**mapez** - [telegram](https://t.me/mapezz)
