---
layout: index
navtitle: Home
title: RTLAMR
excerpt: An rtl-sdr receiver for smart meters operating in the 900MHz ISM band.
---

### Purpose
For several years now utilities have been using "smart meters" to optimize their residential meter reading infrastructure. Smart meters continuously transmit consumption information in the 900MHz ISM band allowing utilities to simply send readers driving through neighborhoods to collect commodity consumption information. The protocol used to transmit these messages is fairly straight forward, however I have yet to find any reasonably priced product for receiving these messages.

This project is a proof of concept software defined radio receiver for these messages. We make use of an inexpensive rtl-sdr dongle to allow users to non-invasively record and analyze the commodity consumption of their household.

Currently the only known supported and tested meter is the Itron C1SR. However, the protocol is designed to be useful for several different commodities and should be capable of receiving messages from any ERT capable smart meter.

### Requirements
 * GoLang >=1.2
 * GCC (on windows TDM-GCCx64 works nicely)
 * libfftw >=3.3
   * Windows: [http://www.fftw.org/install/windows.html](http://www.fftw.org/install/windows.html)
   * Linux (debian): libfftw3-dev
 * rtl-sdr
   * Windows: [pre-built binaries](http://sdr.osmocom.org/trac/attachment/wiki/rtl-sdr/RelWithDebInfo.zip)
   * Linux: [source and build instructions](http://sdr.osmocom.org/trac/wiki/rtl-sdr)

### Building
This project requires two other packages I've written for SDR related things in Go. The package [`github.com/bemasher/rtltcp`](http://godoc.org/github.com/bemasher/rtltcp) provides a means of controlling and sampling from rtl-sdr dongles. This package will be automatically downloaded and installed when getting rtlamr.

The second package needed is [`github.com/bemasher/fftw`](http://godoc.org/github.com/bemasher/fftw), which may require more effort to build. Assuming for linux you already have the necessary library, no extra work should need to be done. For windows a library file will need to be generated from the dll and def files for gcc. The FFTW defs and dlls can be found here: http://www.fftw.org/install/windows.html

#### On Windows

	go get -d github.com/bemasher/fftw
	dlltool -d libfftw3-3.def -D libfftw3-3.dll -l $GOPATH/src/github.com/bemasher/fftw/libfftw3.a
	go get github.com/bemasher/rtlamr

#### On Linux (Debian/Ubuntu)
	
	sudo apt-get install libfftw3-dev
	go get github.com/bemasher/rtlamr

This will produce the binary `$GOPATH/bin/rtlamr`. For convenience it's common to add `$GOPATH/bin` to the path.

### Usage
Available command-line flags are as follows:

	$ rtlamr -h
	Usage of rtlamr:
	  -centerfreq=920299072: center frequency to receive on
	  -duration=0: time to run for, 0 for infinite
	  -logfile="/dev/stdout": log statement dump file
	  -samplefile="NUL": received message signal dump file
	  -server="127.0.0.1:1234": address or hostname of rtl_tcp instance

Running the receiver is as simple as starting an `rtl_tcp` instance and then starting the receiver:

	$ rtl_tcp -a 0.0.0.0 &
	$ rtlamr

Output is as follows, note that the meter ID's and checksums have been obscured to avoid releasing potentially sensitive information:

	recv.go:540: Config: {ServerAddr:127.0.0.1:1234 Freq:920299072 TimeLimit:0 LogFile:/dev/stdout SampleFile:NUL}
	recv.go:541: BlockSize: 16384
	recv.go:542: SampleRate: 2.4e+06
	recv.go:543: DataRate: 32768
	recv.go:544: SymbolLength: 73.2421875
	recv.go:545: PacketSymbols: 192
	recv.go:546: PacketLength: 14062.5
	recv.go:547: CenterFreq: 920299072
	recv.go:117: BCH: {GenPoly:16F63 PolyLen:16 Syndromes:3240}
	recv.go:553: Running...
	{ID:17581### Type: 7 Tamper:{Phy:3 Enc:1} Consumption:  324064 Checksum:0x16##}
	{ID:17581### Type: 7 Tamper:{Phy:2 Enc:1} Consumption:  642693 Checksum:0x38##}
	{ID:17573### Type: 7 Tamper:{Phy:3 Enc:0} Consumption: 1179955 Checksum:0xEF##}
	{ID:17581### Type: 7 Tamper:{Phy:1 Enc:0} Consumption:  882699 Checksum:0xB8##}
	{ID:17581### Type: 7 Tamper:{Phy:2 Enc:0} Consumption: 1002678 Checksum:0xF4##}
	{ID:17581### Type: 7 Tamper:{Phy:2 Enc:1} Consumption:  642693 Checksum:0x38##}
	{ID:17581### Type: 7 Tamper:{Phy:2 Enc:1} Consumption:  867042 Checksum:0x39##}
	{ID:17573### Type: 7 Tamper:{Phy:3 Enc:1} Consumption: 3812203 Checksum:0xC2##}
	{ID:17573### Type: 7 Tamper:{Phy:3 Enc:0} Consumption: 1179955 Checksum:0xEF##}
	{ID:17581### Type: 7 Tamper:{Phy:3 Enc:0} Consumption: 1001648 Checksum:0x0E##}
	{ID:17581### Type: 7 Tamper:{Phy:1 Enc:0} Consumption:  882701 Checksum:0x28##}

Using the provided antenna in most DVB-T dongle kits, I can reliably receive consumption messages from ~15 different meters from my apartment alone, with error correction for up to two errors this jumps to ~19 meters. This will likely improve once adaptive preamble quality thresholding is implemented.

### Ethics
_Do not use this for nefarious purposes._ If you do, I don't want to know about it, I am not and will not be responsible for your lack of common decency and/or foresight. However, if you find a clever non-evil use for this, by all means, share.

### Feedback
If you have any general questions or feedback leave a comment below. For bugs, feature suggestions and anything directly relating to the program itself, submit an issue in github.

### Future

 * There's still a decent amount of house-keeping that needs to be done to clean up the code for both readability and performance.
 * Move away from dependence on FFTW. While FFTW is a great library integration with Go is messy and it's absence would greatly simplify the build process.
 * Implement direct error correction rather than brute-force method.
 * Finish tools for discovery and usage of hopping pattern for a particular meter. There's enough material in this alone for another writeup.
 * Implement adaptive preamble quality thresholding to improve false positive rejection.
