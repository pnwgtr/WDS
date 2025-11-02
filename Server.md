---


---

<h1 id="poweredge-r420-–-from-bare-metal-to-usaf-sdc-imaging-server-windows-server-2022"><strong>PowerEdge R420 – From Bare Metal to USAF SDC Imaging Server (Windows Server 2022)</strong></h1>
<hr>
<h2 id="table-of-contents"><strong>Table of Contents</strong></h2>
<ol>
<li>
<p>[Overview &amp; Purpose]</p>
</li>
<li>
<p>[Prerequisites]</p>
</li>
<li>
<p>[Hardware Preparation]</p>
</li>
<li>
<p>[BIOS and RAID Configuration]</p>
</li>
<li>
<p>[Installing Windows Server 2022]</p>
</li>
<li>
<p>[Network Configuration (Dual NIC Setup)]</p>
</li>
<li>
<p>[DHCP Server Setup]</p>
</li>
<li>
<p>[Windows Deployment Services (WDS) Setup]</p>
</li>
<li>
<p>[Importing and Deploying the USAF SDC Image]</p>
</li>
<li>
<p>[PXE Boot and Endpoint Imaging]</p>
</li>
<li>
<p>[Common Issues &amp; Troubleshooting</p>
</li>
<li>
<p>[Appendix: Reference Tables &amp; Commands]</p>
</li>
</ol>
<hr>
<h2 id="overview--purpose"><strong>1. Overview &amp; Purpose</strong></h2>
<p>This guide walks technicians through building a  <strong>Windows Server 2022–based imaging server</strong>  on a  <strong>Dell PowerEdge R420</strong>.<br>
By the end, the server will:</p>
<ul>
<li>
<p>Provide DHCP to an isolated imaging subnet.</p>
</li>
<li>
<p>Host a Windows Deployment Services (WDS) environment.</p>
</li>
<li>
<p>Deploy  <strong>USAF Standard Desktop Configuration (SDC)</strong>  images to endpoints via PXE boot.</p>
</li>
</ul>
<p>The process covers:</p>
<ul>
<li>
<p>Bare-metal setup</p>
</li>
<li>
<p>RAID configuration</p>
</li>
<li>
<p>Server 2022 installation</p>
</li>
<li>
<p>Dual-NIC networking</p>
</li>
<li>
<p>DHCP/WDS roles</p>
</li>
<li>
<p>Endpoint imaging and troubleshooting</p>
</li>
</ul>
<hr>
<h2 id="prerequisites"><strong>2. Prerequisites</strong></h2>
<h3 id="hardware"><strong>Hardware</strong></h3>
<ul>
<li>
<p>Dell PowerEdge R420 (12-core, dual NICs)</p>
</li>
<li>
<p>Minimum 32 GB RAM</p>
</li>
<li>
<p>At least two drives (SSD recommended)</p>
</li>
<li>
<p>Ubiquiti Flex Mini 2.5G Switch</p>
</li>
<li>
<p>Ethernet cables for imaging clients</p>
</li>
</ul>
<h3 id="software"><strong>Software</strong></h3>
<ul>
<li>
<p>Windows Server 2022 ISO (Evaluation OK for 180 days)<br>
→ Download:  <a href="https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022">Microsoft Evaluation Center</a></p>
</li>
<li>
<p>USAF SDC image (WIM format)</p>
</li>
<li>
<p>Optional: Dell iDRAC access for remote management</p>
</li>
</ul>
<hr>
<h2 id="hardware-preparation"><strong>3. Hardware Preparation</strong></h2>
<ol>
<li>
<p>Connect a monitor and keyboard or access the iDRAC web console.</p>
</li>
<li>
<p>Power on the R420 and enter  <strong>BIOS Setup</strong>  by pressing  <strong>F2</strong>.</p>
</li>
<li>
<p>Ensure the following settings:</p>
<ul>
<li>
<p><strong>Boot Mode:</strong>  UEFI</p>
</li>
<li>
<p><strong>Integrated NICs:</strong>  Enabled</p>
</li>
<li>
<p><strong>Thermal Mode:</strong>  <em>Minimum Power</em>  (for quieter operation)</p>
</li>
<li>
<p><strong>System Profile:</strong>  <em>Performance per Watt (DAPC)</em></p>
</li>
</ul>
</li>
<li>
<p>Save and exit BIOS.</p>
</li>
</ol>
<hr>
<h2 id="bios-and-raid-configuration"><strong>4. BIOS and RAID Configuration</strong></h2>
<p><strong>"We only have 2 drive so we are limited on RAID options"</strong></p>
<ol>
<li>
<p>When prompted during boot, press  <strong>Ctrl+R</strong>  to open the  <strong>PERC RAID controller</strong>.</p>
</li>
<li>
<p>Delete any previous Virtual Disks if present.</p>
</li>
<li>
<p>Create a new  <strong>Virtual Disk</strong>:</p>
<ul>
<li>
<p>RAID Level:  <strong>RAID 1</strong>  (for redundancy)</p>
</li>
<li>
<p>Drives: Select your two primary drives</p>
</li>
<li>
<p>Initialization:  <em>Fast Init</em></p>
</li>
</ul>
</li>
<li>
<p>Confirm and initialize the volume.</p>
</li>
<li>
<p>Exit and reboot.</p>
</li>
</ol>
<hr>
<h2 id="installing-windows-server-2022">5. Installing Windows Server 2022</h2>
<p>This section covers installing Windows Server 2022 Standard (Desktop Experience) onto the Dell PowerEdge R420.<br>
Even though this model supports RAID, your training lab can install directly to a single drive if needed.</p>
<hr>
<h3 id="start-the-installation">5.1 Start the Installation</h3>
<ol>
<li>Insert your <strong>Windows Server 2022 USB installer</strong> or mount the ISO using <strong>iDRAC → Virtual Media → Mount ISO</strong>.</li>
<li>Boot the server. When prompted, press <strong>F11 → Boot Manager → One-Time Boot → Removable Drive</strong> (or choose the mounted ISO).</li>
<li>When the Windows setup screen appears:
<ul>
<li>Choose language, time, and keyboard → <strong>Next</strong></li>
<li>Click <strong>Install Now</strong></li>
</ul>
</li>
<li>When asked for edition, select <strong>Windows Server 2022 Standard (Desktop Experience)</strong> → <strong>Next</strong></li>
<li>Accept the license terms → <strong>Next</strong></li>
<li>Choose <strong>Custom: Install Windows only (advanced)</strong></li>
</ol>
<hr>
<h3 id="select-or-prepare-the-drive">5.2 Select or Prepare the Drive</h3>
<p>At this screen you’ll see a list of available disks.</p>
<h4 id="if-drives-are-visible">If drives are visible:</h4>
<ul>
<li>Highlight the desired disk (for example, <strong>Drive 0 Unallocated Space</strong>).</li>
<li>Click <strong>Next</strong> and Windows will automatically create the needed partitions.</li>
</ul>
<h4 id="if-no-drives-appear">If <strong>no drives appear</strong>:</h4>
<ol>
<li>Click <strong>Load Driver</strong> at the bottom left.</li>
<li>Insert or mount the Dell storage driver package (downloaded from Dell Support → R420 → Drivers → PERC or SATA controller).</li>
<li>Browse to the folder containing the <code>.inf</code> file and click <strong>OK</strong>.</li>
<li>Once drivers load, your disk(s) should appear.</li>
</ol>
<p><em>Technician Tip:<br>
If using AHCI mode instead of RAID, you may not need Dell’s PERC driver—Windows 2022 usually detects it automatically.</em></p>
<pre class=" language-5"><code class="prism .3 language-5">If you see **“Windows cannot install to this disk”** or similar:

1. Open a command prompt by pressing **Shift + F10**.  
2. Type:
   ```powershell
   diskpart
   list disk
   select disk 0
   clean
   exit
This removes any old partitions.
3. Click Refresh in the installer and try again.
4. Select the cleaned drive → Next.
</code></pre>
<p><em>Technician Tip:<br>
Use the “clean” command only when you are sure nothing important is on the disk—it erases all partitions instantly.</em></p>
<p><strong>5.4 Installation Time</strong><br>
Setup typically takes 10–15 minutes.</p>
<p>The system will reboot several times; do not remove the USB/ISO until the login screen appears.</p>
<p>When you reach the Administrator password prompt, create a strong local password.</p>
<p><strong>5.5 Post-Install Configuration</strong><br>
After the desktop loads:</p>
<p>Log in as Administrator.</p>
<p>Open Server Manager → Local Server.</p>
<p>Disable IE Enhanced Security Configuration for Administrators (makes downloads easier).</p>
<p>Optionally rename the server:</p>
<pre><code>***powershell***
Rename-Computer -NewName "R420-IMAGING" -Restart
</code></pre>
<p><em>Technician Tip:<br>
Always reboot after renaming.  If you skip this, DHCP and WDS roles later will register under the wrong hostname.</em></p>
<hr>
<h3 id="installation-verification-checklist">5.6 Installation Verification Checklist</h3>
<p>Use this checklist immediately after Windows Server 2022 finishes installing.<br>
It ensures the OS, disk, and NICs are healthy before continuing to configuration.</p>

<table>
<thead>
<tr>
<th><strong>Step</strong></th>
<th><strong>Action</strong></th>
<th><strong>Expected Result</strong></th>
<th><strong>Pass (✓)</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>1</td>
<td>Log in as <strong>Administrator</strong></td>
<td>Desktop loads without error messages</td>
<td></td>
</tr>
<tr>
<td>2</td>
<td>Press <strong>Ctrl + Shift + Esc → Task Manager → Performance → Memory/CPU</strong></td>
<td>CPU and RAM values appear (proves hardware recognized)</td>
<td></td>
</tr>
<tr>
<td>3</td>
<td>Press <strong>Win + X → Disk Management</strong></td>
<td>System (C:) appears, healthy, ~60–80 GB free</td>
<td></td>
</tr>
<tr>
<td>4</td>
<td>In Disk Management, right-click C: → <strong>Properties → Tools → Check</strong></td>
<td>No errors reported</td>
<td></td>
</tr>
<tr>
<td>5</td>
<td><strong>Device Manager → Network adapters</strong></td>
<td>Two adapters appear (Broadcom/Intel) with no yellow warning icons</td>
<td></td>
</tr>
<tr>
<td>6</td>
<td>Open PowerShell → run <code>Get-NetAdapter</code></td>
<td>Both NICs listed as <em>Up</em></td>
<td></td>
</tr>
<tr>
<td>7</td>
<td>Run <code>ping 8.8.8.8</code> (if connected to Internet NIC)</td>
<td>Replies received</td>
<td></td>
</tr>
<tr>
<td>8</td>
<td>Open <strong>Server Manager → Dashboard</strong></td>
<td>Server shows “Windows Server 2022 Standard (Desktop Experience)”</td>
<td></td>
</tr>
<tr>
<td>9</td>
<td>Confirm correct time zone and system clock</td>
<td>Matches local time</td>
<td></td>
</tr>
<tr>
<td>10</td>
<td>Take snapshot or note configuration</td>
<td>Baseline recorded before role installation</td>
<td></td>
</tr>
</tbody>
</table><p><em>Technician Tip:<br>
If any step fails, correct it before installing roles.<br>
Networking and storage issues early in the build will cause DHCP or WDS to fail later.</em></p>
<h3 id="post-install-configuration"><strong>Post-install Configuration</strong></h3>
<ul>
<li>
<p>Username:  <code>Administrator</code></p>
</li>
<li>
<p>Password: create a strong local admin password.</p>
</li>
<li>
<p>Log in and open:</p>
<p><code>Server Manager → Local Server</code></p>
<p>Turn off IE Enhanced Security Configuration for admins.</p>
</li>
</ul>
<hr>
<h2 id="network-configuration-dual-nic-setup"><strong>6. Network Configuration (Dual NIC Setup)</strong></h2>
<p>The Dell R420 has two physical network adapters (NICs).</p>
<p>For this lab, one will connect to the**.COM** for Internet and updates, and the other will serve as the <strong>isolated imaging network</strong> for PXE deployments.</p>
<p>This dual-NIC configuration ensures that PXE traffic does not interfere with your production LAN.</p>
<hr>
<h3 id="physical-connections">6.1  Physical Connections</h3>
<ul>
<li>
<p><strong>NIC1 (Internet / Management):</strong><br>
Connects to your corporate network or router. Provides Internet access and Windows updates.</p>
</li>
<li>
<p><strong>NIC2 (Imaging):</strong><br>
Connects directly to the <strong>Ubiquiti Flex Mini switch</strong>, which serves the isolated imaging subnet.<br>
All client PCs that will be imaged connect to this same switch.</p>
</li>
</ul>
<p><em>Technician Tip:<br>
Never plug the imaging switch into your corporate LAN.<br>
The DHCP server on this subnet will issue addresses meant only for PXE clients.</em></p>
<hr>
<h3 id="rename-network-adapters">6.2  Rename Network Adapters</h3>
<ol>
<li>Open <strong>Control Panel → Network and Internet → Network Connections</strong><br>
<em>(You can also type <code>ncpa.cpl</code> in the Run dialog.)</em></li>
<li>Right-click each adapter and choose <strong>Rename</strong>:
<ul>
<li><code>Internet</code> – for the adapter connected to your LAN</li>
<li><code>Imaging</code> – for the adapter connected to the Ubiquiti switch</li>
</ul>
</li>
</ol>
<p><em>Technician Tip:<br>
Label them physically with tape if needed.<br>
Confusing the two NICs is one of the most common mistakes during imaging setup.</em></p>
<hr>
<h3 id="assign-static-ip-addresses">6.3  Assign Static IP Addresses</h3>
<p>You’ll give each adapter a fixed IP address so the server always knows which network is which.</p>
<h4 id="nic1-–-internet--management">NIC1 – Internet / Management</h4>
<ol>
<li>
<p>Right-click <strong>Internet → Properties → Internet Protocol Version 4 (TCP/IPv4)</strong> → <strong>Properties</strong></p>
</li>
<li>
<p>Enter:<br>
IP address: 192.168.1.50<br>
Subnet mask: 255.255.255.0<br>
Default gateway: 192.168.1.1<br>
Preferred DNS server: 8.8.8.8</p>
</li>
<li>
<p>Click <strong>OK → Close.</strong></p>
</li>
</ol>
<h4 id="nic2-–-imaging-network">NIC2 – Imaging Network</h4>
<ol>
<li>
<p>Right-click <strong>Imaging → Properties → Internet Protocol Version 4 (TCP/IPv4)</strong> → <strong>Properties</strong></p>
</li>
<li>
<p>Enter:<br>
IP address: 192.168.50.1<br>
Subnet mask: 255.255.255.0<br>
Default gateway: (leave blank)<br>
Preferred DNS server: (leave blank)</p>
</li>
<li>
<p>Click <strong>OK → Close.</strong></p>
</li>
</ol>
<p><em>Technician Tip:<br>
Do not assign a gateway to the Imaging NIC.<br>
If you do, Windows may send Internet traffic out through the wrong adapter, breaking WDS and PXE communication.</em></p>
<hr>
<h3 id="verify-network-configuration">6.4  Verify Network Configuration</h3>
<p>Open PowerShell and run:</p>
<pre class=" language-powershell"><code class="prism  language-powershell">Get<span class="token operator">-</span>NetIPAddress
Expected output:
<span class="token function">Copy</span> code
IPAddress         InterfaceAlias
<span class="token operator">--</span>-<span class="token operator">--</span>-<span class="token operator">--</span>-<span class="token operator">-</span>        <span class="token operator">--</span>-<span class="token operator">--</span>-<span class="token operator">--</span>-<span class="token operator">--</span>-<span class="token operator">--</span>-<span class="token operator">-</span>
192<span class="token punctuation">.</span>168<span class="token punctuation">.</span>1<span class="token punctuation">.</span>50      Internet
192<span class="token punctuation">.</span>168<span class="token punctuation">.</span>50<span class="token punctuation">.</span>1      Imaging
</code></pre>
<p>If you see both addresses listed, your static IPs are configured correctly.</p>
<p><strong>6.5 Optional: Rename Interfaces via PowerShell</strong><br>
You can also rename and assign IPs entirely through PowerShell:</p>
<pre><code>powershell
Rename-NetAdapter -Name "Ethernet0" -NewName "Internet"
Rename-NetAdapter -Name "Ethernet1" -NewName "Imaging"

New-NetIPAddress -InterfaceAlias "Internet" -IPAddress 192.168.1.50 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Internet" -ServerAddresses 8.8.8.8

New-NetIPAddress -InterfaceAlias "Imaging" -IPAddress 192.168.50.1 -PrefixLength 24
</code></pre>
<p><em>Technician Tip:<br>
If you reimage or replace the network card, these names may revert.<br>
Always confirm NIC names after reinstalling Windows or updating drivers.</em></p>
<p>6.6 Troubleshooting Common Issues<br>
Symptom	Cause	Fix<br>
“Unidentified Network” warning	No gateway or DHCP present	Safe to ignore on Imaging NIC<br>
Only one NIC has Internet	Normal	Imaging NIC is isolated intentionally<br>
PXE clients can’t contact WDS	Wrong cable or DHCP inactive	Ensure Imaging NIC (192.168.50.1) is cabled to switch and scope is Active<br>
Internet connection drops when Imaging NIC enabled	Routing conflict	Confirm Imaging NIC has no gateway configured</p>
<p>6.7 Verification Checklist<br>
Step	Action	Expected Result	Pass (✓)<br>
1	Both NICs visible in ncpa.cpl	“Internet” and “Imaging” appear	<br>
2	Get-NetIPAddress shows two unique subnets	192.168.1.50 and 192.168.50.1	<br>
3	Ping 8.8.8.8 from server	Internet NIC working	<br>
4	Ping 192.168.50.1 from a test client (if connected)	Imaging NIC reachable	<br>
5	No “default gateway” listed for Imaging NIC</p>
<h2 id="dhcp-server-setup"><strong>7. DHCP Server Setup</strong></h2>
<ol>
<li>
<p>In  <strong>Server Manager → Add Roles and Features</strong>, select:</p>
<ul>
<li><strong>DHCP Server</strong></li>
</ul>
</li>
<li>
<p>After install, open  <strong>DHCP Management Console</strong>:</p>
<p><code>Tools → DHCP</code></p>
</li>
<li>
<p>Right-click your server →  <strong>New Scope…</strong></p>
<ul>
<li>
<p>Name:  <code>Imaging Network</code></p>
</li>
<li>
<p>Start IP:  <code>192.168.50.10</code></p>
</li>
<li>
<p>End IP:  <code>192.168.50.200</code></p>
</li>
<li>
<p>Subnet Mask:  <code>255.255.255.0</code></p>
</li>
<li>
<p>Router:  <em>(leave blank)</em></p>
</li>
<li>
<p>DNS:  <em>(optional, 192.168.50.1)</em></p>
</li>
</ul>
</li>
<li>
<p>Activate the scope.</p>
</li>
</ol>
<p><strong>Verify:</strong></p>
<pre class=" language-powershell"><code class="prism  language-powershell">Powershell 
Get<span class="token operator">-</span>DhcpServerv4Scope` 
</code></pre>
<p><strong>Expected output:</strong></p>
<p><code>ScopeId: 192.168.50.0 StartRange: 192.168.50.10 EndRange: 192.168.50.200</code></p>
<h2 id="windows-deployment-services-wds-setup-–-complete-walkthrough">8. Windows Deployment Services (WDS) Setup – Complete Walkthrough</h2>
<p>This section installs <strong>Windows Deployment Services</strong>, configures it for PXE,<br>
and adds the <strong>USAF SDC boot and install images</strong>.<br>
Follow these steps exactly in order.</p>
<hr>
<h3 id="add-the-wds-role">8.1  Add the WDS Role</h3>
<ol>
<li><strong>Server Manager → Manage → Add Roles and Features</strong></li>
<li>Click <strong>Next</strong> four times until you reach <strong>Select Server Roles</strong>.</li>
<li>Scroll down and check  <strong>Windows Deployment Services</strong>.
<ul>
<li>A pop-up appears → click <strong>Add Features</strong>.</li>
</ul>
</li>
<li>Click <strong>Next</strong> until you reach <strong>Role Services</strong>.</li>
<li>On <strong>Role Services</strong> check <strong>both</strong> boxes:
<ul>
<li><strong>Deployment Server</strong></li>
<li><strong>Transport Server</strong></li>
</ul>
</li>
<li>Click <strong>Next → Install</strong>.<br>
Wait for “Installation Succeeded.”</li>
</ol>
<p>Technician Tip:<br>
If you forget to tick Deployment Server, clients will never PXE-boot—only Transport Server installs.<br>
Re-run “Add Roles and Features” if you miss it.</p>
<hr>
<h3 id="initial-wds-configuration">8.2  Initial WDS Configuration</h3>
<ol>
<li>Open <strong>Server Manager → Tools → Windows Deployment Services</strong>.</li>
<li>In the left pane, right-click your server name → <strong>Configure Server</strong>.</li>
<li>When prompted:
<ul>
<li><strong>RemoteInstall folder:</strong><br>
Select or create <code>D:\RemoteInstall</code> (if no D: drive, use C:).</li>
<li><strong>DHCP Options:</strong><br>
<em>Do not listen on DHCP ports</em><br>
<em>Configure DHCP options to indicate that this server is also a PXE server</em></li>
<li><strong>PXE Response:</strong> choose <em>Respond to all client computers (known and unknown)</em>.</li>
</ul>
</li>
<li>Finish → Close → Yes to start the WDS service.</li>
</ol>
<pre class=" language-verify"><code class="prism : language-verify">powershell
Get-Service wdsserver
You should see Status : Running
</code></pre>
<p>Technician Tip:<br>
If the wizard fails to start WDS, reboot once; Windows often needs to re-bind network ports after new roles install.</p>
<p><strong>8.3 Prepare Your Image Files</strong><br>
You need two files from your USAF SDC ISO.</p>
<p>Right-click the SDC ISO → Mount (it will appear as drive E:).</p>
<p>Open E:\Sources.</p>
<p>Locate boot.wim (boot image)</p>
<p>Locate install.wim (install image)</p>
<p>If you see install.esd instead, you must convert it first<br>
using DISM (see section 9 or below).</p>
<p><strong>8.4 Add the Boot Image</strong><br>
In WDS console → expand your server.</p>
<p>Right-click Boot Images → Add Boot Image.</p>
<p>Browse to E:\Sources\boot.wim.</p>
<p>Image Name: Windows PE x64 (USAF SDC)</p>
<p>Click Next → Finish.</p>
<p><em>Technician Tip:</em><br>
This boot.wim is what clients download first during PXE boot.<br>
Without it, PXE will time out after “TFTP Download.”</p>
<p><strong>8.5 Add the Install Image</strong><br>
Right-click Install Images → Add Install Image.</p>
<p>When prompted to create a group, name it USAF_SDC.</p>
<p>Browse to E:\Sources\install.wim.</p>
<p>Select it → Next → Finish.</p>
<p>Confirm both images show green check marks under your server.</p>
<p><strong>8.6 Verify and Test</strong><br>
Run these commands to confirm configuration:</p>
<h1 id="check-that-both-roles-are-installed">Check that both roles are installed</h1>
<pre><code>Powershell Get-WindowsFeature -Name WDS*,DHCP*
</code></pre>
<h1 id="check-service-status">Check service status</h1>
<pre><code>Powershell Get-Service wdsserver,dhcpserver
</code></pre>
<h1 id="list-images-known-to-wds">List images known to WDS</h1>
<pre><code>Powershellwdsutil /get-allimages /detailed
</code></pre>
<p><strong>Both boot.wim and install.wim should appear in the list.</strong></p>
<p><strong>8.7 Optional</strong> – Convert install.esd to install.wim<br>
Only if your ISO contains install.esd instead of install.wim:</p>
<pre class=" language-undefined"><code class="prism language-***powershell*** language-undefined">mkdir C:\Sources
copy E:\Sources\install.esd C:\Sources\
Dism /Export-Image /SourceImageFile:"C:\Sources\install.esd" /SourceIndex:1 `
     /DestinationImageFile:"C:\Sources\install.wim" /Compress:max /CheckIntegrity
</code></pre>
<p>Then import C:\Sources\install.wim as your install image.</p>
<p>Technician Tip:<br>
Conversion can take 5–10 minutes. Don’t close the window until you see “Export completed successfully.”</p>
<p>8.8 Restart and Final Check<br>
Reboot the server once to ensure WDS binds cleanly to the Imaging NIC.</p>
<p>After reboot, run:</p>
<pre class=" language-undefined"><code class="prism language-***powershell*** language-undefined">Powershell Get-Service wdsserver
</code></pre>
<p>It should be Running (Automatic).</p>
<p>Verify the PXE settings:</p>
<p>DHCP scope 192.168.50.0 active</p>
<p>WDS shows both boot and install images</p>
<p>Services set to Automatic startup</p>
<h1 id="field-verification-test-–-confirm-wds-readiness">8.9 Field Verification Test – Confirm WDS Readiness</h1>
<p>Use this 10-minute check before moving to PXE imaging.<br>
Every step must succeed before a workstation can deploy an image.</p>
<hr>
<h3 id="objective"><strong>Objective</strong></h3>
<p>Verify that:</p>
<ul>
<li>DHCP is leasing addresses on the imaging subnet</li>
<li>WDS is responding to PXE requests</li>
<li>Boot and install images are present and valid</li>
<li>The server is bound to the correct NIC</li>
</ul>
<h3 id="checklist"><strong>Checklist</strong></h3>

<table>
<thead>
<tr>
<th>Step</th>
<th>Action</th>
<th>Expected Result</th>
<th>Pass (✓)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>1</strong></td>
<td>From PowerShell: <code>Get-Service wdsserver</code></td>
<td>Status shows <strong>Running</strong></td>
<td></td>
</tr>
<tr>
<td><strong>2</strong></td>
<td>From PowerShell: <code>Get-Service dhcpserver</code></td>
<td>Status shows <strong>Running</strong></td>
<td></td>
</tr>
<tr>
<td><strong>3</strong></td>
<td>From PowerShell: <code>Get-DhcpServerv4Scope</code></td>
<td>Lists scope <strong>192.168.50.0</strong> Active = True</td>
<td></td>
</tr>
<tr>
<td><strong>4</strong></td>
<td>Open <strong>WDS Console → Boot Images</strong></td>
<td><code>Windows PE x64 (USAF SDC)</code> shows green check</td>
<td></td>
</tr>
<tr>
<td><strong>5</strong></td>
<td>Open <strong>WDS Console → Install Images</strong></td>
<td><code>USAF_SDC</code> image group with install.wim present</td>
<td></td>
</tr>
<tr>
<td><strong>6</strong></td>
<td>From PowerShell: <code>wdsutil /get-allimages /detailed</code></td>
<td>Both boot and install images listed</td>
<td></td>
</tr>
<tr>
<td><strong>7</strong></td>
<td>Plug a spare PC into the imaging switch and power it on</td>
<td>Client displays “Press F12 for Network Boot”</td>
<td></td>
</tr>
<tr>
<td><strong>8</strong></td>
<td>Press F12 at prompt</td>
<td>“Downloading boot.wim …” appears</td>
<td></td>
</tr>
<tr>
<td><strong>9</strong></td>
<td>When WinPE loads, cancel the wizard (no deployment)</td>
<td>Confirms successful PXE connection</td>
<td></td>
</tr>
<tr>
<td><strong>10</strong></td>
<td>Disconnect client</td>
<td>Ready for full USAF SDC imaging</td>
<td></td>
</tr>
</tbody>
</table><hr>
<h3 id="troubleshooting-if-a-step-fails"><strong>Troubleshooting if a Step Fails</strong></h3>

<table>
<thead>
<tr>
<th>Issue</th>
<th>Common Fix</th>
</tr>
</thead>
<tbody>
<tr>
<td>WDS service stopped</td>
<td><code>Restart-Service wdsserver</code></td>
</tr>
<tr>
<td>DHCP scope missing</td>
<td>Recreate 192.168.50.0/24 scope and activate</td>
</tr>
<tr>
<td>No PXE prompt on client</td>
<td>Check cable; ensure Imaging NIC = 192.168.50.1; verify “Do not listen on DHCP ports” checked</td>
</tr>
<tr>
<td>Boot image missing</td>
<td>Re-add <code>boot.wim</code> under Boot Images</td>
</tr>
<tr>
<td>Client downloads but errors after “Starting Windows”</td>
<td>Re-import both WIMs and reboot server</td>
</tr>
</tbody>
</table><p>Technician Tip:<br>
Run this test anytime you rebuild or patch the server.<br>
If any step fails, fix it before moving on to Section 10 (PXE Boot and Imaging).</p>
<p>Technician Tip:<br>
If PXE clients still don’t respond, disable and re-enable the Imaging NIC in Network Connections—this refreshes the binding between WDS and DHCP.</p>
<hr>
<h2 id="importing-and-deploying-the-usaf-sdc-image">9. Importing and Deploying the USAF SDC Image</h2>
<p>This section walks you through importing the official <strong>USAF Standard Desktop Configuration (SDC)</strong> image into Windows Deployment Services (WDS) and preparing it for deployment.</p>
<p>You will add two images:</p>
<ul>
<li><strong>Boot Image (boot.wim)</strong> – the WinPE environment clients load over PXE.</li>
<li><strong>Install Image (install.wim)</strong> – the actual USAF SDC operating system that gets applied to the workstation.</li>
</ul>
<hr>
<h3 id="locate-and-mount-the-image-source">9.1 Locate and Mount the Image Source</h3>
<ol>
<li>Determine where your USAF SDC image is stored:
<ul>
<li>If it’s an <strong>ISO file</strong>, right-click → <strong>Mount</strong>.</li>
<li>If it’s on a USB or shared drive, note the letter (e.g., <code>E:</code>).</li>
</ul>
</li>
<li>Navigate to the <strong>Sources</strong> folder:<br>
E:\Sources\</li>
</ol>
<p>markdown<br>
Copy code<br>
3. Verify the following files exist:</p>
<ul>
<li><code>boot.wim</code> (Windows Preinstallation Environment)</li>
<li><code>install.wim</code> (USAF SDC image)</li>
</ul>
<p>If you see <code>install.esd</code> instead of <code>install.wim</code>, you’ll need to convert it (see section 9.6).</p>
<p>Technician Tip:<br>
These two WIM files are the heart of every Windows deployment.<br>
The boot image provides the PXE environment; the install image is the operating system itself.</p>
<hr>
<h3 id="add-the-boot-image-winpe">9.2 Add the Boot Image (WinPE)</h3>
<ol>
<li>
<p>Open <strong>Server Manager → Tools → Windows Deployment Services</strong>.</p>
</li>
<li>
<p>Expand your server name in the left pane.</p>
</li>
<li>
<p>Right-click <strong>Boot Images → Add Boot Image</strong>.</p>
</li>
<li>
<p>Browse to your mounted ISO path:<br>
E:\Sources\boot.wim</p>
</li>
<li>
<p>Click <strong>Next</strong>.</p>
</li>
</ol>
<ul>
<li><strong>Image Name:</strong> <code>Windows PE x64 (USAF SDC Boot)</code></li>
<li><strong>Description:</strong> <code>Boot image for USAF SDC PXE deployment.</code></li>
</ul>
<ol start="6">
<li>Click <strong>Next → Finish.</strong></li>
</ol>
<p>You should now see the new boot image listed under <strong>Boot Images</strong> with a green check mark.</p>
<p>Technician Tip:<br>
If your boot image fails to add or shows a red X,<br>
the file may be locked or corrupt—copy it locally to C:\Temp before importing.</p>
<hr>
<h3 id="add-the-install-image-usaf-sdc-operating-system">9.3 Add the Install Image (USAF SDC Operating System)</h3>
<ol>
<li>
<p>In the same WDS console, right-click <strong>Install Images → Add Install Image.</strong></p>
</li>
<li>
<p>When prompted to create a group, type:<br>
USAF_SDC<br>
and click <strong>Next.</strong></p>
</li>
<li>
<p>Browse to:<br>
E:\Sources\install.wim</p>
</li>
<li>
<p>Click <strong>Next</strong> and allow WDS to enumerate the image contents.</p>
</li>
</ol>
<ul>
<li>If multiple editions appear (e.g., Standard, Datacenter), select the correct <strong>USAF-approved build</strong> (usually “Windows 10 Enterprise” or “Windows 11 Enterprise” depending on SDC version).</li>
</ul>
<ol start="5">
<li>Click <strong>Next → Finish.</strong></li>
<li>Expand <strong>Install Images</strong> → <strong>USAF_SDC</strong> group to verify your image appears with a green check.</li>
</ol>
<p>Technician Tip:<br>
Image groups in WDS let you organize by version or purpose.<br>
For example, you can later add USAF_SDC_Win11 alongside USAF_SDC_Win10.</p>
<hr>
<h3 id="verify-image-metadata">9.4 Verify Image Metadata</h3>
<p>From PowerShell, confirm WDS recognizes your images:</p>
<pre class=" language-powershell"><code class="prism  language-powershell"><span class="token comment"># List all boot images</span>
wdsutil <span class="token operator">/</span>get<span class="token operator">-</span>allimages <span class="token operator">/</span>imageType:Boot

<span class="token comment"># List all install images</span>
wdsutil <span class="token operator">/</span>get<span class="token operator">-</span>allimages <span class="token operator">/</span>imageType:Install
</code></pre>
<p>Expected output includes:<br>
Image Name : Windows PE x64 (USAF SDC Boot)<br>
Image Name : Windows 10 Enterprise (USAF SDC)</p>
<p>Technician Tip:<br>
If either image doesn’t appear, check the folder permissions for<br>
D:\RemoteInstall — SYSTEM and Administrators must have Full Control.</p>
<h3 id="validate-image-functionality-dry-run-test">9.5 Validate Image Functionality** (Dry-Run Test)</h3>
<p>Before imaging real machines, confirm PXE clients can load your new boot image.</p>
<p>Connect a spare workstation to the Imaging switch.</p>
<p>Boot the client → press F12 for Network Boot.</p>
<p>Verify:</p>
<p>It receives an IP from the DHCP scope (192.168.50.x range).</p>
<p>Displays “Press F12 for Network Service Boot”.</p>
<p>Downloads and loads the USAF SDC boot image.</p>
<p>Once WinPE loads and the image selection window appears, cancel (don’t deploy).<br>
This confirms both images are functional.</p>
<p><em><em>Technician Tip:</em><br>
If you reach the Windows Setup screen, your WDS configuration is working.<br>
Always cancel here to avoid accidentally overwriting a training workstation.</em></p>
<h3 id="optional-convert-install.esd-to-install.wim">9.6 (Optional) Convert install.esd to install.wim</h3>
<p>If your ISO contains install.esd instead of install.wim, convert it using the Deployment Image Servicing and Management (DISM) tool.</p>
<p>Copy the ESD file to a local folder:</p>
<pre class=" language-powershell"><code class="prism  language-powershell">powershell 
mkdir C:\Sources 
<span class="token function">copy</span> E:\Sources\install<span class="token punctuation">.</span>esd C:\Sources\

Dism <span class="token operator">/</span>Export<span class="token operator">-</span>Image <span class="token operator">/</span>SourceImageFile:<span class="token string">"C:\Sources\install.esd"</span> <span class="token operator">/</span>SourceIndex:1 `
     <span class="token operator">/</span>DestinationImageFile:<span class="token string">"C:\Sources\install.wim"</span> <span class="token operator">/</span>Compress:max <span class="token operator">/</span>CheckIntegrity
</code></pre>
<p>Import the new C:\Sources\install.wim file following steps in 9.3 Add the Install Image.</p>
<p><em><em>Technician Tip:</em><br>
This conversion only needs to be done once. Store the resulting install.wim with your other master images.</em></p>
<h2 id="post-import-verification-checklist">9.7 Post-Import Verification Checklist</h2>
<p>Check	Expected Result	Pass (✓)<br>
boot.wim imported under Boot Images	Green check icon visible	<br>
install.wim imported under Install Images	Green check icon visible	<br>
WDS service running	Get-Service wdsserver shows Status = Running	<br>
PXE client can load WinPE and reach setup screen	Verified via dry-run test	<br>
No errors in Event Viewer → Deployment-Services-Diagnostics	No red errors</p>
<hr>
<h2 id="pxe-boot-and-endpoint-imaging"><strong>10. PXE Boot and Endpoint Imaging</strong></h2>
<ol>
<li>
<p>Plug client PC into the  <strong>Ubiquiti Flex Mini Switch</strong>  connected to the Imaging NIC.</p>
</li>
<li>
<p>Power on and press  <strong>F12</strong>  → select  <strong>PXE Boot / Network Boot</strong>.</p>
</li>
<li>
<p>The sequence should be:</p>
<ul>
<li>
<p>Client obtains IP from DHCP</p>
</li>
<li>
<p>“Press F12 for Network Service Boot” appears</p>
</li>
<li>
<p>Downloads  <code>boot.wim</code>  from WDS</p>
</li>
<li>
<p>Windows Setup launches and shows the USAF SDC image list</p>
</li>
</ul>
</li>
<li>
<p>Select desired image and begin installation.</p>
</li>
</ol>
<p><strong>Verification Command on Server:</strong></p>
<p><code>Powershell: Get-WdsClient</code></p>
<p>Shows connected PXE clients.</p>
<hr>
<h2 id="common-issues--troubleshooting"><strong>11. Common Issues &amp; Troubleshooting</strong></h2>
<h2 id="common-issues--troubleshooting-1">11. Common Issues &amp; Troubleshooting</h2>

<table>
<thead>
<tr>
<th><strong>Issue</strong></th>
<th><strong>Cause</strong></th>
<th><strong>Fix</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>PXE boot fails (“No DHCP offer received”)</td>
<td>Client not getting IP</td>
<td>Check cable, switch, and confirm DHCP scope is activated.</td>
</tr>
<tr>
<td>PXE times out</td>
<td>DHCP and WDS ports conflict</td>
<td>Confirm “Do not listen on DHCP ports” was checked during WDS setup.</td>
</tr>
<tr>
<td>Client reboots after PXE screen</td>
<td>Boot image missing</td>
<td>Verify <code>boot.wim</code> exists in WDS Boot Images.</td>
</tr>
<tr>
<td>Imaging fails mid-deployment</td>
<td>Corrupt WIM or bad disk</td>
<td>Reimport image and verify RAID health.</td>
</tr>
<tr>
<td>Slow imaging speed</td>
<td>1 Gb NIC or old switch</td>
<td>Use the Ubiquiti Flex Mini 2.5G switch and ensure full-duplex mode is enabled.</td>
</tr>
</tbody>
</table><hr>
<h2 id="appendix-reference-tables--commands"><strong>12. Appendix: Reference Tables &amp; Commands</strong></h2>
<h3 id="ip-address-plan"><strong>IP Address Plan</strong></h3>
<h3 id="ip-address-plan-1">IP Address Plan</h3>

<table>
<thead>
<tr>
<th><strong>Interface</strong></th>
<th><strong>Purpose</strong></th>
<th><strong>IP Address</strong></th>
<th><strong>Gateway</strong></th>
<th><strong>DNS</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>NIC1</td>
<td>Internet / Management</td>
<td>192.168.1.50</td>
<td>192.168.1.1</td>
<td>8.8.8.8</td>
</tr>
<tr>
<td>NIC2</td>
<td>Imaging Network</td>
<td>192.168.50.1</td>
<td><em>(none)</em></td>
<td><em>(none)</em></td>
</tr>
</tbody>
</table><h3 id="dhcp-scope"><strong>DHCP Scope</strong></h3>
<h3 id="dhcp-scope-configuration">DHCP Scope Configuration</h3>

<table>
<thead>
<tr>
<th><strong>Setting</strong></th>
<th><strong>Value</strong></th>
</tr>
</thead>
<tbody>
<tr>
<td>Subnet</td>
<td>192.168.50.0/24</td>
</tr>
<tr>
<td>IP Range</td>
<td>192.168.50.10 – 192.168.50.200</td>
</tr>
<tr>
<td>Lease Duration</td>
<td>8 days</td>
</tr>
<tr>
<td>Router Option</td>
<td><em>(blank)</em></td>
</tr>
<tr>
<td>DNS Option</td>
<td>192.168.50.1</td>
</tr>
</tbody>
</table><h3 id="useful-commands"><strong>Useful Commands</strong></h3>
<p>`# Verify network configuration<br>
Get-NetIPAddress</p>
<h1 id="verify-dhcp-lease-activity">Verify DHCP lease activity</h1>
<p>Get-DhcpServerv4Lease -ScopeId 192.168.50.0</p>
<h1 id="check-wds-service-status">Check WDS service status</h1>
<p>Get-Service wdsserver</p>
<h1 id="restart-wds-service">Restart WDS service</h1>
<p>Restart-Service wdsserver</p>
<h1 id="list-connected-pxe-clients">List connected PXE clients</h1>
<p>Get-WdsClient`</p>
<hr>

