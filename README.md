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

The PS/2 signal represents an appealing and relatively favourable target for
eavesdropping. The main advantage is its serial nature as data is transmitted
one bit at a time. Also, PS/" is use is idely spread across IoT solutions, such as [ATMs](https://www.alibaba.com/product-detail/Vandalproof-16-Keys-Stainless-Steel-Keyboard_60817438401.html?spm=a2700.galleryofferlist.normalList.23.47c676f0u6gqsh) or Access Control Systems.

Cable Pinout:

<pre>
                                  
- Pin 1:  +DATA    Data                         ---- 
- Pin 2:           Not connected              / 6||5 \
- Pin 3:  GND      Ground                    | 4 || 3 |
- Pin 4:  Vcc      +5 V DC at 275 mA          \ 2  1 /
- Pin 5:  +CLK     Clock                        ----
- Pin 6:           Not connected              
</pre>


As the wires are very close and not shielded against each other it is theorized
that a fortuitous leakage of information goes from the data wire to the ground
wire and/or cable shielding due to electromagnetic coupling.

The ground wire as well as the cable shielding are routed to the main power
adapter/cable ground which is then connected to the power socket and finally
the electric grid.

This eventually leads to keystrokes leakage to the electric grid which can then
be detected on the power outlet itself, including nearby ones sharing the same
electric line.

The clock frequency of the PS/2 signal is lower than any other component or
signal emanated from the PC (everything else is typically above the MHz), this
allows noise filtering and keystrokes signal extraction.

In order to implement the attack the ground from a nearby power socket is
routed to the ADC using a modified power cable (remember the disclaimer) which
separates the ground wire for probing and includes a resistor between the two
probe hooks. The current dispersed on the ground is measured using the voltage
potential difference between the two ends of the resistor.

With "nearby" power socket we identify anything connected to the same electric
system within a reasonable range, distances are discussed in the results
paragraph.

In order to accomplish the measurement a "reference" ground is needed, as any
ADC would need a proper ground for its own operation but at the same time the
electrical grid ground is the target of our measurements. Because of this the
main ground cannot be used as the equipment ground, as that would lead to null
potential difference at the two ends of the probe.

A "reference" ground is any piece of metal with a direct physical connection to
the Earth, a sink or toilet pipe is perfect for this purpose (while albeit not
very classy) and easily reachable (especially if you are performing the attack
from an hotel room).

Diagram:

```
         power socket     power socket
    --------- : -------------- : ------------------------------ . . .
   |          ^                ^
   |          |                |                       -----------------
 -----        |                * -------------------> | Vin             |
  ---       ----               |                      |                 |
   -       | PC |              |                      |                 |
  gnd       ----               -                      |                 |
           /   / ps/2         | |                     |     Analog      |
    ps/2  /   /               | | ~ 150 Ohm           |        2        |
         /  mouse             | |  probe resistor     |     Digital     |
     keyboard                  -                      |                 |
                               |                      |                 |
                               * -------------------> | Vref            |
                               |                      |                 |
                             -----                     -----------------
                              ---  "reference" gnd
                               -
```           

Andrea "lcars" Barisani and Daniele Bianco's research shows there is no significant degradation of signal quality between 1 and 15 meter tests, suggesting that attenuation is not a concern at this range.

*It should be noted that attenuation coefficients for wire copper are often
estimated for much higher frequencies (>1Mhz) than the PS/2 signal, considering
a typical copper cable with a coefficient of 0.1 dB after 60m theoretically
(strong emphasis here) 50% of the signal survives. For reference a typical
leakage emission has an output power of ~1 nW (10^-9 Watt).*

In conclusion the results clearly show that information about the keyboard
keystrokes indeed leaks on the power grid and can be remotely detected.


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

Rigol, LeCroy, and some Tektronix osciloscopes formats are also supported by using the [CONVERSION](https://github.com/zadewg/deside/blob/master/CONVERSION.py) module.

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

## Importing deside examples

##### Working with data
``` python
from deside.DIMENSIONS import SourceData

csvdata = SourceData(CSV='oscil.csv')
x1 = csvdata.Xpoints
y1 = csvdata.Ypoints
matrix1 = csvdata.Matrix

arraydata = SourceData(X=[1,2,3,4], Y=[4,3,2,1])
x2 = arraydata.Xpoints
y2 = arraydata.Ypoints
matrix2 = arraydata.Matrix

matrixdata = SourceData(Matrix = [[1,4],[2,3],[3,2],[4,1]])
x3 = matrixdata.Xpoints
y3 = matrixdata.Ypoints
matrix3 = matrixdata.Matrix

assert x1 == x2 == x3
assert y1 == y2 == y3
assert matrix1 == matrix2 == matrix3
```

##### Checking if data has constant frequency
``` python
from deside.ASYNC import Sync
from deside.DIMENSIONS import SourceData


csvdata = SourceData(CSV='oscil.csv')
time = csvdata.Xpoints
print(Sync().check(time)) #returns True / False
```

##### Band stop filter
``` python
from deside.FFT import Fourier
from deside.DIMENSIONS import SourceData

csvdata = SourceData(CSV='oscil.csv')
values = csvdata.Ypoints
clean = Fourier().degrid(values, freq=[50, 60]) #removes 50, 60hz signals.
```
##### Signal smoothing
``` python
from deside.FILTERS import NoiseWork
from deside.DIMENSIONS import SourceData


csvdata = SourceData(CSV='oscil.csv')
signal = csvdata.Ypoints

noisy = NoiseWork(signal).AN 
moving_average = NoiseWork(noisy).MA
```
##### Digital conversion
``` python
from deside.THRESHOLD import hyst
from deside.DIMENSIONS import SourceData


csvdata = SourceData(CSV='oscil.csv')
signal = csvdata.Ypoints
time = csvdata.Xpoints

hysts = hyst(time, signal, 1, 60, 10).hysts 
```

### Acknowledgments

* [Sniff keystrokes with lasers/voltmeters -Defcon 2009](http://www.blackhat.com/presentations/bh-usa-09/BARISANI/BHUSA09-Barisani-Keystrokes-SLIDES.pdf)

---

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

**mapez** - [telegram](https://t.me/mapezz)
