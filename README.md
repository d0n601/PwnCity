# PwnCity

## Red Team Infrastructure
Describe a bit about the servers we've got. 

## Demo
There may be more than one path through PwnCity, but this is the one I'll be presenting on Feb 24th at the [OWASP Sacramento Chapter](https://owasp.org/www-chapter-sacramento/) meeting.

### Initial Access
1. Scan the IP to discover SSH and TeamCity `nmap -p- blabla`.
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
5. Setup a tunnel and RDP into `bruno`...
6. Loot Machine.


### ToDo
1. Create user `bruno` on BuildAgents, with creds found on `rockyou.txt`.
2. Login as `bruno` on BuildAgents so that creds are stored plaintext.
3. Spin up new Windows 10 VM as `bruno-PC` and use same creds as above.
4. Put secrets on Bruno's PC.
  * SSH keys to source code repositories. 
  * Passwords stored in Google Chrome.
5. Work on method to establish persistence...my repo sux
6. Build into Terraform/Ansible.



# Defenses

### Endpoint Security
PwnAgent01 has Microsoft Defender enabled. Although it's certainly still possible to defeat this, the malicious build step we demonstrated will be blocked.  
![blocked0](https://user-images.githubusercontent.com/8961705/155248314-9d28ef64-1a5f-4abf-aceb-448158efa4ea.png)  

![blocked](https://user-images.githubusercontent.com/8961705/155248281-6b07edea-04cb-42d8-934c-7c26f0f4259f.png)



# Extra Credit
* Defeat Microsoft Defender and get an Empire agent to launch on `PwnAgent01` via TeamCity.



# References
1. [TeamCity Hardening Guide](https://blog.jetbrains.com/teamcity/2021/02/hardening-your-teamcity-server/)
2. [TeamCity SuperUser](https://www.jetbrains.com/help/teamcity/super-user.html)
