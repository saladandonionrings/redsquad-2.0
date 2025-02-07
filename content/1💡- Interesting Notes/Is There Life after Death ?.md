![[death.gif]]

**After compromising an infrastructure, web app, what is next ?**

The MITRE ATT&CK resource is a good starting point. The end result of the attack will look a bit different based on the **goal of the bad actor**. The goal might be to :
- Steal sensitive company/client/employee information, ransom, breaking/DoS applications,
- Defacement for financial/reputation loss.

> Regardless of the goal, I would start with discovery if I'm already inside the infrastructure/app.
> This would allow me to check what other systems are within the infrastructure (and later try to exploit them).
> **For the app :**
> - I would check what data it has/gathers.
> - If the app is based on micro-services,
> - What additional data I could gather from other systems/ apps.
> 
> **Dump IT ALL :**
> - I would download everything I gain access to. That might be user data or employee data,
> - If I gained access to an employee's PC I would download everything they have there and everything they have access to like data, emails, personal pics, and so on.
> - I would then analyze the data to see if there is anything I can sell, bribe for, and any details that would open other attack vectors (phishing, spear phishing, whaling, access/info about other systems, etc.).
> **After the discovery part is done, I would try to gain access to other systems as lateral movement and do again the first and second step when I'm in.**
> 
> **How would I get it?**
> Based on the data I've obtained when doing the discovery part I might've found some vulnerable systems, old systems with existing CVE's or just found some credentials on the employee computer.
> 
> **Privilege Escalation**
> This could be done by using vulnerabilities in old systems, stealing tokens, modifying the app to steal and send details to remote systems, maybe creating some fake apps that look like internal apps and steal credentials, and so on.
> Whatever I do next, I want to still have access to the app/infrastructure so I would start placing **backdoors**, **reverse shells**, creating **admin** **accounts**, having open sessions to the employee like RDP, and maybe leave **open ssh tunnels**.
> 
> **The last step**
> _It would be different depending on my goal._
> If this is only money or destruction I would ransomware the hell out of them, systems, databases, user data, employee data, and pretty much everything that I had access to.
> If I want to just steal data, I would not do anything destructive and stay in low profile to have the data coming in until they notice it and try to cut me off. I would deface their sites to show off or promote myself.


This scheme explains well : 
![[Pasted image 20241025154558.png]]

# Sources
https://www.reddit.com/r/HowToHack/comments/wnsbwh/is_there_life_after_death/
