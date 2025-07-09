DIVA Application. apk Android Testing:

Installation:

Applications I will be using to carry out the vulnerability testing.
1.	Android Studio
2.	SDK Platform Tools from Android studio
3.	DIVA apk
4.	Jadx-gui
5.	Apktool
6.	Java runtime environment
How to install diva apk on the android device:
Open powershell/cmd and run the following command
I used Android Debug Bridge (adb) to verify my emulator is running.
Android Debug Bridge (adb) is a versatile command-line tool that lets you communicate with a device. The adb command facilitates a variety of device actions, such as installing and debugging apps, and it provides access to a Unix shell that you can use to run a variety of commands on a device.
Step-by-Step: Load DIVA into Emulator:
Downloaded emulator which is suitable to test DivaApplication.
![Screenshot](Screenshot%202025-07-07%20182250.png)

Downlaod DivaApplication.apk from github
![Screenshot](Screenshot%202025-07-07%20182525.png)
 
Step1 - Navigate to the folder where DivaApplication.apk is located
cd "C:\Users\meeni\Downloads\diva-apk-file-main\diva-apk-file-main"
Step2 – Install the APK into emulator
Run: adb install DivaApplication.apk
Output shows success.
Step3 – Launch DIVA in emulator
From cmd -  adb shell monkey -p jakhar.aseem.diva -c android.intent.category.LAUNCHER 
After successfully installing it, we can see it in our android device.
![Insecure Logging ](Screenshot%insecure-logging-2.png)
![Insecure Logging](Screenshot%202025-07-05%20164921.png)
 
**1. INSECURE LOGGING**
![Screenshot](Screenshot%202025-07-05%20165604.png)

![Screenshot](Screenshot%202025-07-05%20165158.png)
 
 
First, I used Jadx-gui to read the source code to find the vulnerable code causing the insecure logging.

From the image above, we can see that the insecure logging is happening in the public void section. When the user inputs their credentials, they are stored in diva-log and converted to strings, making them visible to plain eyes and anyone who gets access to diva-log.
 
![Screenshot](Screenshot%202025-07-07%20114320.png)
 
Conclusion: It shows that the logging of credentials or sensitive information is not secured, making it available to anyone going through the logs of the device.

**2. HARCODING ISSUES — PART 1**

![Screenshot](Screenshot%202025-07-07%20115434.png) 
First, you use Jadx-gui to find the location of the vulnerable code in the source code of the application.
 
From the image above, when a user inserts the vendor key, the user will get a response depending on the outcome of the vendor key. If the vendor is equal to vendorsecretkey, it will display Access granted! See you on the other side; otherwise, it displays Access Denied! See you in hell!
From this explanation, we can see that the developer made it easy for anyone who has access to the source code by leaving the correct vendor key in the source code of the application.
![Screenshot](Screenshot%202025-07-07%20121249.png) 
This is the feedback the user get when successfully logged in with the right vendor key.
This is the feedback the user get when successfully logged in with the wrong vendor key.
![Screenshot](Screenshot%202025-07-07%20121429.png)


**3. INSECURE DATA STORAGE — PART 1**
 
Using Jadx-gui to read the source code of the application, we can see the directory in which the user credentials are stored, and with our terminal, we can get root access and find the location of the directory on the Android device.
spedit.putString("user", usr.getText().toString()); - This is where the username of the user will be located
spedit.putString("password", pwd.getText().toString()); - This is where the user password can be found
SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this); - the directory of the stored credentials
![Screenshot](Screenshot%202025-07-07%20121828.png)

The image below shows the details provided by the user and the response received after clicking the save button.
![Screenshot](Screenshot%202025-07-07%20123432.png)
 
Run adb shell to gain root access to the Android device and ls to list all the files in the directory.
Next, run cd data/data/ to enter the directory and ls to list all the files in the directory.
![Screenshot](Screenshot%202025-07-07%20124615.png) 
Run cd jakhar.aseem.diva to enter the directory and ls to list the files in the directory.
![Screenshot](Screenshot%202025-07-07%20124822.png)
 
From the image above, we found the shared preferences directory
To access the directory, run cd shared_prefs and ls to list the items in the directory.
![Screenshot](Screenshot%202025-07-07%20125122.png) 
Now, run cat jakhar.aseem.diva_preferences.xml to read the xml file.
![Screenshot](Screenshot%202025-07-07%20125347.png) 
Anusha – name inserted by the user
123456 – password given by the user
Conclusion: Storing credentials in plain text is referred to as an insecure way of storing data.

**4. INSECURE DATA STORAGE — PART 2**

First, we are going to use Jadx-gui to analyze the source code to find the vulnerability of the application.
![Screenshot](Screenshot%202025-07-07%20151600.png) 
From the image above, we can see the location of the stored credentials and the way they are being stored, which is in a SQL database format. Now, we are going to gain root access on the Android device and locate the database where ids2 is stored to see if it is a directory or a readable file.

Before gaining root access on the device, we are going to insert user credentials to get something saved in the databases and then locate it.

![Screenshot](Screenshot%202025-07-07%20153839.png) 
First, gain root access to the device using the adb shell. Then, run cd data/data/jakhar.aseem.diva/databases/ to the directory and ls to list the items in the directory.
![Screenshot](Screenshot%202025-07-07%20155330.png)

Run ls -la to list all the files and items in the directory.
![Screenshot](Screenshot%202025-07-07%20155516.png)
 
To read the sql format in ids2, exit the root access to the android device and pull the file to your system.
Run adb pull /data/data/jakhar.aseem.diva/databases/ids2
![Screenshot](screenshot%202025-07-07%20175348.png)
![Screenshot](Screenshot%202025-07-07%20175023.png)  
Open the database with sqlite CLI using -  sqlite3 ids2.db
Check available tables -  .tables
See table structure - .schema myuser
View stored data - SELECT * FROM myuser;
Output with user entered credentials - Followup|abcdef1234
That’s the insecurely stored username and password from the DIVA challenge. 
**OR**
To read the sql format in ids2, exit the root access to the android device and pull the file to your system.
Run adb pull /data/data/jakhar.aseem.diva/databases/ids2
![Screenshot](Screenshot%202025-07-08%20151844.png)
 
And use sqlite viewer to read the ids2 file, click here to go the website.
![Screenshot](Screenshot%202025-07-08%20151957.png)
 
Drag and drop the ids2 file, and change the android_metadata to myuser to view the credentials.
![Screenshot](Screenshot%202025-07-08%20152102.png)
![Screenshot](Screenshot%202025-07-08%20152148.png)
![Screenshot](Screenshot%202025-07-08%20151651.png)
 
 
 
**5. INSECURE DATA STORAGE — PART 3**

![Screenshot](Screenshot%202025-07-08%20153222.png)
 
Using Jadx-gui read the source code to find the vulnerability.
We found out that the application created a temp file named **uinfo**.
![Screenshot](Screenshot%202025-07-08%20153411.png)
 
 
To access the stored credential, I used adb shell to gain root access of the android device.
Run cd data/data/jakhar.aseem.diva/ to enter the application directory and ls -la to list all the files and directories under the specified directory.
Next, use cat uinfo931559435tmp to print the file content to a standard output.
 
Conclusion: From the image above, we can see the user credentials present in the location we got when using Jadx-gui.

**6. INSECURE DATA STORAGE — PART 4**

![Screenshot](Screenshot%202025-07-08%20161027.png)

 
Using Jadx-gui to read the source code of the application, I found out the location where the user credentials are being stored and the name of the file, which is on the external storage device on the Android phone.
 
If we try to input any credential, we will get an error because the application has not been given the password to anything on the SD card.
![Screenshot](Screenshot%202025-07-08%20161027.png)
![Screenshot](Screenshot%202025-07-08%20161706.png)
  
Change the storage permission of the application.
![Screenshot](Screenshot%202025-07-08%20161851.png)
![Screenshot](Screenshot%202025-07-08%20162045.png)
  
To begin, run adb shell to gain root access of the device and ls to list all the items in the directory
![Screenshot](Screenshot%202025-07-08%20162433.png)
 
Run cd mnt to access the external storage on the Android device and ls to list the items in the directory.
![Screenshot](Screenshot%202025-07-08%20162620.png)
 
Next, run cd sdcard to enter the directory and ls -la to list all the items in the directory.
![Screenshot](Screenshot%202025-07-08%20162734.png)

Run cat uinfo.txt to print out the content of the file to a standard output.
![Screenshot](Screenshot%202025-07-08%20162947.png)
 
Conclusion: We can see that if we use ls to list the items in the directory, we won’t see uinfo.txt. Therefore, it is advised to try both commands if we suspect the file is in that directory.

**7. INPUT VALIDATION ISSUES — PART 1**

![Screenshot](Screenshot%202025-07-08%20172031.png)
 
Using Jadx-gui to read the source code of the application, we will notice that this input validation is vulnerable to SQL injection. Also, we can see all three user credentials present in the database just by reading the source without digging deep, which is not a safe way to store credentials.
![Screenshot](Screenshot%202025-07-08%20172349.png)
 
Tried giving a random user credential and see what response we get.
![Screenshot](Screenshot%202025-07-08%20172506.png)
 
Now, let’s use SQL injection to bypass the search validation and get all three credentials in the database.
![Screenshot](Screenshot%202025-07-08%20172901.png)
 

**8. INPUT VALIDATION ISSUES — PART 2**

![Screenshot](Screenshot%202025-07-08%20173204.png)
 
This challenge is about accessing sensitive information using the file path of the sensitive information. Tried the file path from the previous challenge (vuln 6).
![Screenshot](Screenshot%202025-07-08%20174029.png)

## 🧾 Key Findings

During static and dynamic testing of the DIVA-Application.apk, several common Android vulnerabilities were identified and successfully exploited for learning purposes:

1. **Insecure Logging**
   - Sensitive credentials (username/password) are logged in plain text in logcat.

2. **Hardcoded Secrets**
   - Vendor keys and logic used for access control are hardcoded in the app, making it easy to bypass login mechanisms.

3. **Insecure Data Storage**
   - User credentials are stored in `SharedPreferences` without encryption.
   - Internal storage and data directories are accessible on rooted/emulated devices.

4. **Input Validation Issues**
   - Lack of input sanitization enables injection-based attacks.
   - Potential for bypassing client-side validation.
     
   ##  This is only part1 and will continue part2 on access control Flaws.

# ⚠️ Disclaimer

This project is intended strictly for **educational and ethical penetration testing purposes** only. The DIVA (Damn Insecure and Vulnerable App) is deliberately built with known vulnerabilities to help security professionals, developers, and learners improve their understanding of mobile application security.

- Do **not** use this application or any related testing methods against unauthorized systems or applications.
- The author and contributors of this repository are **not responsible** for any misuse or illegal activities performed using the knowledge gained from this project.

Always follow responsible disclosure and ethical hacking practices.   
