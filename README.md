# [Unlimited Access] BPWin REST API Data User Guide

- Version: v1.2
- Revision History:

| Date | Changes Made |
|----------|----------|
| 06/02/2023 | DEMO meeting held |
| 07/17/2023 | Upgrading API package and optimized endpoints accordingly. |
| 12/05/2023 | Added authentication process. Changing document revision to v1.1 |
| 12/12/2023 | Updated endpoint "/health", "/data/selected-device" <br/> Updated field names in /ws - Socket Status <br/> Updated endpoint /data/job-stats <br/> Specified which endpoints can be used in manual or autohandler mode <br/> Updated Authentication section with code sample <br/> Change document revision to v1.2 | 
| 1/4/2024 | Updated Devkit Program code. |
| 4/1/2024 | Rectified Quick Starter section and started Github repo. |
| 4/9/2024 | Added Virtual Enviornment based Quick Starter Guide. |

## Quick Starter with DataPanel.py

### Setup Instructions
1. Install python 3.11, make sure to select 'add to path' and 'pip' during installation.
    
    https://www.python.org/downloads/release/python-3119/

        (Optional) If path of python was not added during installation, you can add it by:
        1. Search 'Environment Variables'
        2. Click on 'Environment Variables' under Advanced tab
        3. Select 'Path' on System Variable and click on Edit
        4. Click New and enter the scripts folder of installed python. Depending on installation.
        It may look like: 'C:\Users\UserName\AppData\Roaming\Python\Scripts'

2. Open CMD and Install Pipenv if not already done.
    ```sh
    pip install pipenv
    ```
3. Git Clone the project and go to the cloned project folder. Replace below 'to/the/path/of' with the location where the project was cloned:
    Install git if not already done, https://git-scm.com/downloads

    ```sh
    git clone https://github.com/rishavbpm/BPWinRestAPIData.git
    cd to/the/path/of/BPWinRestAPIData
    ```
4. Install below dependencies using pipenv:
    ```sh
    pipenv install
    ```
        (Optional) If you find error about python not found.
        Add python path to pipenv.
        Replace 'to/the/path/of/installed/' with the location where python was installed 
        (example: C:\Users\UserName\AppData\Local\Programs\Python\Python311\python.exe)

        ```sh
        pipenv --python to/the/path/of/installed/python.exe
        ```

5. Start Virtual Enviornment:
    ```sh
    pipenv shell
    ```
6. Make sure to have empty Auth.txt in "C:\BP" folder. If not, create one.

7. Run the web application
    ```sh
    streamlit run ./DataPanel.py
    ```
## Detailed setup instruction
### Python requirements
In order to utilize this starter program, some conditions need to be satisfied:
- Python version 3.8 or higher is installed on the computer
- Prerequisite python packages are installed:
  - plotly version 5.20.0
  - streamlit version 1.16.0

### Other setup requirements
- BPWin 8.0.1 or later version is opened on a PC. We call this PC running BPWin the **Server PC**. 
- The **Server PC** should:
  - Be connected to either a BPM APS machine, or at least a programmer site (preferably 9th Gen site, with APS simulation mode turned on)
  - Have an IPV4 address that is accessible by the **Client PC** (The PC running this DataPanel.py). Or the same PC is used as Client PC and Server PC.
- Identify the IPV4 address of the **Server PC**. If the same PC is used as Client PC and Server PC, the IP address is "localhost". **This DataPanel.py is using localhost as the IP Address**.

### Run DataPanel.py
Make sure all listed conditions are satisfied. Start by downloading "DataPanel.py". Edit it according to your settings: 
- Substitute "ip_address" with the Server PC IP address.
- Change "api_version" to the REST API version of BPWin. As for this current document, the BPWin REST API version is "1.2".
- Save the file in an accessible directory on your PC. For example, if the "DataPanel.py" is saved to "C:\", the full file path should be "C:\DataPanel.py".
- Open a cmd line prompt, type the command: ```streamlit run C:\DataPanel.py```
- The command window will show the url ```http://localhost:8501```. Normally your default browser will automatically open to show the real-time data panel. If that doesn’t happen, you can copy the url and paste it to your browser to view the panel.

## Authentication Process
This section talks about the authentication process for using the BPWin REST API endpoints. An authentication process prevents any unauthorized access to the BPWin running on your APS machines. 
### Setting Up API Passphrases
To begin using the BPWin API, you need to set up passphrases for each APS machine you intend to connect to. Follow these steps:
- Locate the BP Folder: First, navigate to the BP folder on your computer. 
  - To know where the BP folder is located, open the Registry editor. Go to "Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\BP Microsystems\BPWin\Directory Paths" and look for "BP" in it. Normally the folder should be "C:\BP". 
- Create a Passphrase File: For each APS machine, create a text file with your API passphrase in the BP folder. The name of the text file should be "Auth.txt". You can choose to use the same passphrase for multiple APS machines or set different ones for each. Ensure that the passphrase is secure and not easily guessable. File Requirements:
  - If the file is not present in the BP folder, the BPWin REST API feature will be disabled on that APS machine.
  - If the file is present but empty, it will be considered as opening the BPWin REST API on that machine to HTTP calls from anywhere.
- After you update the passphrase, you need to restart BPWin for it to take effect. 

### Initiating the Connection
Open BPWin as usual. Wait for it to finish initialization. Before making any other API calls, you need to initiate the connection by calling the /auth endpoint. Follow these steps:

- Make the First API Call: Use your preferred programming language or API client to make a GET request to the /auth endpoint of your BPWin. 
Code Example in Python

```python
import requests
response = requests.get("http://192.168.0.121:8080/auth")
print(response.text)
```

Result format

```{ "Number" : 7241978785468376164 }```

The result includes a Random Challenge Number generated by BPWin. BPWin generates this random challenge number and passes it back to your client application as a response. This challenge number changes with every opening of the BPWin. That means, if closing and re-opening BPWin, you need to call /auth again to get a new Random Challenge number. 

### Making API Calls
After successfully initiating the connection, you can start making other API calls. Follow these steps:
- Include Random Challenge Number and Passphrase in request Headers: For each API call, include the random challenge number received during the authentication step in the request headers. Additionally, include your APS machine's passphrase in the headers. Note that both should be in string format, surrounded by “”.
Code Example in Python (calling the /version endpoint): 

```python
import requests
Headers = { "Random-Challenge" : "7241978785468376164", "Passphrase" : "123123" }
response = requests.get("http://192.168.0.121:8080/version", headers = Headers)
print(response.text)
```

Result:
If it succeeds, it will return data from the endpoint. 
```
{
    "API_Version":"1.0",
    "BPWin_Version":"V8.0.0"
}
```

It will show the error message when failed
```
{
    "Success": 0,
    "Message": "Authentication failed. The passphrase or Random Challenge Number is incorrect."
}
```

- BPWin will verify the challenge number and password. If the verification succeeds, BPWin will send the requested data back to your client application, and the API call is completed. If the verification fails, BPWin will return an HTTP 403 error.
- Ensure that you maintain the security of your API passphrases to keep your APS machine interactions secure and reliable.


# API Endpoints
### GET: /health
*Manual* *Autohandler*

Getting whether there is connection to the REST API system of the BPWin. Calling this method checks if the connection to the REST API of BPWin is established. If the call does not return an HTTP code of 200(OK), it means some of the following conditions may not be met. 
BPWin is open. 
BPWin supports REST API functionality. 
The IP address of the PC running BPWin is correct and accessible. 

Result
```
{
    "Success": 1,
    "Message": "BPWin is ready."
}
```

Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/health")
```

### GET: /version
*Manual* *Autohandler*

Getting the version of the REST API system, including API version and BPWin version. For the scope of this documentation, the API version is 1.0.

Result
```
{
    "API_Version": "1.0",
    "BPWin_Version": "V7.0.9 DeviceSupportUpdate.99 (12/21/2022)"
}
```

Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/version")
```

## Job Summary Data
The Data Section consists of REST API endpoints that get data from BPWin through GET requests. The endpoints starting with /data/ can be called right after BPWin initialization, as long as there is data available. The endpoints starting with /data/job/ can only be called during a job session. 

### GET: /data/machine-name
*Manual* *Autohandler*

Get the information of the machine currently identified by BPWin. For an APS machine, the result will include its Type, Description and Serial Number. For a manual site, it will only indicate it’s manual. API functionalities are currently not eligible for manual mode. 

Result(JSON format)
When BPWin is connected with an APS. 
```
{
    "Type": "APS",
    "Description": "4000 APS",
    "Serial_Number": "H4911810120010"
}
```
When only manual sites are attached.
```
{
    "Type": "Manual",
    "Description": null,
    "Serial_Number": null
}
```

Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/data/machine-name")
```

### GET: /data/job-name
*Manual* *Autohandler*

Get the file names associated with this job, including BP, ABP and Data Pattern. For BP and ABP files, there will also be the paths on the PC. 

Result(JSON format)
When the corresponding files are not loaded, the path will indicate "No File Loaded". 
```
{
    "DataPattern_FileName": "No Data Pattern File Loaded",
    "BP_FileName": "No Job File Loaded",
    "ABP_FileName": "No APS Workflow File Loaded"
}
```
When there are corresponding files loaded, the paths of files will be displayed. 
```
{
    "DataPattern_FileName": "APITestDataPattern.bin",
    "BP_FileName": "C:\\APITestFile\\APITestBp.bp",
    "ABP_FileName": "C:\\APITestFile\\APITestAbp.abp"
}
```
Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/data/job-name")
```

### GET: /data/selected-device
*Manual* *Autohandler*

Get the name of the device currently selected in the BPWin instance. 

Result(JSON format)
When there is a job loaded or device selected, there will be a full name same as shown in the BPWin device selector. There are also separated Device and Manufacturer names. 
```
{
    "FullName": "Winbond W25N01GVZEIG",
    "Device": "W25N01GVZEIG",
    "Manufacturer": "Winbond"
}
```
When there is no device selected, the FullName field will show “No Device Selected”. The other fields will be empty. 
```
{
    "FullName": "No Device Selected",
    "Device": "",
    "Manufacturer": ""
}
```

Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/data/selected-device")
```

### GET: /data/job-stats
*Manual* *Autohandler*

Get the statistics of the job session. This data is the same as the job statistics parameters displayed on the BPWin sidebar while a job session is in progress. 
Machine Status: All status available are “Active”, “Idle”, “Paused” and “Error”. 
Light tower status: This includes Red, Amber and Green lights. The data is the same as shown on the physical light tower of the APS machine connected. 

Result(JSON format)
When no job session is in progress, the following message will return. 
```
{
    "Success": 0,
    "Message": "Job statistics are only available when job session is in progress. "
}
```
When BPWin is in manual mode, the format will be as follows.
```
{
    "Job_Yield": 0,
    "Elapsed_Time": "00:00:02",
    "Passed_Device": 3,
    "Failed_Device": 0,
    "Active_Device": 0,
    "Remaining_Device": 17
}
```

When BPWin is in autohandler mode, the format will be as follows. 
```
{
    "Machine_Status": "Active",
    "Red_Light_On": "false",
    "Amber_Light_On": "false",
    "Green_Light_On": "true",
    "Job_Yield": 0,
    "Elapsed_Time": "00:00:01",
    "Remaining_Time": "--:--:--",
    "Idle_Time": "00:00:00",
    "Actual_DPH": 0,
    "Potential_DPH": 0,
    "Passed_Device": 1,
    "Failed_Device": 0,
    "Active_Device": 0,
    "Remaining_Device": 19
}
```

Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/data/job-stats")
```

### GET: /data/system-info
*Manual* *Autohandler*

Returns a JSON string that describes the system’s current hardware and software configuration. The “system” for which this method returns information about includes the BPWin application, the currently-connected programmer, and the currently-connected Automated Programming System (APS).
See Appendix A for more information.

Example in Python
```python
import requests
requests.get("http://[IP Address and Port]/data/system-info")
```

# Websocket Endpoint - Real-time status
/ws
*Manual* *Autohandler*
This endpoint utilizes Websocket technology. It keeps the connection between the client and BPWin and pushes BPWin real-time status to the client. 
Events in BPWin monitored by this endpoint are listed as follows. 

## Monitored Events and Results(JSON format)
### Data Pattern Status
Applicable to Data Pattern Load operation only. There are only 2 kinds of "Status" at the moment: "Complete" and "Fail". "Progress" shows the percentage of progress. Right now only "0" corresponding to "Fail" and "100" corresponding to "Complete" will show up. 
```
{
    "Category" : "Event",
    "Note" : "DataPattern",
    "Operation": "Load",
    "Progress": "100",
    "Status": "Complete",
    "Timestamp": "2023-01-25 13:13:43 -0600"
}
```

### Job File Status
Applicable to Load and Save operations of BP and ABP files. The possible "Status" are "Complete" and "Fail". 
```
{
    "Category" : "Event",
    "Note" : "JobFile" ,
    "Directory": "C:\APITestFile\APITestAbp.abp",
    "Operation": "Load",
    "Status": "Complete",
    "Timestamp": "2023-01-25 13:15:46 -0600"
}
```

### Job Session Status
The following result indicates a job session begins. 
```
{
    "Category" : "Event",
    "Note" : "Job" ,
    "Status" : "Begin" ,
    "Timestamp" : "2023-01-26 09:48:10 -0600" 
}
```

A Successful completion of the job session will show Code 0.
```
{
    "Category" : "Event",
    "Note" : "Job" ,
    "Code" : "0" ,
    "Status" : "Complete" ,
    "Timestamp" : "2023-01-26 09:52:02 -0600" 
}
```

Soft Abort shows Code 1000. 
```
{
    "Category" : "Event",
    "Note" : "Job" ,
    "Code" : "1000" ,
    "Status" : "Complete" ,
    "Timestamp" : "2023-01-26 09:50:03 -0600" 
}
```

Hard Abort shows Code 1001.
```
{
    "Category" : "Event",
    "Note" : "Job" ,
    "Code" : "1001" ,
    "Status" : "Complete" ,
    "Timestamp" : "2023-01-26 09:55:33 -0600" 
}
```

### Socket Status
Whenever there is a change to the Socket Status, there will be a notification. There are 3 categories of socket status during job sessions that will show up in the notification: "Active", "Pass" and "Fail". 
Note that the Site and Socket numbers are 1-based. Socket 1 corresponds to Socket A in BPWin. 

The following result shows up for a "Socket Active" notification.
```
{
    "APS Serial Number": "H3911910180556",
    "Category": "Event",
    "DP Checksum": "F402D078h",
    "Note": "SocketStatus",
    "Site_SN": "43196",
    "Site_Socket": "3A",
    "Socket_PartNumber": "FVE4ASM08MLPA",
    "Socket_SN": "A002CAB5",
    "Status": "Active",
    "Timestamp": "2023-12-08 17:08:16 -0600"
}
```

The following result will show up for "Socket Pass" or "Socket Fail", with an extra field called "ResultCode". ResultCode 0 means Pass. A complete list of possible ResultCode can be found in Appendix B. 
```
{
    "APS Serial Number": "H3911910180556",
    "Category": "Event",
    "DP Checksum": "F402D078h",
    "Note": "SocketStatus",
    "ResultCode": "0",
    "Site_SN": "43196",
    "Site_Socket": "3A",
    "Socket_PartNumber": "FVE4ASM08MLPA",
    "Socket_SN": "A002CAB5",
    "Status": "Pass",
    "Timestamp": "2023-12-08 17:09:12 -0600"
}
```

Example in Python
The following code sample calls the websocket endpoint on BPWin and keeps the connection. While the connection is alive, whenever BPWin pushes a notification, the python script will print it out. 

```python
import asyncio
import websockets
import threading

async def websocket_handler():
    websocket_url = 'ws://[IP Address and Port]/ws'

    async with websockets.connect(websocket_url) as websocket:
        while True:
            message = await websocket.recv()
            print('Received:', message)
            
def run_websocket_handler():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    loop.run_until_complete(websocket_handler())
    loop.close()
            
if __name__ == "__main__":
    websocket_thread = threading.Thread(target=run_websocket_handler)
    websocket_thread.start()

    # Keep the main thread running to allow WebSocket thread to continue
    while True:
        pass
```


# Appendix A: The System Information JSON Introduction
In the Structure section below you will find an illustration of the structure of the JSON document retrieved by calling the /getsysteminfo endpoint.  In terms of the JSON string, each item in the illustration represents the name of a node, items in italics represent nodes that have children (that is, nodes that consist of child nodes), and items in normal text represent nodes without children (that is, nodes that consist of only data).  Child nodes are found underneath their parents at a greater indentation level (in other words, below and to the right of their parents).  

## Structure
BpWinVersion
OperatorMode
Programmer
   Communication
      PortMode
   Model
   SiteInfo
      MasterSite
      SocketModuleInfo
         Name
         SocketsPerSite
         DaughterCardName
      Site*
         Ordinal
         SerialNumber
         SocketModule
            Odometer
            TripOdometer
            Socket*
               Ordinal
               Odometer
               TripOdometer

* = Nodes that may appear 0 or more times in their location in the hierarchy.

## Reference
### //BpWinVersion

The version number of the BPWin application currently running the API.

**Example Data**

4.64.0

### //OperatorMode

Whether BPWin is currently running in operator mode. One of two possible strings: “Yes” or “No”.

**Example Data**

Yes

### //Programmer//Communication

A parent node consisting of child nodes that represent the currently detected PC to programmer communication configuration.

### //Programmer//Communication//PortMode

The current site communication protocol.

**Example Data**

(EPP)

### //Programmer

Consists of child nodes that contain information about the currently-detected programmer.  This node will be present only if the software is not currently in Demo mode.

### //Programmer//Model

The currently-detected programmer model.

**Example Data**

BP-2700

### //Programmer//SiteInfo

Consists of child nodes that contain information about the site hardware in the programmer.

### //Programmer//SiteInfo//MasterSite

An integer that designates which site on the programmer is currently configured as the “Master Site”.  This value adheres to the numbering scheme described in the Site and Socket Identification section.

**Example Data**

0

### //Programmer//SiteInfo//SocketModuleInfo

Consists of child nodes that contain general information about the currently-installed socket modules in the programmer.  
Because there can be only one type of socket module installed in the programmer at any given time, information exists that can be applied to all socket modules in a programmer at any given time.  This general socket module information is presented in this node.  For information that is specific to each socket module currently installed on the system, see //Programmer//SiteInfo//Site[x]//SocketModule.

### //Programmer//SiteInfo//SocketModuleInfo//Name

The name of the socket module currently installed in the programmer.

**Example Data**

FX4ASM48TD

### //Programmer//SiteInfo//SocketModuleInfo//SocketsPerSite

An integer that represents the number of sockets in each site.

**Example Data**

4

### //Programmer//SiteInfo//SocketModuleInfo//DaughterCardName

The name of the daughter card currently in the socket module.  This item will be present only if there is a daughter card in the socket module.

**Example Data**

WX4ASM48TD

### //Programmer//SiteInfo//Site[x]

Consists of child nodes that contain information about a site in the programmer.  Since programmers contain one or more sites, this node can be present one-or-more times.  Each occurance of this node represents a unique site in the programmer.
The x in the XPath string represents which node of these possibly-multiple nodes.

### //Programmer//SiteInfo//Site[x]//Ordinal

An integer in decimal format that designates which site on the programmer the //Programmer//SiteInfo//Site[x] node represents.  This value adheres to the numbering scheme described in the Site and Socket Identification section.

**Example Data**

1

### //Programmer//SiteInfo//Site[x]//SerialNumber

The site’s serial number as it would appear when running diagnostics in the BPWin application.

**Example Data**

34353

### //Programmer//SiteInfo//Site[x]//SocketModule

Consists of child nodes that contain information about the socket module installed in the site.  This node will not be present if a socket module is not installed in the site.

### //Programmer//SiteInfo//Site[x]//SocketModule//Odometer

An integer in decimal format representing the current value of the socket module’s insertion counter as it would appear when selecting “Tools/Socket Module Counter…” from the BPWin menu.

**Example Data**

26359

### //Programmer//SiteInfo//Site[x]//SocketModule//TripOdometer

An integer in decimal format representing the current value of the socket module’s “Number of Insertions Since Last Reset” as it would appear when selecting “Tools/Socket Module Counter…” from the BPWin menu.

**Example Data**

26346

### //Programmer//SiteInfo//Site[x]//SocketModule//Socket[y]

Consists of child nodes that contain information about a socket in the socket module.  Since socket modules can contain one or more sockets, this node can be present one-or-more times.  Each occurance of this node represents a unique socket in the socket module.
The y in the XPath string represents which node of these possibly-multiple nodes.

### //Programmer//SiteInfo//Site[x]//SocketModule//Socket[y]//Ordinal

An integer in decimal format that designates which socket in the socket module the //Programmer//SiteInfo//Site[x]//SocketModule//Socket[y] node represents.  This value adheres to the numbering scheme described in the Site and Socket Identification section.

### //Programmer//SiteInfo//Site[x]//SocketModule//Socket[y]//Odometer

An integer in decimal format representing the current value of the socket’s insertion counter as it would appear when selecting “Tools/Socket Module Counter…” from the BPWin menu.

**Example Data**

198211

### //Programmer//SiteInfo//Site[x]//SocketModule//Socket[y]//TripOdometer

An integer in decimal format representing the current value of the socket’s “Number Of Insertions Since Last Reset” as it would appear when selecting “Tools/Socket Module Counter…” from the BPWin menu.

**Example Data**

80381


# Appendix B: ResultCode List
The following table shows a list of explanations regarding the FailCode in BPWin. 
Right now this applies to the "FailCode" parameter in the JSON object returned by BPWin. 

| FailCode | Description |
|----------|----------|
| -1 | Internal BPWin error (firewalled unhandled exception).  Send contents of blackbox.log to BP Microsystems technical support. |
| 1 | Undefined |
| 2 | Unknown result code. |
| 3 | Cannot reset hardware. |
| 4 | Excessive current detected. The protection circuit has shut off the power. |
| 5 | Hardware timeout. |
| 8 | This function requires a programmer to be attached therefore the DEMO does not support it. |
| 9 | Programmer execution error. | 
| 10 | Error in programming algorithm. Please call technical support. |
| 14 | There is no chip in the socket. |
| 15 | The chip is not inserted in the socket correctly. |
| 16 | The chip is inserted backwards. | 
| 20 | Command failed. |
| 21 | Cannot program. |
| 22 | Cannot erase. |
| 23 | Invalid electronic signature in chip (device ID). |
| 24 | Invalid electronic signature in chip (algorithm ID). |
| 25 | Invalid electronic signature in chip (manufacturer ID). |
| 26 | Device is not blank. |
| 27 | Device is not secured. |
| 28 | Data in chip does not match buffer. | 
| 30 | Command aborted. |
| 36 | You must properly install the correct socket module or daughter-card. | 
| 37   | Illegal bit detected. |
| 38   | Functional test failed. |
| 39   | Device already secured. |
| 44   | Internal error. Please call technical support. |
| 45   | Hardware requires calibration. Please call technical support. |
| 48   | Cannot Unprotect. |
| 49   | Cannot Protect. |
| 50   | Device's sum does not match sum specified in AFS/Options. |
| 54   | Device is not blank. (Interchangeable with failure code 26) |
| 55   | Invalid single byte identifier in chip. |
| 56   | Some device pins are shorted together. |
| 58   | The device outputs were not tristated. |
| 59   | Excessive leakage current detected. |
| 80   | Excessive current detected. The protection circuit has shut off the power. (Interchangeable with failure code 4). |
| 92   | There is no status available for the specified site. |
| 134  | Device is not encrypted. |
| 135  | Failed to send entire data pattern to device. |
| 137  | This device does not support empty socket test. |
| 145  | Devices should not be present in the slave sockets. |
| 146  | Cannot read device. |
| 149  | A device was detected in a socket. |
| 150  | Could not load BP file. |
| 151  | Software feature not licensed on all sites. |
| 152  | Software feature not licensed on some sites, registration expired on other sites. |
| 154  | The quantity number should be between 1 and 65535. |




