# iPad 3 iOS 10 Port Research

Research repository investigating whether genuine iOS 10 can run on iPad 3,1 (J1AP, Apple A5X).

## Start here
- [General research and project handoff](research.md)
- [Hardware](research/hardware.md)
- [Environment](research/environment.md)
- [Kernelcache and IMG3](research/kernelcache.md)
- [Compatibility](research/compatibility.md)
- [Methodology and preservation rules](research/methodology.md)
- [Status and next steps](research/status.md)

## Current state
Read-only reconnaissance. The iPad remains on iOS 9.3.5 (13G36). A copy of its kernelcache is on the development Mac. Its leading bytes are consistent with an IMG3 container, but the container has not yet been parsed and the payload has not been extracted.

No iOS 10 port has been built or booted. No device-side modification is part of the current research stage.

## Documentation policy
When a discovery is confirmed or a finding changes, update the relevant topic file and root research.md. Preserve raw output, provenance, dates, and confidence labels. Automatic documentation updates are a future goal; no updater is currently active.
