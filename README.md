# Kimsuky PowerShell Backdoor

### Kimsuky PowerShell Backdoor Server

![kimsuky-ps-backdoor](https://github.com/knight0x07/Kimsuky-PS-Backdoor/assets/60843949/686d4088-ef8e-4f22-b4a9-43aa1cd3cf7d)

credits: https://twitter.com/asdasd13asbz/status/1763068671428383152

### Analyzed the Backdoor Client - below is the commented **enum** explaining the Backdoor commands - 

**Commands:**
```
public enum _OP_CODE : ushort
{
	OP_UNIQ_ID	=	0x401, #Check-In Unique ID - Sent with first packet from Client
	OP_REQ_DRIVE_LIST    =	0x402, # Request from Server for logical drive info
	OP_RES_DRIVE_LIST    =	0x403, # Response from client with logical drive info
	OP_REQ_PATH_LIST     =	0x404, # Request from Server for list of dir & files from path
	OP_RES_PATH_LIST     =	0x405, # Response from client with list of dir, files from path
	OP_REQ_PATH_DOWNLOAD =	0x406, # Request from server to exfiltrate file/dir to the C2 - arg: file/dir_path;c2_url
	OP_RES_PATH_DOWNLOAD =	0x407, # Response from client once the file/dir (ZIP + b64 encoded) is exfiltrated to C2
	OP_REQ_PATH_DELETE   =	0x408, # Request from server to delete dir/file - arg:path
	OP_RES_PATH_DELETE   =	0x409, # Response from client after deleting dir/file
	OP_REQ_FILE_UPLOAD   =	0x40A, # Request from server to upload file on the machine
	OP_RES_FILE_UPLOAD   =	0x40B, # Response from client once the uploaded file is written on the machine
	OP_REQ_PATH_RENAME   =	0x40C, # Request from server to rename file/folder - arg:oldfilename,newfilename
	OP_RES_PATH_RENAME   =	0x40D, # Response from client after renaming file/folder
	OP_REQ_CREATE_DIR    =  0x40E, # Request from server to create directory - arg: path - add (2) if already created
	OP_RES_CREATE_DIR    =  0x40F, # Response from server after creating directory
	OP_REQ_RESTART       =  0x410, # Restart connection
	OP_REQ_CLOSE         =  0x411, # Close connection
	OP_REQ_REMOVE        =  0x412, # Close connection
	OP_RES_DRIVE_ERROR	= 0x413, # Sent from client: no drives found / no permissions / io error
	OP_REQ_EXECUTE	= 0x414, # Request from Server to execute file/command - arg:path
	OP_RES_EXECUTE	= 0x415, # Response from client after executing the file/command via IEX - uses OP_REQ_EXECUTE
	OP_REQ_CREATE_ZIP	= 0x416, # Request from server to ZIP archive files/directory arg: path
	OP_RES_CREATE_ZIP	= 0x417  # Response from server after ZIP archiving the files/directory - uses OP_REQ_CREATE_ZIP
}
```
### Notes

- Unique ID Gen - md5(ip + mac)
- Send/Recv RC4 Key - rc4(unique_id + ('_r' | '_s')

**Commands:**

**OP_REQ_DRIVE_LIST:**

Sends across following logical drive information to the server - 
- drive.Name
- drive.VolumeLabel
- drive.DriveType
- drive.DriveFormat

**OP_REQ_PATH_LIST:**

Sends across following file & directory information to the server - 
- dir.Name
- dir.LastWriteTime
- file.Name
- file.Length
- file.LastWriteTime

**OP_REQ_PATH_DOWNLOAD:**

Exfiltrate the files/directory to the C2 server via a POST Request.

- If the provided target path is a directory, the directory is ZIP archived on the endpoint, and if it is a file path it is used as it is.
- Request parameters to C2:
	- C2 URL: C2_URL/show.php | C2_URL provided from the Server
	- User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36 Edg/87.0.664.75
	- POST Request Body:
 			
   			- filename = ToBase64String(filename) | filename: file to be exfiltrated
			- Data: ToBase64String(file_contents) ; File contents of file to be exfiltrated
- POST Request with the exfiltrated file/directory sent to C2 server via Invoke-WebRequest
