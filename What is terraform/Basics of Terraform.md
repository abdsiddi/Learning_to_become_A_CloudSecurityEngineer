Terraform is basically a tool that allows Iac which stands for ( Infrastructure as code ), It allows us to define , and manage cloud
infrastructure using code rather than the grapghical user interface.
So basically instead of logging in to the lets say aws cloud website and then cliking buttons to create 
servers and satabses, networks, you write a configuration file describing what you want and then terraform builds it for you.
   Now i know what you aare thinking i thought the same , why the hell!!! musat i learn yet another damn language
   why cant i just click buttons seems easier no?
    It all comes down to saving time!, saving time building and saving time fixing, saving time auditing.
    How you ask? Good Question Dumbass
     Version Control: Ever heard about Git? it basically tracks shit : Every change is reviewed, documented, and easily reversible.  
     you make a change dont like it? roll right back and plus since you typed it you also know what exactly you did even a month from then.
     but on the other hand GUI, well nobody is tracking them clicks of yours. 
     So if something breaks, have fun opening tabs.
     Instant Replica: If i as your boss, cause lets face it i am ordered you to build me a cloud environment (whats in it i leave to you imagination, but imagine it being painfully complex tons ec2s and s3 and iam roles and policies blah blah blah)
     you spend a bunch of days building it, but since you are script kiddie or in my language a bitch ! , you used the GUI but since you are a good egg you actually built it.
     and then came ruinning to me daddy daddy look i did it, and just like your actual father who left you when you were an infant.
     i dont show any appreciation and instead i tell you to build exactly the same thing for the Staging environment as you did for Production environment..., see now you have to do all the clicks again but you 
     dont either remeber what you did the first time or even if you now have to spend hours building it again but if you had wriiting via terraform , you just have to change a variable or two and run the code again and in 2 minutes boom!! you got another one.
     Disaster Recovery: If your infrastructure gets deleted or compromised, your code acts as a blueprint to rebuild the entire system instantly. Rebuilding via GUI relies on memory or outdated wikis.
     Error-Free Scale: Deploying 50 identical servers via code takes the same effort as deploying one. Manual GUI clicking is slow and guarantees human error (like forgetting to check a security box on the 40th server).
There are others like terraform but since this is the most widley used , You just learning this one.
The best place to begin learning it , well its the CREATORS Hashicorp   https://developer.hashicorp.com/terraform/tutorials

![HashiCorp Terraform Tutorials](./terraform-tutorials.png)

Now youll see azure, aws etc in the begining just pick one dont try both i know you think your a smart ass but for once listen to PAPA!
I picked AWS:
Here youll learn how to create Terraform.tf what it is , main.tf , variables.tc, output.tf and input.tf and what each of these files actullay do just go through the course take like 3 or 4 days for this.
theyll also teach you how to build and HCP account so you can share thee state file secure terraform use and actually collaborate with people.
(Also get a free aws account cuz i know you broke, when you build something always type terraform destroy )
So after doin the above i went ahead and built one of my own, if you dont know how to build it read it and try figuring out what each block is doing:
