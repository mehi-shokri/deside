# DESIDE
A tool for side chanel data leakage attacks. Alpha.


## Install

```
$ git clone https://github.com/zadewg/deside/
$ cd deside
$ pip install -r requrements.txt
$ git clone https://github.com/kevinpt/ripyl
$ cd ripyl
$ python setup.py install
```
<!--
### Early tests
For a brief overlook at some of the program's key features, run it like so. Note that the program is on a very early development stage.

``python3.6 main.py``
-->

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

` Theoretically, any single ended signaling electrical information transmision can be exploited using this approach. Differential signaling should be used to harden a line against this attack vector.`

&nbsp; 

The PS/2 signal represents an appealing and relatively favourable target for
eavesdropping. The main advantage is its serial nature as data is transmitted
one bit at a time. Also, PS/2 use is widely spread across IoT solutions and Critical Systems, such as [ATM](https://www.alibaba.com/product-detail/Vandalproof-16-Keys-Stainless-Steel-Keyboard_60817438401.html?spm=a2700.galleryofferlist.normalList.23.47c676f0u6gqsh) or  [Security Access Control.](http://www.seaelectronica.com.ar/lectores-rfid/para-pc/rfid-ps2)

&nbsp; 

**Cable Pinout**

<img align="right" src="/overview/ps2pinout.png" title="Angular" hspace="20" height="150" width="150"/>

<pre>                             
- Pin 1:  +DATA    Data                       
- Pin 2:           Not connected         
- Pin 3:  GND      Ground                 
- Pin 4:  Vcc      +5 V DC at 275 mA       
- Pin 5:  +CLK     Clock                 
- Pin 6:           Not connected              
</pre>
<!--
\- Pin 1:  +DATA    Data                       
\- Pin 2:           Not connected         
\- Pin 3:  GND      Ground                 
\- Pin 4:  Vcc      +5 V DC at 275 mA       
\- Pin 5:  +CLK     Clock                 
\- Pin 6:           Not connected              
-->



As wires are very close and not shielded against each other it is theorized
that a fortuitous leakage of information will go from the data wire to the ground
wire and/or cable shielding due to electromagnetic coupling.

The ground wire as well as the cable shielding are routed to the main power
adapter/cable ground which is then connected to the power socket and finally
the electric grid. This eventually leads to keystroke leakage to the electric grid which can then
be detected on the power outlet itself, including nearby ones sharing the same
electric line.

Being in the VLF range, the clock frequency of the PS/2 signal is lower than any other component or
signal emanated from a computer (everything else is typically above the MHz), this
allows noise filtering and signal extraction.

&nbsp; 

In order to implement the attack the ground from a nearby (same electric system within a reasonable range)  power socket is
routed to an oscilloscope used as an ADC using a modified power cable which
separates the ground wire for probing and includes a resistor between the two
probe hooks. The current dispersed on the ground is measured using the voltage
potential difference between the two ends of the resistor.

In order to accomplish the measurement a "reference" ground is needed, as any
ADC would need a proper ground for its own operation but at the same time the
electrical grid ground is the target of our measurements. Because of this the
main ground cannot be used as the equipment ground, as that would lead to null
potential difference at the two ends of the probe.

A "reference" ground is any piece of metal with a direct physical connection to
the Earth, a sink or toilet pipe is perfect for this purpose and easily reachable (especially if you are hacking the ATM from the bank branch restrooms or going full-spy on a hotel room).

&nbsp; 


**Setup diagram**

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

In conclusion, results clearly show that information is indeed leaked to the power grid and keystrokes can be remotely measured and processed with great reliability.


&nbsp; 


**Other potential uses for this software might be:**
- Touchless, passive, PCB/IC micro probing.
- Over-The-Air data leakage analysis, wireless receiver (ADC connected to an antenna).
- General TEMPEST.
- General scientific data processing.


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

* [Sniff keystrokes with lasers/voltmeters -Defcon 2009](https://dev.inversepath.com/download/tempest/tempest_2009.pdf)

---

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

**mapez** - [telegram](https://t.me/mapezz)
