# Setting up EMP FWK

### Prequisites:
How to build the framework: https://serenity.web.cern.ch/serenity/emp-fwk/firmware/instructions.html.

You want to make sure you install the prerequisites:

* Xilinx Vivado: 2021.2
* Python 2.7 - available on most linux distributions, natively or as miniconda distribution.
* ipbb: dev/2022f pre-release or greater - the IPbus Builder Tool. Note: a single ipbb installation is not work area specific and suffices for any number of projects. Check EMP repo.

## Building FWK
1. Download ippb command files:
   `curl -L https://github.com/ipbus/ipbb/archive/dev/2022f.tar.gz | tar xvz`.

2. Source the following file as follows: `source ipbb-dev-2022f/env.sh`.

3. Run the `ipbb` commands specified on the link, which I also put inside the `EMP_setup.sh`. So, do `bash EMP_setup.sh`. (breaks down at the moment. Need authorizatios and github and gitlab keys? I added these on Scully, but they probably need to approved by Terence. I asked about this on Slack UCSD tier 2).
