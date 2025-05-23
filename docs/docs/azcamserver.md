# AzcamServer

*azcamserver* is the main server application for the *azcam* acquisition and analysis package. It usually runs in an IPython window and is used to control data acquistion. 

## Configuration and startup

An *azcamserver* process is really only useful with a customized configuration script and environment which defines the hardware to be controlled.  Configuration scripts from existing environments may be used as examples. They are imported into a python or IPython session or use a startup script to create a new application.

An example code snippet to start *azcamserver* when using the *azcam-itl environment* is:

```python
# server-side (azcamserver)
import azcam
import azcam_itl.server
instrument = azcam.db.tools["instrument"]
exposure = azcam.db.tools["exposure"]
instrument.set_wavelength(450)
wavelength = instrument.get_wavelength()
print(f"Current wavelength is {wavelength}")
exposure.expose(2., 'flat', "a 450 nm flat field image")
```

Another example code snippet to start an *azcamserver* process is:

```
ipython -i -m azcam.server_itl --profile azcamserver
```

and then in the IPython window:

```python
instrument.set_wavelength(450)
wavelength = instrument.get_wavelength()
print(f"Current wavelength is {wavelength}")
exposure.expose(2., 'flat', "a 450 nm flat field image")
```

## Tools
AzCam's *tools* are used to define and control a system.  The supported systems include:

- Astronomical Research Cameras, Inc. gen1, gen2, and gen3 controllers. See [this link](https://www.astro-cam.com).
- STA Archon controllers. See [this link](http://www.sta-inc.net/archon).
-  OCIW Magellan CCD controllers (ITL version). See [this link](http://instrumentation.obs.carnegiescience.edu/ccd/gcam.html).
- ASCOM cameras. See [this link](https://ascom-standards.org). This code has been used for QHY, ZWO, and Moravian cameras.

### Exposure Tools

These tools defines the exposure interface.  The default tool name is *exposure*.

The *exposure* tool often coordinates the actions of the hardware tools such as *controller*, 
*instrument*, etc. For example, when *exposure* is initialized the tools in the 
*exposure.tools_init* list are initialized.  Similarly when *exposure* is reset, the tools in the 
*exposure.tools_reset* list are reset.

[Autogenerated docs for the base Exposure class](autocode/exposure.md)

#### Astronomical Research Cameras Controllers
```python
import azcam.server
from azcam.tools.arc.exposure_arc import ExposureArc
exposure = ExposureArc()
exposure.filetype = exposure.filetypes["MEF"]
exposure.image.filetype = exposure.filetypes["MEF"]
exposure.sendimage.set_remote_imageserver("localhost", 6543)
exposure.image.remote_imageserver_filename = "/data/image.fits"
exposure.image.server_type = "azcam"
```

#### STA Archon Controllers

```python
import azcam.server
from azcam.tools.archon.exposure_archon import ExposureArchon
exposure = ExposureArchon()
filetype = "MEF"
exposure.fileconverter.set_detector_config(detector_sta3800)
exposure.filetype = exposure.filetypes[filetype]
exposure.image.filetype = exposure.filetypes[filetype]
exposure.display_image = 1
exposure.image.remote_imageserver_flag = 0
exposure.add_extensions = 1
```

#### Magellan Controllers

```python
import azcam.server
from azcam.tools.mag.exposure_mag import ExposureMag
exposure = ExposureMag()
filetype = "BIN"
exposure.filetype = exposure.filetypes[filetype]
exposure.image.filetype = exposure.filetypes[filetype]
exposure.display_image = 1
exposure.image.remote_imageserver_flag = 0
exposure.set_filename("/azcam/soguider/image.bin")
exposure.test_image = 0
exposure.root = "image"
exposure.display_image = 0
exposure.image.make_lockfile = 1
```

#### ASCOM Exposure

```python
import azcam.server
from azcam.tools.ascom.exposure_ascom import ExposureASCOM
exposure = ExposureASCOM()
filetype = "FITS"
exposure.filetype = exposure.filetypes[filetype]
exposure.image.filetype = exposure.filetypes[filetype]
exposure.display_image = 1
exposure.image.remote_imageserver_flag = 0
exposure.set_filename("/data/zwo/asi1294/image.fits")
exposure.display_image = 1
```

#### Sendimage

The **exposure.sendimage** class supports sending an image to a remote host running an image server which receives the image.

```python
remote_imageserver_host = "10.0.0.1"
remote_imageserver_port = 6543
exposure.sendimage.set_remote_imageserver(remote_imageserver_host, remote_imageserver_port, "azcam")
```


### Controller Tools

These tools defines the camera controller interface. The default tool name is *controller*. 

[Documentation for the base Controller class](autocode/controller.md)


#### Astronomical Research Cameras Controllers

```python
import azcam.server
from azcam.tools.arc.controller_arc import ControllerArc
controller = ControllerArc()
controller.timing_board = "arc22"
controller.clock_boards = ["arc32"]
controller.video_boards = ["arc45", "arc45"]
controller.utility_board = None
controller.set_boards()
controller.pci_file = os.path.join(azcam.db.systemfolder, "dspcode", "dsppci3", "pci3.lod")
controller.video_gain = 2
controller.video_speed = 1
```

**Camera Servers**

*Camera servers* are separate executable programs which manage direct interaction with 
controller hardware on some systems. Communication with a camera server takes place over a 
socket via communication protocols defined between *azcam* and a specific camera server program. These 
camera servers are necessary when specialized drivers for the camera hardware are required.  They are 
usually written in C/C++. 

**DSP Code**

The DSP code which runs in the ARC controllers is assembled and linked with
Motorola software tools. These tools are typically installed in the folder `/azcam/motoroladsptools/` on a
Windows machine as required by the batch files which assemble and link the code.

While the AzCam application code for the ARC timing board is typically downloaded during
camera initialization, the boot code must be compatible for this to work properly. Therefore
AzCam-compatible DSP boot code may need to be burned into the timing board EEPROMs before use, depending on configuration. 

The gen3 PCI fiber optic interface boards and the gen3 utility boards use the original ARC code and do not need to be changed. The gen1 and gen2 situations are more complex.

For ARC system, the *xxx.lod* files are downlowded to the boards.

#### STA Archon Controllers
```python
import azcam.server
from azcam.tools.archon.controller_archon import ControllerArchon
controller = ControllerArchon()
controller.camserver.port = 4242
controller.camserver.host = "10.0.2.10"
controller.header.set_keyword("DEWAR", "ITL1", "Dewar name")
controller.timing_file = os.path.join(
    azcam.db.systemfolder, "archon_code", "ITL1_STA3800C_Master.acf"
)
```

#### Magellan Controllers

```python
import azcam.server
from azcam.tools.mag.controller_mag import ControllerMag
controller = ControllerMag()
controller.camserver.set_server("some_machine", 2402)
controller.timing_file = os.path.join(azcam.db.datafolder, "dspcode/gcam_ccd57.s")
```

**Camera Servers**

*Camera servers* are separate executable programs which manage direct interaction with controller hardware on some systems. Communication with a camera server takes place over a socket via communication protocols defined between *azcam* and a specific camera server program. These camera servers are necessary when specialized drivers for the camera hardware are required.  They are usually written in C/C++. 

**DSP Code**

The DSP code which runs in Magellan controllers is assembled and linked with
Motorola software tools. These tools should be installed in the folder `/azcam/motoroladsptools/` on Windows machines, as required by the batch files which assemble and link the code.

For Magellan systems, there is only one DSP file which is downloaded during initialization. 

Note that *xxx.s* files are loaded for the Magellan systems.

#### ASCOM Controller

```python
import azcam.server
from azcam.tools.ascom.controller_ascom import ControllerASCOM
controller = ControllerASCOM()
```

### Temperature Controller Tools

This tool defines the temperature controller interface. The default tool name is *tempcon*.

[Documentation for the TempCon class](autocode/tempcon.md)

#### CryoCon Testerature Controller

This tools supports Cryogenic Control Systems Inc. (cryo-con) temperature controllers. See http://www.cryocon.com/.

**Example Code**

```python
import azcam.server
from azcam.tools.cryocon.tempcon_cryocon24 import TempConCryoCon24
tempcon = TempConCryoCon24()
tempcon.description = "cryoconqb"
tempcon.host = "10.0.0.44"
tempcon.control_temperature = -100.0
tempcon.init_commands = [
"input A:units C",
"input B:units C",
"input C:units C",
"input A:isenix 2",
"input B:isenix 2",
"input C:isenix 2",
"loop 1:type pid",
"loop 1:range mid",
"loop 1:maxpwr 100",
]
```
### Instrument Tool

This tool defines the instrument interface.  The default tool name is *instrument*.

[Documentation for the Instrument class](autocode/instrument.md)

### Telescope Tool

This tool defines the telescope interface. The default tool name is *telescope*.

[Documentation for the Telescope class](autocode/telescope.md)

### Display Tool

This tool defines the image display interface. The default tool name is *display*.

```python
rois = display.get_rois(2, 'detector')  
display.display(test.fits')
```

#### SAO Ds9 Image Display Tool

This tool supports SAO's ds9 display tool running under Windows. See https://sites.google.com/cfa.harvard.edu/saoimageds9.

See https://github.com/mplesser/azcam-ds9-winsupport for support code which may be helpful when displaying images on Windows computers

```python
from azcam.tools.ds9.ds9display import Ds9Display
display = Ds9Display()
display.display("test.fits")
rois = display.get_rois(0, "detector")
print(rois)
```
[Documentation for the Display class](autocode/display.md)
