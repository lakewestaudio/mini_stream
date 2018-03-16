# Serial Communication API for LakeWest Mini Streamer

get command returns push
set command returns push to confirm new value 
push command to Mini Streamer is not a valid cmd, its only pushed back by Mini Streamer to return/confirm set value.

Terms definition:
Streamer Software - any software running on LakeWest Mini Streamer hardware using this open protocol
Mini Streamer - LakeWest hardware - connected onto CM3 UART port

UART is running at 115200kbps, no parity, 1 stop bit. Using pins GPIO32 (TX to MCU) and GPIO33 (RX from MCU).

WHITE network LED is connected on pin GPIO37, RED network LED is connected on pin GPIO38. 
Pin GPIO39 is MCU reset and must be set as input, unless UART bootloader cant be accesed and needs to be forced with pin GPIO35. GPIO34 shall be HIGH when CM3 finished booting and should go LOW when its safe to cut off CM3 power supply.

GPIO43 is lan reset, most rpi firmwares take care of that (just needs to be defined in dtblob). 
GPIO40 when low shuts down the WiFi radio (by default must be forced high). 
GPIO41 is a wake pin from the WiFi module (its purpose is not really known RN).
            
#### USB DAC port

```shell
[getUSB]{}
```

```shell
[setUSB]{activeRoute:2,powerShutdown:0}
```

```shell
[pushUSB]{activeRoute:2,powerShutdown:0}
```
activeRoute:

	0   Streamer - DAC USB is routed to the CM3 USB host port,	
	1   USB pass-thru - cleaner mode, DAC USB is routed to back panel device USB,		
	2 - 255 reserved for future use.
  
powerShutdown:

	0   +5.2V USB output clean power for DAC is enabled,	
	1   disables power for USB dac, keeps data going (user has to power DAC from different supply)		
	2 - 255 reserved for future use.

#### ADC REEDBACK

```shell
[getADC]{port:1}
```

```shell
[pushADC]{voltage:522,current:496,power:259}
```

port:

	0   "USB1",	
	1   "USB2",	
  	2   "USB3",	
  	3   "PWR"
	
	voltage - return voltage with 2 decimal point precision - e.g. 522 is 5.22V, 1856 is 18.56V etc..
	current - recurns current in mA - e.g. 496 is 496mA
	power - returns power with 2 decimal places - 259 is 2.59W
	
#### ADC CALIBRATION

```shell
[getADCcal]{port:1,type:0} - reads back calibration value with:
```
```shell
[pushADCcal]{adcCal:+-xxxxxxx}
```

```shell
[setADCcal]{port:1,type:0} - calibrates port
```

```shell
[pushADCcal]{confirmCalibrationStart}
```

```shell
[replyADCcal]{y} OR [replyADCcal]{n}
```
port:

	0   "USB1",	
	1   "USB2",	
  	2   "USB3",	
type:

	0   "OFFSET",	
	1   "RDSON",	only valid for USB2 and 3
  	2   "VOLTAGE",	

Calibration will timeout after 5s without reply

#### ADC CALIBRATION - recall factory cal defaults

```shell
[getADCdefaults]{port:1} - reads out factory defaults
```

```shell
[setADCdefaults]{port:1} - sets back factory defaults for specific port
```

```shell
[pushADCcal]{offset:xxxx,voltage:-xxxx,rdson:80} 
``` 

RDSon is sent back only for USB 2 and 3


#### ADC CALIBRATION - set factory cal defaults 
##### DONT USE, ONLY FOR PRODUCTION PURPOSES!!!

```shell
[setADCfactory]{I know what I'm doing}
```

```shell
[pushADCfactory]{confirmCalibrationDefaults}
```

```shell
[replyADCfactory]{y} OR [replyADCfactory]{n}
```

Calling setADCfactory will load current calibration profile for all ports into internal memory as factory defaults.

#### POWER/UNIT STATUS

```shell
[getPowerStatus]{}
```

```shell
[pushPowerStatus]{status:0}
```

status:

	0   Everything is OK,	
	1   Input voltage too low, audio performance could be degraded, connect 18V power supply
  	2   Unit failure, input regulator could be damaged
  	
	
#### Mini Streamer HW BOOT CONTROL

```shell
[pushBoot]{boot:0}
```

```shell
[setBoot]{boot:0}
```

boot:

	0   note defined,	
	1   puts CM3 into USB bootloader mode where new Streamer Software image can be uploaded over USB	
	
Bootloader mode has to be switched off by user presing "USB" toggle button on the unit front panel

	
#### POWER OFF CONTROL

Is pushed from the Mini Streamer HW to Streamer Software to signal user has switched off the unit with hardware front panel button. Streamer Software then needs to start shutting down. After its mostly ready to cut the power, it sets turn off with cmd set to 1. DevDAC confirms with push cmd:1 and then cuts the CM3 power in 10 seconds.

```shell
[getTurnOff]{}
```

```shell
[setTurnOff]{cmd:0}
```


```shell
[pushTurnOff]{cmd:0}
```
cmd:

	0 - start shutting down - either user pressed a hardware power button or is shutting it down thru the Streamer Software
	1 - unit is ready to cut the power in 10 seconds

#### Mini Streamer HW VERSION

```shell
[getBoardVersion]{}
```

```shell
[pushBoardVersion]{boardRev:12,firmwareRev:11}
```

#### STREAMER VERSION

```shell
[getStreamerVersion]{}
```

```shell
[pushStreamerVersion]{version:1223}
```

```shell
[setStreamerVersion]{version:1223}
```
#### TODO:

Define UART bootloader for onboard firmware upgrade
