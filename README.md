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
**Note:** Operational security is largely ignored here since this is a demo. 

### Initial Access
1. Scan the IP to discover SSH and TeamCity `nmap -Pn -p- 52.234.0.18`.
   * Note: It won't respond to ICMP, so `nmap 52.234.0.18` makes it seem dead.
   * Note: If you only scan the top 1000 TCP ports, you miss TeamCity `nmap -Pn 52.234.0.18`.
   ![portscan](https://user-images.githubusercontent.com/8961705/155561063-0a8a142d-4675-4b3e-a811-acc53b561339.png)  
2. Browse to the TeamCity URL [http://52.234.0.18:8111](http://52.234.0.18:8111).
    ![teamcitylogin](https://user-images.githubusercontent.com/8961705/155561509-4260185a-2f67-48b2-a4d9-b66b0ea4c79f.png)
3. Navigate to [http://52.234.0.18:8111/registerUser.html](http://52.234.0.18:8111/registerUser.html) create a new user `bob`, password `bobhacks?`.  
![bobregister](https://user-images.githubusercontent.com/8961705/155562055-8fadb90b-0718-4070-93c2-708465b92a4e.png)
4. As `bob` navigate to `Projects > SimpleMavenSample > Build > Settings` 
  * See that `Parameters` contains credentials. 
  ![leakcreds](https://user-images.githubusercontent.com/8961705/155563479-e85fdc1a-5c4c-4bcf-b339-8a93fd093533.png)
5. See if credentials are reused for `ssh dev@168.62.29.0`, and ssh in as low privileged user. 
  ![sshin](https://user-images.githubusercontent.com/8961705/155565080-c4de89b3-d8ff-4428-a47e-f00b82a14c30.png)

### From the Foothold
We could tunnel from our initial foothold. Knowing that RDP is open on two build agents would allow us to attempt to authenticate via the creds we've found...but that's not as fun.
1. Explore the TeamCity server a bit and check out the upser user token `cat /home/dev/TeamCity/TeamCity/logs/teamcity-server.log | grep "Super user"`.  
![sutoken](https://user-images.githubusercontent.com/8961705/155600482-0fbff1f2-18a2-4d90-a4c4-1d7027d616e6.png)
2. Now login as the Super!
![superlogin](https://user-images.githubusercontent.com/8961705/155605281-3775750d-9d5b-4a46-8267-49a766cf2626.png)
3. Create new Project *PwnAgent* via `Administration > Projects > Create project`, and get a shell on the build agents.
![createproject](https://user-images.githubusercontent.com/8961705/155601956-8804c90b-24c8-43da-8a72-ee5354d6fe49.png)
![buildsteps](https://user-images.githubusercontent.com/8961705/155602113-3e81cf04-3ef0-402b-8d7c-c6a7b4024a96.png)
4. Edit the build step so that it executes *Always, even if build stop command was issues*, and modify the following:
    * **Command executable:** `cmd.exe`
    * **Command parameters:** `/c %system.teamcity.build.checkoutDir%/launcher.bat`  
    * Once you can see how these work, you understand how code execution works here, and can modify it to do what you like. 
    ![build_step](https://user-images.githubusercontent.com/8961705/155602912-184f4977-254d-430e-b365-6f7bcaed0bd0.png)
    * Alternatively, you can avoid using files in the repo, leaving it blank. All code can be shoved into a build step.   
    ![alternative](https://user-images.githubusercontent.com/8961705/155603599-017ce751-5f04-432d-a531-ceec6940eae7.png)
5. Select the `...` next to `Run` on the menu, and then on the desired agent you're targeting. If all goes well you'll have an agent call back.  
![run](https://user-images.githubusercontent.com/8961705/155604796-0fb0cecb-0970-4dc6-9e3a-b30f86556b77.png)
![agentcallback](https://user-images.githubusercontent.com/8961705/155604811-80bef586-8f31-42dd-8b73-3464f0108822.png)
6. Run Mimikatz to dump login creds and get `bruno`'s password.
7. Run `powershell/lateral_movement/invoke_smbexec` to get beacon on Bruno-PC via NTML hash.
9. Loot Bruno's PC, steal his Chrome credentials.


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
* Determine the other ways in/around the network that aren't directly outlined above.


# Administrative Notes  
This section is just a collection of snippets that were useful when administering the lab environment.  

#### RDP into Windows Hosts  
1. From Kali, dynamic port forward on `TeamCity` host to access local resources `ssh -D 9050 dev@52.234.0.18`.  
2. RDP via Proxychains with `proxychains4 xfreerdp /u:dev /v:10.0.0.6:3389`.



# References
1. [TeamCity Hardening Guide](https://blog.jetbrains.com/teamcity/2021/02/hardening-your-teamcity-server/)
2. [TeamCity SuperUser](https://www.jetbrains.com/help/teamcity/super-user.html)
3. [Pentest TeamCity](https://github.com/kacperszurek/pentest_teamcity)


