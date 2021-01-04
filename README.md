# Dig-it
Digi Router Password Decoder

![alt text](https://github.com/billchaison/Dig-it/blob/main/digi00.png)

The Digi wireless router (WR21) stores passwords as base64 using XOR obfuscation.  An example password looks like this:

`user 1 epassword "KD5lSVJDVVg="`

The stored password can be seen in the web interface by executing the `CONFIG C SHOW` command after navigating to "Administration >> Execute a command".

![alt text](https://github.com/billchaison/Dig-it/blob/main/digi01.png)

The following bash script `digi_pw_decoder.sh` can be used to decode the password by supplying the Base64 string as an argument.

```bash
#!/usr/bin/bash

if [ $# -ne 1 ]
then
   echo "Supply the Base64 encoded password string as a parameter."
   exit
fi

encpwd=( $(echo $1 | base64 -d | xxd -p -c 40 | head -n 1 | fold -w 2) )

deckey=("58" "5f" "16" "3a" "25" "2c" "27" "3c" "32" "3d" "21" "7d" "36" "7c" "75" "64" "6c" "6c" "6c" "75" "37" "2c" "75" "06" "34" "27" "3c" "34" "3b" "75" "06" "2c" "26" "21" "30" "38" "26" "75" "78" "75")

decpwd=""

for i in "${!encpwd[@]}"
do
   decchr=$(printf "%02x" $(("0x${encpwd[$i]}"^"0x${deckey[$i]}")))
   decpwd="$decpwd$decchr"
done
echo $decpwd | xxd -r -p | hexdump -Cv
```

Here is an example output:<br />
![alt text](https://github.com/billchaison/Dig-it/blob/main/digi02.png)
