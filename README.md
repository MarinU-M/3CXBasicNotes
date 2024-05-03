# 3CX Basic Certification
---------------------------------------------

## Configuring a Desktop Phone:

### 3CX supported IP phones:
- Fanvil
- Grandstream
- Htek
- snom
- Yealink

Find full supported IP phone list is [here](https://www.3cx.com/sip-phones/)

### Provisioning Types:
RPS
- The phone is pre-programmed from the factory to connect to the vendor's RPS server and request the provisioning URL. 3CX sends the vendor's RPS server the URL and when a new phone boots up it will obtain the URL from the RPS server.

DHCP option 66
- For larger networks with manageable DHCP servers, you can enter the base URL in DHCP Option 66. A phone will query this field when it obtains its IP and if a URL is specified it will go there by adding its MAC address.

PnP (Plug and Play)
- Can be used for 3CX in the cloud (phones behind an SBC)
	1. 3CX on-premise - Phones on local LAN
- Fast and simple
- Admin driven
- Based on multicast messages
	1. When the phone plugged in, it announces that it was been plugged in, PBX receives the announcement, admin approves and assigns desktop phone.
	2. The multicast ends at the router/border devices

More info is [here](https://www.3cx.com/docs/manual/ip-phones/)

BLF (Busy Lamp Field)
- IP Phones
- monitors and extensions call state
	1. Idle
	2. Ringing
	3. In Call
- Used for
	1. Pickups
	2. Transfers
	3. Speed Dial
- More Options
	1. Lines Keys
	2. Profile state cnahe
	3. queue state change
	4. Speed dials
	5. Custom speed dials
	6. Shared parking

RPS (Direct SIP/Stun)
- Phones that are not behind a 3CX SBC, must be configured with a 3CX RPS server.
- In the 3CX management console, when you're adding a new phone, need to select **Direct SIP (STUN - remote)** in the options, and the provisioning link

Legacy phones: Cisco, Polycom, and Aastra
- Need to wipe the phone's old firmware and install new 3CX firmware on it
- In the management console, add a new phone, select the phone model and provisioning type
- ***Not supported anymore, not recommended***


3CX Web Client
- Browser-based, no software installation required
	1. Prerequisites(browser): Chrome, Firefox, Edge
	2. Prerequisites(platform); Windows, Mac
- Can run alongside IP phones on the same extention number by **SIP Forking**
- Can control IP phone with mouse
	1. Dial 
	2. Pickup
	3. Transfer
	4. Conferencing
	5. Recording Start/Stop
- Can send Welcome email manually by admin or automatically when the extention is made
	1. 3cxconfig file or QR code
	2. need to make sure the user install it

3CX APP client
- Windows 10 or higher
- Mac 10.10 or higher, 10.13 is prefered (as of the time of lockdown)
- iOS 13.0 or higher
- Android 7.X or higher

---------------------------------------------------

## Configuring the Firewall:
- Need to setup the firewall because the voice call needs to pass through it to reach your PBX.
- Voice calls use several ports because they use SIP and RTP.
- Some configurable firewalls
	1. pfsense
	2. Watchgurad
	3. Sonicwall
- **3CX WILL NOT CONFIGURE YOUR FIREWALL**

Firewall ports to be opened
Purpose | Port | In or Out | Desc.
--- | --- | --- | --- 
SIP Trunk / VOIP Provider|5090 udp|Inbound|SIP communications
--- | --- | --- | ---
- |any udp|Inbound|RTP
--- | --- | --- | --- 
remote 3CX apps|5090 udp/tcp|Inbound|the 3CX tunnel
--- | --- | --- | --- 
- |443 or 5001 tcp|Inbound|Presence and Provisioning
--- | --- | --- | --- 
- |443 tcp|Outbound|Google Android PUSH
--- | --- | --- | --- 
- |2195, 2196 tcp|Outbound|Apple iOS PUSH
--- | --- | --- | --- 
Remote IP Phones / Bridges via Direct SIP|5060 udp/tcp|Inbound| -
--- | --- | --- | --- 
- |5061 tcp|Inbound|(If using secure SIP)
--- | --- | --- | --- 
- |any udp|Inbound|RTP
--- | --- | --- | --- 
- |443 or 5001 tcp|Inbound|provisioning
--- | --- | --- | --- 
3CX WebMeeting, SMTP ans Activations|443 tcp|Outbound|webmeeting.3cx.net
--- | --- | --- | --- 
- |443 or 5001 tcp|Inbound|to notify users of incoming web meetings
--- | --- | --- | --- 
- |2528 tcp|Outbound|tosend emails using 3CX SMTP
--- | --- | --- | --- 
Remote Configuration Wizard|5014 tcp|Inbound|HTTP

Also, you need to **disable SIP ALG** (Application Layer Gateway)

Full ports used my 3CX:
	https://www.3cx.com/docs/ports/

Some support help and what they offer:
	https://www.3cx.com/ordering/procedures/


NAT and Ports
NAT translates public (IPv4) addresses to Private IP Addresses (NAT/PAT)
Allows Access or restricts it (ACLs)
Required to allow connections to 3CX from
	Providers
	3CX apps
	Supported IP Phones
	3CX SBC and Brides
IPv6 is also supported

3CX PBX requires Full Cone NAT, this allows incoming traffic from uncontacted sources

Port Preservations
This makes sure that the internal source port and the external source port stay the same
	Ensures connectivity
	Eliminates Destination Implementations


SIP ALG (Application Layer Gateway)
This inspects the contents of the packet and tampers with it uncontrollably
Made for clients not severs
This is why it is recommended to DISABLE SIP ALG

How the firewall checker works:
https://www.3cx.com/docs/firewall-checker/


Main things firewall checker checks
	Full come NAT has been correctly configured
	Port Preservation has been correctly configured and tested
	SIP ALG is disabled and tested 


---------------------------------------------------

Configuring VoIP Providers / SIP Trunks:

SIP Trunks (VoIP Providers)
	Supported providers have:
		Passed IOT
		Have an easy setup
		Support from 3CX Support Team

	Generic VoIP Providers
		Not tested
		Not easy to setup
		No 3CX support available

SIP Trunking provides access to the Public Switched Telephone Network (PSTN)
Transport via Existing Internat Lines, and there is no additional hardware required


Inbound Rules

Inbound rules

These rules process and route the incoming calls to the right destination like if they go to voicemail, an extension, or a queue

Making DID rules
management console -> Add DID Rule
Need to decide where the DID calls go like
	End call
	Extension
	Voicemail box for extension
	Forward to Outside Number
	Send fax or email of extension

You can also specify hours and dates that the DID numbers can work on

Caller ID Rule
Can make a rule based on the caller ID that is calling you
	The same as DID rules

DID

Routes incoming calls to a destination based on the Dialed Number
This bypasses the receptionist and goes directly to the person they are trying to reach
These are virtual numbers assigned directly to the customer physical line

	Applies to
		Extensions
		Voicemail of an extension
		Forward to outside number
		Fax Service

Can also route to a different destination based on
	Date
	Time of day

Can add all this in the management console -> inbound rules

Outbound Rules
These dictate how 3CX routes outgoing calls.
You need to create at least one outbound rule to start using 3CX 

Allow or restrict called numbers
	Destination number
	originating internal extension
	dialed number length

Can manage all this in the management console -> outbound rules
More info at 
https://www.3cx.com/docs/manual/sip-trunks/


Emergency numbers
Make sure you need to know all of your emergency numbers
Add all of the numbers and set everything up in the management console -> settings -> PBX -> emergency numbers

Outbound calls drop after 32 seconds by default

VoIP consumes bandwidth, and it is the rule of thumb that each VoIP call uses about 30 - 120 kb per second

Firewall checks:
Go to the management console -> PBX status -> Firewall -> Run
This will run the firewall check and tell you if it passed or not




Holiday Hours Rules
You can make inbound rules based on the date it is, so you can make holiday hours or days rules where all calls will be redirected straight to voicemail
Managment console -> Settings -> Timezone, Office Hours & Holidays



---------------------------------------------------


Digital Receptionists:

These are the menus that you hear when you call a business
They create a voice prompt
Prompt user for input
Predefined Transfer Options based on input
CAn call any extension
Use WAV file format PCM 8 kHz 16bit Mono

Creating digial receptionist
managment console -> Digital receptionist -> add
Here you can add all the prompts and audio files that the receptionist will use

receptionist options
	Connect to extension
	Connect to Ring Group
	Connect to Queue
	Connect to Digital Receptionist
		Multi-layer voice menues
	Transfer to voice mail
	Nothing / repeat prompt

Call by name
	Call an extension based on their last name
	Smart dialing
This requires self-identification prompts that you have to make
	
If there is more than 1 match, it plays the self ID for each match and the user selects

Direct dialing
	When you can call an extension without a menu option
	The interdigit timeout is 3 seconds
Can be disabled globally for all digital receptionists

No input or invalid input
No input
	timeout
	Goes back to normal menu options

Invalid Input
Can transfer to
	Extension
	Ring group
	Queue
	IVR/Digital receptionist
	Repeat Prompt

HOL digital receptionist (Holiday receptionist)
Can make a separate digital receptionist for holidays

MS Exchange Server with 3CX
https://www.3cx.com/docs/exchange-server-2013-config/


---------------------------------------------------


Installing:

Can be installed on almost anything including Windows (Not XP), Linux, Apple, local VM
Use PBXExpress for Amazon AWS, Amazon Lightsail, Google Cloud, Microsoft Azure, OVH, OpenStack API v2

Hardware Requirements:
https://www.3cx.com/docs/recommended-hardware-specifications-for-3cx/

License Keys:
3 Types
	Standard (Queues, Bridges, reports, 3CX FQDN and SMTP only)
	Pro (Adv. Reports & Queues, CRM, Hotel)
	Enterprise (Adv. Recording Control, Customisation of Phone Logo, Standby Licence, Failover)
More detail on the editions:
https://www.3cx.com/phone-system/edition-comparison/

3CX Domain Service
Free public FQDN
Free trusted SSL certificate
Support for Dynamic IPs

PBX Ports
Web Server
	HTTP - Default 5000 (Only local LAN - Based on RFC)
	HTTPS - Default 5001 (Local LAN and WAN)
SIP Server - Default 5060
Tunnel - Default 5090
CAN ONLY CONFIGURE PORTS DURING INSTALL

Internal FQDN or IP?
FQDN
	Allows split DNS setups
	Allows failover setup
	Configurable internal DNS server
	Configurable internal DHCP server

If there is nothing to manage, install on IP

Admin email Receives:
	Update Information
	Backup & restore progress
	info and warnings

Mail server (3CX SMTP, Custom SMTP or mail provider) sends:
	Welcome Email
	Faxes
	Voicemails
	Missed call notifications


---------------------------------------------------

Operating Systems:

Windows 10, 8.1, 8, 7 SP1
Mac OS 10.13, 10.12, 10.11, 10.10
Apple IOS, iOS 10.0 or higher
Android version 4.X or higher

Provisioning
It is a prefilled config used to automatically configure the device
This is great because it is Vendor Specific Syntax, Removes Human Error, and Boosts Deployment Speed

Email Provisioning
This is great for remote provisioning
Works by Admin sends a welcome email with the desired 3CX config file, the user opens the email and file, 3CX app automatically imports the config

QR Code Provisioning
This is used to provide mobile apps on phones

If you are using mobile or outside of local LAN, you need to open NAT ports on the WAN
Open NAT ports WAN - 3CX PBX
	Tunnel Port - TCP/UDP (Default 5090)
	HTTPS Port - TCP/UDP (Default 5001)

PUSH on mobile
This allows for the APP to hibernate
	No battery consumption
	APP wakes up on incoming call

Flow:
Incoming call -> 3CX -> Vendor PUSH Service -> Mobile Device -> APP

Windows APP
Can be used in either "Softphone Mode" or "CTI Mode"
Softphone Mode
	Make and receive calls on my computer without a desk phone
CTI Mode
	A desk phone is required for the app to manage your calls from the desktop

The interface is the same for the Mac app and the same for android and apple ios

Setting up PUSH on mobile devices
https://www.3cx.com/docs/firebase-android-client/
Google Cloud Messaging is being discontinued, need to use Firebase now
Log into firebase with your google account, go to cloud messaging and copy the Server Key and Sender ID
Log into your 3CX management console, go to Settings -> 3CX Client, and put in the server key and sender ID here, Click OK to save


---------------------------------------------------


Voicemail:

Voicemail is enabled by default for all 3CX extensions

Voicemail notifications
	Notification Only
	Notification with WAV file attached
	Notification with WAV file attached + Delete voicemail from PBX

Pre-Recorded files are WAV format 8kHz 16bit Mono
They are recorded from phone
Can have different greetings depending on the forwarding rule

Voicemail transcription
Translates speech to text
Uses Microsoft Bing Speech API or GCloud Speech API
Requires PRO or Enterprise license
	Transcribed messages can be views in email or 3CX web client

CAn assign voicemail quote
	Assign quota (disk space)
	email when nearing quota (%)
	auto delete old voicemails (days)

Voicemail can be disabled
	Has a 4 digit PIn by default (Can increase to 10 digit pin)
	3 failed attempt blocks you out for 2 mins

Voicemail is not backed up by default
	Use send an email and delete option to reduce backup size

Change voicemail setting in
Managemnt console -> Settings -> greetings

You can also check your voicemail if you have an IP phone
Here are the supported phones
	https://www.3cx.com/user-manual/ip-phone-voicemail-conference-calls/


Group voicemail
Need to create a ring group that the voicemails will be forwarded to


Voicemail transcription (Speech to text)
You need to first get your API key from your provider, either that be Microsoft Bing Speech API or GCloud speech API
Need to secure the API key by restricting it to an IP for an IP phone

Enable Voicemail transcription in 3CX
Managment console -> Settings -> Voicemail and enable voicemail transcription
Enter the API key you have copied.
