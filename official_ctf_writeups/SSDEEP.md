![sssec-logo-trans](https://user-images.githubusercontent.com/119784145/207296858-e539a085-44bc-4ff1-82e4-70c86a42d8a9.png)

# SSSEC SSDEEP OFFICIAL WRITEUP

![SSDEEP,LOGO](https://user-images.githubusercontent.com/119784145/207301869-2a0d3215-09c8-4cf4-8e19-bc7e0a6bb836.jpg)

   <sub>by c1nn3r:vampire:</sub>
   
## Recon

Starting this off by the usual routine, first export the ip to make things easier for later

<sub>im working on this mounted as a virtual machine!</sub>
```bash
export ip=192.168.56.104
```
![ping and export](https://user-images.githubusercontent.com/119784145/207312306-e743f577-39e6-44dd-8b63-7271b9f1839a.png)

Performing an initial nmap scan,
```bash
nmap -sV -sC $ip
```

![nmap_first](https://user-images.githubusercontent.com/119784145/207301936-3bc853f6-3f61-402e-a400-f7871176563c.png)

- seems like port 80 is open
 
Scanning all possible ports
```bash
nmap -sV -sC $ip -p-
```
![nmap_allports](https://user-images.githubusercontent.com/119784145/207302816-17fcb04d-e536-4aa0-8687-17f8e8824262.png)
- all ports scan returned three more ports with running a service 'tcpwrapped'

Checking out port 80

![checking_80](https://user-images.githubusercontent.com/119784145/207303923-c0b9e1b4-30f8-47aa-a513-6640081ebbec.png)
- nothing interesting here

Checking the source code of index page
![80_source](https://user-images.githubusercontent.com/119784145/207304184-9ea551e3-7152-43bb-89a1-3ef8c86cf45e.png)
- interesting file name 'rockme.jpg', noted!📓

## Enumeration
Since port 80 is the only 'functional' looking port on the machine, enumerating to get more information
### Directory bruteforcing
Bruteforcing directories with dirb its default wordlist
```bash
dirb http://$ip
```
![dirbusting_](https://user-images.githubusercontent.com/119784145/207305415-a7797d50-8d70-47f9-a551-2ad1f4f1e1e2.png)

- found directory backups
- found pages, index.html, style.css, robots.txt

Checking out backups/

![backupsdir](https://user-images.githubusercontent.com/119784145/207306650-780a2ebf-afe4-4526-9987-954e383f1ec5.png)

- interesting files, note.txt and ssbackup.zip

contents of note.txt

```


To James
Hey, man! i have created a backup copy of the source code of our program on the server, just in case that idiot notc1nn3r messes it up again :)
                                                     ~ sssecadmin1


```
- okay, some sort of program running on the server, maybe those three 'tcpwrapped' ports are some sort of netcat services?

Making a directory to work in
 ```bash
 mkdir loot
 cd loot
 ```
![making loot_folder](https://user-images.githubusercontent.com/119784145/207308018-f6e53399-b08f-4c79-996b-8791984c89f2.png)
<sub> ls the new directory lol:clown:</sub>

Downloading ssbackup.zip

```bash
wget http://$ip/backups/ssbackup.zip
```
![downloading_backup](https://user-images.githubusercontent.com/119784145/207307663-a50b18e9-3c34-47af-9871-7324b57529c5.png)

Checking out downloaded file
```bash
file ssbackup.zip
```
![checking_backup_zip](https://user-images.githubusercontent.com/119784145/207308545-39b2c8a6-af89-4dfa-8288-13e252a0c4c8.png)
- seems like its encrypted, need the password for the files

Gotta check that 'rockme.jpg' file too...
```bash
wget http://$ip/rockme.jpg
binwalk rockme.jpg
```
![downloaded_binwaled_image](https://user-images.githubusercontent.com/119784145/207309283-4811e175-2c53-4258-ab60-c5e1d91037bb.png)
- nothing embedded in the image... 🤔

### Enummeration and recon summary
- The server has one primary port open, 80 and its hosting a web server 
- The webserver is just a picture of a submarine with a peculiar name 'rockme.jpg'
 - maybe its hinting to the rockyou wordlist?
- There is a backups/ directory available on the server, with two files 'note.txt' and 'ssbackup.zip'
 - 'ssbackup.zip' is a password protected zip archive with the source code for programs running on the server?
 - 'note.txt' has got some juicy information, three possible usernames

<sub>moving on.. ⏭ </sub>

Digging deeper and trying to find out things in these files and server...

## Steganography

So one obvious thing that comes in mind when encountering images is hidden files within them..
and since the image is named 'rockme', its must be checked.

Running stegseek to try and crack this thing, ps. stegseek runs with rockyou in default
```bash
stegseek rockme.jpg
```
![stegsee_found_it](https://user-images.githubusercontent.com/119784145/207312509-fe2e114d-237d-42bf-b9b7-49e95a4afdf8.png)
- Stegseek found the password and extracted the files!

checking and extracting the embedded file
![extracting_stegged](https://user-images.githubusercontent.com/119784145/207313885-28738bd7-a792-4847-9e3b-51637f3c685b.png)
- seems like a zip file, with a 'sssec.dic' file inside of it

![head_dictionary](https://user-images.githubusercontent.com/119784145/207314953-4ee5ee74-6279-493a-ab9a-2c34994e020f.png)
- sssec.dic is a dictionary ... maybe with the password for the backup zip 

## Bruteforcing zip

With the dictionary now i can try and crack the password for the ssbackup.zip file 

proceeding with john the ripper, converting it into a file john could work on and specifying the sssec.dic as wordlist

```bash
zip2john ssbackup.zip > 2j.txt
john 2j.txt --wordlist=sssec.dic
```
![cracking_zip](https://user-images.githubusercontent.com/119784145/207315210-595ed6d9-1f41-4547-845d-56986d4523ce.png)
- and sure enough, john found  the password for the zip!!

Extracting the files!
![extracted_zip](https://user-images.githubusercontent.com/119784145/207316047-3b85a8dc-79ed-4ae0-b211-6e7030cf8a8d.png)
<sub>engrampa is 1337</sub>

## Reverse Engineering

Now that i have the source code files for whatever program is running on this server, i can take a look at them and get more details about those ports?

Checking 'sssec.py'
![sssecpyiscorrupt](https://user-images.githubusercontent.com/119784145/207317180-8730585c-b7de-4ab2-8083-10364a4367f4.png)
- seems like its corrupted or something...

Checking 'ssdeep'
![ssdeepsourcecode](https://user-images.githubusercontent.com/119784145/207317521-5ad18f5f-dce2-499f-bba7-b6d4188000b1.png)
- a long string variable assigned as 'intvar'...also looks like hex 

![ssdeepsourcecode2](https://user-images.githubusercontent.com/119784145/207317753-d1700f98-3007-456c-bd45-1cf4dd119941.png)
- Seems like some kind of python portal program...

## Deobfuscation 

Since intvar seems like hex, gotta try and decode/deobfuscate whatever its representing

![ssdeepsourcecode_copytheintvar](https://user-images.githubusercontent.com/119784145/207317979-27e8e600-ce5f-445a-8a61-aa2d03a59559.png)
- copying the string

Using CyberChef <sub> because i want to keep my sanity </sub>

![cyberchef 1](https://user-images.githubusercontent.com/119784145/207318358-cd1f0f55-6d6b-4974-a8c8-449eb318fa34.jpg)
- From hex,  decodes into what seems like base64 , but with the equal signs at the top...this might be reversed

![cyberchef 2](https://user-images.githubusercontent.com/119784145/207318749-04a30b2e-81ce-4edf-8a03-6facccfed425.jpg)
- Reversed, and From base64 outputs binary 

![cyberchef 3](https://user-images.githubusercontent.com/119784145/207319171-23276965-183b-4cf6-803d-4d54e1665b20.jpg)
- And again, From binary then from hex gives us the first flag and some text!

Replacing the giant hex with the deobfuscated version
![intvar_deofuscated](https://user-images.githubusercontent.com/119784145/207320044-08ffb70c-ee2e-4b32-9f72-7cb160309f60.png)

## Bruteforcing ports

Okay so now that ive have got the first flag <sub>and a proper music taste</sub> lets proceed poking around those ports 

Started with another nmap scan
```bash
nmap -sV -sC $ip -p- 
```
![nmap changes ports](https://user-images.githubusercontent.com/119784145/207320219-680f46b6-6fed-46b6-949f-2d2914c13fd1.png)
- okay, now its seems like the others are gone and we have only this one port.. a bug maybe?

Trying to netcat into it
```bash
nc $ip 20383 -vv
```
and it refuses the connection as if that port wasnt open, doing the nmap scan again shows a new port open...

THIS IS WEIRD

# cringe Enlightment moment <sub> + and a networking lesson lol</sub>

But wait, what if the target is running a loop with netcat to the ssdeep program, If thats the case the port changes every connection, even when nmap scans it, its connecting to it and grabbing the banner and other signatures...


## Rabbit hole

## More Reverse enginnering

## Euerica moment
