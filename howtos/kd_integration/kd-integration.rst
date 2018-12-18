.. _KD-integration:

Keyword Detection integration
#############################

Keyword Detection (KD) a.k.a Voice Activation a.k.a Sound Trigger is a feature
that allows to trigger activity of speech recognition engine depending on
successful detection of a predefined keyphrase(keyword). Primary motivation to
offload keyphrase detection algorithm to embedded processing environment (i.e.
dedicated DSP) is reduction of a system power consumption while listeining for
an utterance.

The terms Voice Activation and Keyphrase Detection are
often used interchangebly to describe end-2-end system behaviour including:
* Keyphrase detection algorithm
* Keyphrase enrollment (parametrization of keyphrase detection algorithm)
* Audio stream management that is used to transport utterances 
* Minimization of system level power consumption during keyphrase detection
* System wake up on keyphrase detection

The term "Keyphrase Detection Component" typically is used to identify a
firmware processing component that implements alogrithm for detection of a
keyphrase in an audio stream. This section describes steps to be taken in order
to integrate Keyphrase Detection componenton Intel cAVS platform. Depending on
system level requirements for keyphrase detection algorithm and speech
recognition engine different policies for keyphrase buffering and voice data
streaming may be applied. The document covers reference implementation
available in SOF. Specifically it covers functional changes and provides
outline to platform specific changes for low power flows enablement. 

Timing sequence
***************

.. uml:: images/kd-timing-diagram.pu
   :caption: Basic diagram for a timing sequence


As depicted on a diagram above keyphrase is followed by user command. In order 
to balance power savings and user experience the host (CPU) shall be activated 
only if keyphrase is detected. To reduce number of false triggers for user 
commands the keyphrase can be send to host for additional (the 2nd stage) 
verification. This requires in FW ability to buffer keyphrase. Keyphrase 
transmission to the host shall be as fast as possible (faster than real-time) to 
Reduce latency for system response.


FW topology
***********

.. uml:: images/kd-component-diagram.pu
   :caption: Basic diagram for FW components topology

The diagram provides overview to FW components / pipelines and HW components
that play role in KD flows. The components are responsible for:

1. Keyphrase Buffer pipeline

- DMIC DAI configures hw interface to input data from microphones 

- Keyphrase Buffer Managrer is responsible for management of a captured data
  from microphones. The audio buffer with historic audio data is implemented 
  as a cyclic buffer. On successfull detection of a keyphrase the buffer is 
  drained during a burst transmission to a host. 

2. Keyphrase Buffer pipeline

- Channel selector - there are 2 channels buffered (for later processing on
  host CPU) but typically keyphrase detection algorithms accept mono input there
  is a need for selection of channel used for processing. Decision on selected
  channel is made by platform integrator. The component can accept parameters
  from a topology file.
- Keyphrase detection algorithm - the algorithm that
  accepts audio frames and returns information if keyphrase is detected. Note
  that FW infrastructure allows to notify Keyphrase buffer component if the
  detection algorithm succeeded. 


3. Speech capture stream pipeline

- Host - the component configures DMA to host CPU and is responsible for a
  transmission from local SRAM to host CPU accessible memory.

Note that to optimize number of compute cycles the processed frame size (number
of audio samples) used by FW components shall be the same and is driven
keyphrase detection algorithm input format.

End-2-End flows
***************

.. uml:: images/kd-e2e-sequence-diagram.pu
   :caption: E2E flow for SW/FW components

*TBD detailed description as diagram is pending review*


Power Flows
***********

The section covers power flows enabled on Intel platforms with cAVS IP. 
It's of high importance to understand partitioning between SW and FW:


- SW is responsible for configuration of a host accessible registers and
  notification to FW that nothing is preventing FW from entering
  opportunistically into low power flows.

- FW is responsible for making internal
  decisions on power/clock gating HW subsection and entering into ultra-low power
  modes of operation.

.. uml:: images/sys-dev-states.pu
   :caption: Basic diagram for D0iX entry/exit for cAVS platforms


cAVS power flows
****************

.. uml:: images/sw-fw-hw-D0ix.pu
   :caption: Basic diagram for D0iX entry/exit for cAVS platforms

FW D0iX entry / exit flows
**************************

.. uml:: images/fw-D0ix-entry.pu
   :caption: FW activity for D0iX entry/exit for cAVS platforms

Memory Layout
*************

LP SRAM shall be opportunistically used for code and data specific for keyword
detection flow. At the beginning of LP SRAM shall be located structures
mandatory for low power sequencer flow (magic word and LP restore vector).
Followed by keyword detection algorithm and beginning of keyphrase buffer.

Scheduling considerations 
*************************
As keyword detection algorithm can work in parallel with other streams the 
preemption need to be applied.


Tasks:
******

* Scheduling
