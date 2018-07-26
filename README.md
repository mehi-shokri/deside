# deside COMING-SOON
A tool for side chanel data leakage attacks. Pre-alpha.


## Prerequisites

See requirements.txt


### Early tests.

Explain what these tests test and why

```
python3.6 main.py
```

## Program flow.

* Data source parsing
* (Optional) Unasynchronous source correction.
* (Optional) Discrete time iFFT based 50-60 Hertz power grid noise reduction. 
* Discrete time FIR based band pass filter to isolate target frequency.
* (Optional) Signal smoothing. See [FILTERS](https://github.com/zadewg/deside-COMING-SOON/edit/master/FILTERS.py)
* Digital conversion with hystheresis.
* (Optional) Packet decoding, checksum.
* (Optional) Statistical colision analysis on decoded ps2 packets.
* (Optional) Statistcal frequency analysis on decoded messages.


## Attack scenario.

*Coming soon.



### Built With

* [Ripyl](https://github.com/kevinpt/ripyl) - Protocol decode and synthesis library.


### Author

* **mapez** - [telegram](https://t.me/mapezz)


### License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details


### Acknowledgments

* [Sniff keystrokes with lasers/voltmeters -Defcon 2009](http://www.blackhat.com/presentations/bh-usa-09/BARISANI/BHUSA09-Barisani-Keystrokes-SLIDES.pdf)
