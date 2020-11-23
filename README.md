# Firemware Hacking
These are notes from a SANS Tech Tuesday lab with Larry Pesce.

## Firmware Samples
Included in the repository here.
1. camera1/2
2. radiosonde - Intel hex format which needs to be converted to bin before analysis
3. router-firmware.zip - encrypted zip file which needs to be cracked to get to the firmware
4. SUV-QNX.rar - SUV entertainment system firmware

## Cracking the Zip file
* fcrackzip
* John
* Hashcat
### Hashcat
```bash
# get hashes in hashcat format I COULD NOT GET THIS TO CRACK ON HASHCAT CORRECTLY
# Did some googling and found a forum which says hashcat does not support pkzip
# type cracking, and recommended using john. So that may be the issue. Didnt observe the date.
zip2john router-firmware.zip | cut -d ':' -f 2 > zip_hashes.txt

# spoke of using the correct mask to crack this
hashcat64.exe -m <pkzip encrypted> zip_hashes.txt "a$a$a$a$a$"
```bash
### John
zip2john router-firmware.zip > john_zip_hashes.txt
john --fork=4 --rules --wordlist /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt
```

## Binwalk
Can unzip firmware into navigatable directories. **Note** - Binwalk that comes installed with Kali has issues. It should be manaully installed into a virtual environment as per their github instructions to get all the features working correctly.

```bash
# get file systems and partitions
binwalk camera-firemware.bin

# extract file systems to disk
binwalk camera-firmware.bin --extract

# entropy sacnner looking for secrets / encryption (versus zip). Encryption will be uniform randomness while zip will have peaks and valleys.
binwalk camera-firmware.bin -E

# on kali, binwalk hung unrar'ing, so did it manaully
binwalk SUV-QNX.rar --extract
# or
mkdir _extracted-SUV-QNX && cp SUV-QNX.rar/_extracted-SUV-QNX && cd _extracted-SUV-QNX && unrar SUV-QNZ.rar
```
## Sifting Through
Often times, embedded systems will not use shadow files and have the hashes right in the password file. Or, the shadow file will be readable once extracted.
```bash
unshadow /path/to/passwd /path/to/shadow > unshadowed
hashcat64.exe -m500 -a 0 -o cracked.txt unshadowed /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt

# sift through files using strings
# jffs2 is read/write
# squashfs is read-only
strings -8 550040.squashfs 
strings -8 5F0040.jffs2 
strings -8 5F0040.jffs2  | less

# loog for web servers and content being hosted such as cgis
```
## Intel Hex Format
Decode with:
```bash
objcopy -I ihex -O binary radiosonde.hex radiosonde.bin
binwalk radiosonde.bin
```

