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

## Quick Starter with DataPanel.py
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
- Open a cmd line prompt, type the command: ```streamlit run C:\Devkit.py```
- The command window will show the url ```http://localhost:8501```. Normally your default browser will automatically open to show the real-time data panel. If that doesn’t happen, you can copy the url and paste it to your browser to view the panel.

## Authentication Process
This section talks about the authentication process for using the BPWin REST API endpoints. An authentication process prevents any unauthorized access to the BPWin running on your APS machines. 
### Setting Up API Passphrases
To begin using the BPWin API, you need to set up passphrases for each APS machine you intend to connect to. Follow these steps:
Locate the BP Folder: First, navigate to the BP folder on your computer. 
To know where the BP folder is located, open the Registry editor. Go to “Computer\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\BP Microsystems\BPWin\Directory Paths” and look for “BP” in it. Normally the folder should be “C:\BP”. 
Create a Passphrase File: For each APS machine, create a text file with your API passphrase in the BP folder. The name of the text file should be “Auth.txt”. You can choose to use the same passphrase for multiple APS machines or set different ones for each. Ensure that the passphrase is secure and not easily guessable. File Requirements:
If the file is not present in the BP folder, the BPWin REST API feature will be disabled on that APS machine.
If the file is present but empty, it will be considered as opening the BPWin REST API on that machine to HTTP calls from anywhere.
After you update the passphrase, you need to restart BPWin for it to take effect. 

### Initiating the Connection
Open BPWin as usual. Wait for it to finish initialization. Before making any other API calls, you need to initiate the connection by calling the /auth endpoint. Follow these steps:

Make the First API Call: Use your preferred programming language or API client to make a GET request to the /auth endpoint of your BPWin. 
Code Example in Python

```python
import requests
response = requests.get("http://192.168.0.121:8080/auth")
print(response.text)
```

Result format

```{ "Number" : 7241978785468376164 }```

Random Challenge Number: BPWin will generate a random challenge number and pass it back to your client application as a response. This challenge number changes with every opening of the BPWin. That means, if closing and re-opening BPWin, you need to call /auth again to get a new Random Challenge number. 

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