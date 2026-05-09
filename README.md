# Flashforge 5M Pro CoPrint Config

Public configuration pack for Flashforge Adventurer 5M Pro with CoPrint feeder setup.

## Configuration Note

- The included .cfg set is currently configured for EX1 through EX4 (base KCM setup).
- To use additional feeders, enable and adjust the ECM configs based on your target setup:
	- `ecm_1.cfg` for EX5 through EX8
	- `ecm_2.cfg` for EX9 through EX12
- Update include lines and MCU serial mappings in `printer.cfg` and the relevant ECM file(s) to match your installed hardware.
