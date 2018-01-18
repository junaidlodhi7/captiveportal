# Installing/Configuring Openwrt,Wifidog and AuthServer on Openwrt Supported Routers and Authpuppy. 

Junaid Lodhi, Assistant Software Engineer, Virtual Force Pvt. Ltd.


## How Captive Portal  Works
captive portal is a feature which blocks the user from accessing the network until the user is verified and valiidated. we can set up verification process for authenticated users and guest users.Authenticated users are validated against the database of captive portal and then granted access.
No validation is required by guest users to access the network.
There are two sides which make the captive portal work
**OpenWrt enabled Router** and an authorization Server **AuthPuppy**.



## Sample User Story
### For Guest Users
**1** User connects to the network which is open.

**2** User is redirected to captive Portal Page

**3** User Fill in the form and is granted access to the Network

### For verified Users
**1** User connects to the network which is open.

**2** User is redirected to captive Portal Page

**3** User Fill in the form and granted access for **15minutes** to validate through the email which portal the sends

**4** User click on the link to validate and granted access.



## Setting Up Router and Wifidog
we need to upgrade the firmware of router to install wifidog for this purpose we use openwrt linux based firmware.

here is list of routers for which Openwrt firmware is avilable,  [Table of Hardware](https://wiki.openwrt.org/toh/start).

Download the firmware(bin file), goto routers firmware upgrade section and upload the supported bin file for that router.once it is installed the router is restarted and welcomes you with **LUCI**  UI .
Now we are going to use Putty to install wifidog plugin in router.

## Router Set up
Make a new interface
```C++
//Interface details
    Name: abc //in my case
	protocol: static address
    IPv4 address : 192.168.12.1
    IPv4 netmask : 255.255.255.0
    IPv4 broadcast : 192.168.12.255
 //ADVACNCE SETTING
 check bring up on boot o
 check use built-in IPV^ management
 //PHYSICAL SETTINGS
 check wireless Network : Master "your wifi SSID" ("name of this Interface")
  


```

## Setting Up WifiDog
before you start make sure your router LAN port in connected with PC, and router's WAN port from the 
Open Putty Configuration enter the IP of Router with port 22 and SSH,
you will be prompted to enter Username and Password.
after successful login.

**1** ping any ip of check the internet 

**2** write #  opkg update to update package manager

**3** write # opkg install wifidog

**4** now edit wifidog.conf

wifidog.conf is edited to point the router to authserver for authentication when a user try to enter the network.

**5** write # vi /etc/wifidog.conf

wifidog.conf file will open

**6** find Gateway Id and change it.



###### it is important to note that gateway id and the node we are going to creat on authserver must have the same name.

**7** find gateway interface and change it 

###### it is important to note that name of Gateway interface must be same as the Network Interface we will created in Router.

**8** find Authserver and edit info according to your Authserver details 

```C++
Authserver{

hostname: //put the ip here
SSLAvailable no
Http port //default port for authserver in my case It was 80
path /authpuppy/web
loginScriptPathFragment login/?
PortalScriptPathFragment portal/?
msgScriptPathFragment gw_message.php?
PingScriptPathFragment ping/?
AuthScriptPathFragment auth/?
}
```
press escape then :wq to same changes.

**9** command to start wifidog is # wifidog start  ,but it is recommended to use # wifidog -f -d 7 
so you can see the list of errors if in your case its not working.
## Setting Up Authpuppy (AuthServer)

Download AuthPuppy from  [here](https://launchpad.net/authpuppy/+download).
put the folder in server in my case I pasted the folder in htdocs as I used **XAMPP**

Goto lochost/authpuppy/web

follow the instructions install authserver, the process is self-explanatory. In this process you only have to create database and enter credentials in installation process, authpuppy will use migrations to create tables it requires.

once the Authser is installed successfully we have to download and install plugins to make it work.

Now we have installed authpuppy successfully if you are having any issues you may ask/search  [here](https://answers.launchpad.net/)
we will install plugins to make it work.

login with admin credentials and navigate to manage plugins and install **aplocaluser Plugin**
and **Splashonly Plugin**

Once they are installed successfully we have to setup SMTP SERVER details 
as we need to verifiy user as I told above in **For verified Users** Section.

For the first time go through the application flow to register, so that app cache is created once it is done

GOTO this **C:\xampp\htdocs\authpuppy\cache\frontend\prod\config\congif-factories.yml** path and enter the SMTP details, be careful as yml is indentation sensitive.
keep the same indentation as mentioned below.
```C++
mailer:
    param:
      transport:
        class: Swift_SmtpTransport
        param:
          host:       mail.example.com
          port:       465
          encryption: ssl
          username: mail@abc.com
          password: yourpassword

```

