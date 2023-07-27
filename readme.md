
# Configuring the P-NUCLEO-LRWAN3

## Configuring the gateway

•	Set the server network to the things network using the AT command `AT+PKTFWD= eu1.cloud.thethings.network,1700,1700 `  
•	Set the frequency band to EU433, that is, `AT+CH=EU433`  
•	Turn the log on using `AT+log= On`  


## Configuring the sensor (LRWAN_NS1)

The sensor or the end-node device was configured using the firmware package from the [St_website](https://www.st.com/en/evaluation-tools/p-nucleo-lrwan3.html#tools-software),I-CUBE-LRWAN. 
The AT_master project in the firmware package is required for configuring the sensor.  
•	Install Cube IDE from [St_website].  
•	Download and install the latest drivers for the sensor, link 009 or a later version.  
•	Open the AT_master project in the STM32_CUBE_IDE and apply the following changes.  
o	In the file “sys_conf.h” under the includes, enable the DEBUGGER, that is, change DEBUGGER to “1”.  
o	In the file “sys_app.c”, comment out the `DBG_init();` line.   
o	In “master_app.c”, change the frequency band to EU433, that is, change ` #define FREQ_BAND  to EU433`.    
o	Add the following instruction in the fuction Lora_SetDataRate of the lora_driver.c :    

 ```c
   ATEerror_t Lora_SetDataRate(uint8_t DataRate)
{
  ATEerror_t Status;
 int32_t var = DataRate; // Instruction à ajouter
  Status = Modem_AT_Cmd(AT_SET, AT_DR, &var);
        
               return (Status);
}
```  

Note: To enable ADR, you can change the function of the file, lora_driver.c  
 ```c
/*to adapt the data rate during transmission*/    
LoraCmdRetCode = Lora_SetAdaptiveDataRate(ADAPT_DATA_RATE_ENABLE);  
//LoraCmdRetCode = Lora_SetAdaptiveDataRate(ADAPT_DATA_RATE_DISABLE);
```

•	Save all changes made and Connect the sensor to your computer, build and run the project from the STM32_CUBE_IDE.  
•	Open your terminal emulation software such as tera_term and view the log.  

## Joining TheThingsNetwork

•	Go to thethingsnetwork website(https://www.thethingsnetwork.org/), create an account and login  
•	Register your gateway by entering the gateway EUI (which can be found under the device) and the frequency plan(EU433). Click on Register gateway.  
•	Create an application to add an end-device(sensor) by clicking on Add application and providing the necessary information such as application ID etc.  
•	Click on Add end device and provide the necessary information. Select enter registration details manually for this end device.  
•	Provide the APP EUI, DEV EUI,APP KEY (all provided under the device) and end-device ID.  
•	Make sure to select the same frequency plan for the end-device as the gateway (EU433).  
•	Click on Register end device.  
•	Open tera_term or any emulation software and the end-device should join the network and you should be able to view uplink messages being sent under the live data section on thethingsnetwork.  
Note: In the Cube IDE, select properties from the project tab, select C/C++ build ->settings->select MCU settings and check the “use float with printf and scanf options” as this will allow the use of new library functions used in the AT_master project source code.

## ENABLING SENSORS TO VIEW LIVE DATA

To enable the sensors we had to make changes to the “sys_sensors.c” and “sys_sensors.h” files since the code to enable the sensors on the lrwan_NS1 board was not included. 

In “sys_sensors.h” file, add the following bit of code to include the use of the lrwan_ns1 board; 
```c
#if defined (USE_LRWAN_NS1)
#include "lrwan_ns1_humidity.h"
#include "lrwan_ns1_pressure.h"
#include "lrwan_ns1_temperature.h"
And a reminder to add an “endif” statement at the bottom such as;
#endif  /* LRWAN_NS1 */
```

•	In the “sys_sensors.c” file, comment this line //IKS01A2_ENV_SENSOR_Capabilities_t EnvCapabilities;  
And uncomment the following lines;   
```c
void *HUMIDITY_handle = NULL;
void *TEMPERATURE_handle = NULL;
void *PRESSURE_handle = NULL;
```
•	In every function in the “sys_sensors.c”, include the use of the lrwan_ns1 board to enable the sensors.  
•	NB: Include the code at the beginning of the function after all variable declarations.  
So in the function void EnvSensors_Read(sensor_t *sensor_data), we include;   
```c
#if defined(SENSOR_ENABLED) || defined (LRWAN_NS1)
 	 BSP_HUMIDITY_Get_Hum(HUMIDITY_handle, &HUMIDITY_Value);
 	 BSP_TEMPERATURE_Get_Temp(TEMPERATURE_handle, &TEMPERATURE_Value);
  	BSP_PRESSURE_Get_Press(PRESSURE_handle, &PRESSURE_Value);
#endif /*USE LRWAN1*/  
```
And in the function void  EnvSensors_Init(void, we include;  
```c
 /*LRWAN_NS1/
#if defined(SENSOR_ENABLED) || defined (LRWAN_NS1)
 	 /* Initialize sensors */
  	BSP_HUMIDITY_Init(HTS221_H_0, &HUMIDITY_handle);
  	BSP_TEMPERATURE_Init(HTS221_T_0, &TEMPERATURE_handle);
BSP_PRESSURE_Init(PRESSURE_SENSORS_AUTO, &PRESSURE_handle);  
```
Also include the following code before the “/*Get Capabilities*/  line in the same function;   
```c
 /* Enable */
  /* Enable sensors by bamzy */
  BSP_HUMIDITY_Sensor_Enable(HUMIDITY_handle);
  BSP_TEMPERATURE_Sensor_Enable(TEMPERATURE_handle);
  BSP_PRESSURE_Sensor_Enable(PRESSURE_handle);
#endif  
```

After making all these changes, run the code on the NUCLEO-L073RZ sensor device and it should start to read live sensor data.