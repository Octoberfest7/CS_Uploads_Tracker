# CS_Uploads_Tracker
This is an Aggressor script for use within CobaltStrike to aid in the tracking of files that have been uploaded to target systems via the CobaltStrike upload command.

![image](https://user-images.githubusercontent.com/91164728/200203816-3dd2c913-0b2d-4bf4-bf6f-ec5ab1c0a7c3.png)

During complex and extended operations artifacts may be dropped to disk in several places across quite a few different machines.  This script aims to assist the Operator in tracking uploaded files to ensure proper clean-up upon conclusion of an operation.

## Features

I re-defined the CobaltStrike upload command so that each time a file is uploaded it is also logged in an easy to read format.  The major difference between the two is that the custom defined upload command also calls the 'pwd' command in order to fetch the directory that the file was just uploaded to.  This is necessary for logging purposes.

The new upload command works just like the original one, you can either enter "upload" and then select a file to upload from file browser, or enter "upload my/file/path.exe" and upload without the file browser.  The script will ensure that the file you are trying to upload exists and return an error if it doesn't.

Each time a file is uploaded a new log entry is made.  The script calls the host OS to run md5sum on the uploaded file in order to retrieve and log the uploaded files MD5 hash for deconfliction/tracking. Additionally the entire upload table is written out to a file on the attacker system. 

The upload table tracks the following fields:

1. Upload Date/Time
2. Target Internal IP
3. Upload Directory
4. Remote File Name
5. Uploaded File
6. MD5 Hash

**NOTE: This table does not track changes to file name/location after upload! Information is only accurate at time of upload!**

When the upload.cna script is first loaded, if a file already exists containing upload log entries it is loaded by the script to populate the upload table.  This ensures that should the CobaltStrike client crash or be re-loaded, the records pertaining to uploaded files are preserved and available for the Operator.

The "viewuploads" command displays the table containing uploaded files.  This table tracks all files uploaded across all different beacons in the current CobaltStrike Client.

The "viewuploads" command also enables an Operator to manually create entries in the uploads table.

![image](https://user-images.githubusercontent.com/91164728/200205010-0bbbad22-25d4-4205-a87f-cd7f65cf5d2b.png)

This may be useful when performing lateral movement and moving a file to another target system in the network WITHOUT making use of the upload command.  The operator may run something like 

```viewuploads add 192.168.1.75 c:\windows\system32\mymalware.exe /home/kali/mypayload.exe```

This will add a new entry to the table where:

'192.168.1.75' is the ip of the machine that the file was moved to  
'c:\windows\system32\mymalware.exe' is the location of the file on the machine  
'/home/kali/mypayload.exe' is the original file on the attacker's machine  

It is assumed that the file being moved to the new machine in the network is a payload/file that originated on the attacker's machine before being uploaded to an already-compromised machine, from where it will be moved to the '192.168.1.75' machine as part of lateral movement.  The local file path is required so that the MD5 hash of the file can be retrieved and logged.

The "viewuploads" command can also be used to delete log entires.  This can be useful should a typo be made when entering a log manually, but also when performing clean-up.  Operators may remove upload table entries as they clean their way out of the network to keep an easily-accessible list of what artifacts still reside on disk in the target network. 

![image](https://user-images.githubusercontent.com/91164728/200205078-3c7e5f7f-b627-4671-aa62-c320d7d2d951.png)

## Setup

Simply load upload.cna into CobaltStrike script manager.  On initial load upload.cna will attempt to create uploadedfiles.txt in the CobaltStrike directory.  If you receive an error it could be that the use who is running CobaltStrike doesn't have write permissions in the directory.  

Once uploadedfiles.txt has been created, all subsequent loads of upload.cna will read in uploadedfiles.txt and populate the uploads table with whatever data is in the file.  This ensures that data is not lost when the Client crashes or restarts.

To fully clear the uploads table, simply delete uploadedfiles.txt.  A fresh, blank copy will be made the next time upload.cna is loaded.
