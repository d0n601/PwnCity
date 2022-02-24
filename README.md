# PwnCity
An insecurely configured TeamCity continuous integration environment. 

### Attack Infrastructure
* Kali Linux:      VM on operator machine.
* Ubuntu 20.04LTS: Empire Server

### PwnCity Lab
* Ubuntu 20.04  TeamCity
* Windows 10:   BuildAgent01
* Windows 7:    BuildAgent02
* Windows 10:   Bruno-PC 

#### Creds
Credentials chosen from `rockyou.txt`.  
* bruno:AMOTEbruno84  (Windows)
* dev:Roblerino1995 (Windows/Linux)
* admin:aut0magic (TeamCity)


## Demo
There may be more than one path through PwnCity, but this is the one I'll be presenting on Feb 24th at the [OWASP Sacramento Chapter](https://owasp.org/www-chapter-sacramento/) meeting.

### Initial Access
1. Scan the IP to discover SSH and TeamCity `nmap -Pn -p- 52.234.0.18`.
   * Note: It won't respond to ICMP, so `nmap 52.234.0.18` makes it seem dead.
   * Note: If you only scan the top 1000 TCP ports, you miss TeamCity `nmap -Pn 52.234.0.18`.
   ![portscan](https://user-images.githubusercontent.com/8961705/155561063-0a8a142d-4675-4b3e-a811-acc53b561339.png)  
2. Browse to the TeamCity URL [http://52.234.0.18:8111](http://52.234.0.18:8111).
3. Create a new user `bob`.
4. As `bob` navigate to `Projects > SimpleMavenSample > Build > Settings` 
  * See that `Parameters` contains credentials. 
5. See if credentials are reused for `ssh dev@168.62.29.0`, and ssh in as low privileged user. 

### From the Foothold
We could tunnel from our initial foothold. Knowing that RDP is open on two build agents would allow us to attempt to authenticate via the creds we've found...but that's not as fun.
1. Explore the TeamCity server a bit and check out the upser user token `cat /home/dev/TeamCity/Teamcity/logs/teamcity-server.log | grep "Super user"
2. Now login as the Super!
3. Create new BProject *PwnAgent*, and get a shell on the build agents.
4. Run Mimikatz to dump login creds and get `bruno`'s password.
5. Run `powershell/lateral_movement/invoke_smbexec` to get beacon on Bruno-PC via NTML hash.
6. Loot Bruno's PC, steal his Chrome credentials.


### ToDo
1. Deploy with Terraform.
2. Install things with Ansible.
4. Work on method to establish persistence...my repo sux
5. Build into Terraform/Ansible.



# Defenses

### Endpoint Security
PwnAgent01 has Microsoft Defender enabled. Although it's certainly still possible to defeat this, the malicious build step we demonstrated will be blocked.  
![blocked0](https://user-images.githubusercontent.com/8961705/155248314-9d28ef64-1a5f-4abf-aceb-448158efa4ea.png)  

![blocked](https://user-images.githubusercontent.com/8961705/155248281-6b07edea-04cb-42d8-934c-7c26f0f4259f.png)  

![bruno_edr](https://user-images.githubusercontent.com/8961705/155440006-10a0cc2d-fd86-4239-a2c2-0c6f7ed96c26.png)



# Extra Credit
* Defeat Microsoft Defender and get an Empire agent to launch on `PwnAgent01` via TeamCity.
* Enable Defender on Bruno-PC, and dump creds. 


# Administrative Notes  
This section is just a collection of snippets that were useful when administering the lab environment.  

#### RDP into Windows Hosts  
1. From Kali, dynamic port forward on `TeamCity` host to access local resources `ssh -D 9050 x@52.234.0.18`.  
2. RDP via Proxychains with `proxychains4 xfreerdp /u:dev /v:10.0.0.6:3389.



# References
1. [TeamCity Hardening Guide](https://blog.jetbrains.com/teamcity/2021/02/hardening-your-teamcity-server/)
2. [TeamCity SuperUser](https://www.jetbrains.com/help/teamcity/super-user.html)


