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

## Bruteforcing zip

## Reverse Enginering

## Deobfuscation 

SSSEC{I7S_R1GH7_H3R3_09A8USYBNI87YHK} 9012312412, oh btw you should start listening to deftones

## Bruteforcing ports

## Rabbit hole

## More Reverse enginnering

## Euerica moment
