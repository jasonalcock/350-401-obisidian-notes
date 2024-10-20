---
aliases: 
publish: 
---
%%
date:: [[2024-09-14]]
parent:: 
%%
# [[NETCONF]]

```table-of-contents
```

#### Basic Info
- Uses SSH for transport, TCP 830 by default 
- RPC for messaging which is actions based
- Has a bunch of control plane verb like operations you can perform e.g.:
	- get edit copy or delete config, lock or unlock, commit 
- The rpc, netconf operations and YANG data are all written in xml format.
- Netconf needs a username and to enable login and exec level auth
```
username admin privilege 15 password C1sco12345
aaa new-model
aaa authentication login default local
aaa authorization exec default local
```
- Enabled netconf on a router and commands:
```
#netconf-yang
! enable the use of the candidate datastore
#netconf-yang feature candidate-datastore
Cat8000V#show netconf-yang status
netconf-yang: enabled
netconf-yang candidate-datastore: enabled
netconf-yang side-effect-sync: enabled
netconf-yang ssh port: 830
netconf-yang turbocli: disabled
netconf-yang ssh hostkey algorithms: rsa-sha2-256,rsa-sha2-512,ssh-rsa
netconf-yang ssh encryption algorithms: aes128-ctr,aes192-ctr,aes256-ctr,aes128-cbc,aes256-cbc
netconf-yang ssh MAC algorithms: hmac-sha2-256,hmac-sha2-512,hmac-sha1
netconf-yang ssh KEX algorithms: diffie-hellman-group14-sha1,diffie-hellman-group14-sha256,ecdh-sha2-nistp256,ecdh-sha2-nistp384,ecdh-sha2-nistp521,diffie-hellman-group16-sha512
```

#### The Candidate Datastore Order Of Operations
- The benefit of using a candidate is so you can stage changes, edit and view before committing them then also rollback any changes.
- If you enable candidate store the router will transition from running to a candidate datastore and kick off any current users. 
- The running datastore is not writable through NETCONF sessions so you have write to the candidate.
- The candidate datastore is shared by anyone connecting so you need to lock it, mess with it then after it has been committed unlock it.
- You could copy the running to the candidate but generally you'll just edit the bits you need.
- The commits will only overwrite the sections of code that you are working with.
#### The Candidate Datastore Order Of Operations
- Lock running (prevents others users editing datastore and ensures the config your working on is not changed)
- Lock candidate
- Edit candidate
- You can discard and revert all changes back to running config
- Commit candidate to running
	- Note you can do a commit confirmed that will rollback a commit if a commit is not sent within 10 minutes 
- Unlock running and candidate datastores

#### Simple Get NETCONF Config Example
 - Launch an always on cat8k sandbox from Cisco DevNet
 - Here's the XML code that contains all you need to pull an ios-xe devices configuration using netconf. 
```xml
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="101">
  <get-config>
    <source>
      <running/>
    </source>
    <filter>
      <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
        <hostname>test</hostname>
      </native>
    </filter>
  </get-config>
</rpc>
```

- The XML code above is using multiple schemas
	- The NETCONF protocol defines elements like rpc, get-config
	- YANG models define the vendor specific configs e.g. native
- xmlns is the xml namespace. See here for more [[XML]] info.
- The rpc urn is defined here  [yang/standard/ietf/RFC/ietf-netconf@2011-06-01.yang at main · YangModels/yang · GitHub](https://github.com/YangModels/yang/blob/main/standard/ietf/RFC/ietf-netconf%402011-06-01.yang)
 
```bash
brew install netconf-console
``` 

 ```bash
 netconf-console2 --host=devnetsandboxiosxe.cisco.com --port=830 -u admin -p C1sco12345 <xml-file.xml>
```
 
 - Or use python and ncclient with the XML contained within the script:
  
```python
from ncclient import manager

# Define device credentials
device = {
    "host": "your_router_ip",  # Replace with your router's IP address
    "port": 830,               # NETCONF default port
    "username": "your_username",  # Replace with your username
    "password": "your_password",  # Replace with your password
    "hostkey_verify": False     # Skip host key verification for simplicity
}

# NETCONF filter to specify which part of the configuration you want to retrieve
netconf_filter = """
<filter>
  <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native"/>
</filter>
"""

# Connect to the device and get the running config
with manager.connect(**device, device_params={'name': 'iosxe'}, allow_agent=False, look_for_keys=False) as m:
    # Send the <get-config> RPC to retrieve the running configuration
    running_config = m.get_config('running', filter=netconf_filter).data_xml

    # Print the running configuration
    print(running_config)

```

#### Create Loopback Interface with NETCONF

Using this script grab a copy of your devices current netconf configuration.
```python
import os
import xml.dom.minidom
import xmltodict
import json
from ncclient import manager

# Device credentials
device = {
    "host": "devnetsandboxiosxe.cisco.com",
    "port": 830,
    "username": "admin",
    "password": "C1sco12345",
    "hostkey_verify": False
}

def get_next_versioned_filename(filename, extension):
    version = 1
    new_filename = f"{filename}_v{version}.{extension}"
    while os.path.exists(new_filename):
        version += 1
        new_filename = f"{filename}_v{version}.{extension}"
    return new_filename

# Connect to the device and retrieve the running config
with manager.connect(**device, device_params={'name': 'iosxe'}, allow_agent=False, look_for_keys=False) as m:
    # Get the running configuration as XML
    running_config_xml = m.get_config('running').data_xml

    # Pretty print the XML output
    xml_dom = xml.dom.minidom.parseString(running_config_xml)
    pretty_xml = xml_dom.toprettyxml(indent="    ")  # Indent the XML for readability
    print("Prettified XML output:")
    print(pretty_xml)

    # Get the next available filename for XML
    xml_filename = get_next_versioned_filename("running_config", "xml")
    
    # Save the prettified XML to a file
    with open(xml_filename, "w") as xml_file:
        xml_file.write(pretty_xml)
    print(f"\nXML configuration saved to {xml_filename}")

    # Convert the XML to a Python dictionary
    running_config_dict = xmltodict.parse(running_config_xml)

    # Convert the dictionary to a JSON formatted string
    running_config_json = json.dumps(running_config_dict, indent=4)
    print("\nJSON formatted output:")
    print(running_config_json)

    # Get the next available filename for JSON
    json_filename = get_next_versioned_filename("running_config", "json")
    
    # Save the JSON output to a file
    with open(json_filename, "w") as json_file:
        json_file.write(running_config_json)
    print(f"\nJSON configuration saved to {json_filename}")>)
```

- Create a loopback or whatever config change you need then run the above script again so you can diff the netconf config and learn what's changed.
- If you added a loopback interface 101 this is what you'll see:
```xml
<interfaces xmlns="urn:ietf:params:xml:ns:yang:ietf-interfaces">
	<interface>
		<name>Loopback101</name>
		<description>MY_SIMPLE_PY_RESTCONF_CHANGE</description>
		<type xmlns:ianaift="urn:ietf:params:xml:ns:yang:iana-if-type">ianaift:softwareLoopback</type>
		<enabled>true</enabled>
		<ipv4 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip"/>
		<ipv6 xmlns="urn:ietf:params:xml:ns:yang:ietf-ip"/>
	</interface>
</interfaces>
```

- Wrap the above xml in a root tag of `<config>` then save it to `config_change.xml`
- Then use the following python to save the config to a device
```python
from ncclient import manager

# Define device credentials
device = {
    "host": "devnetsandboxiosxe.cisco.com",  # Replace with your actual device's IP or hostname
    "port": 830,               # NETCONF default port
    "username": "admin",
    "password": "C1sco12345",
    "hostkey_verify": False
}

# Load configuration from an external XML file
def load_config_from_file(xml_file):
    with open(xml_file, 'r') as file:
        return file.read()

# Function to apply configuration to the device using NETCONF
def apply_config_to_running_datastore(xml_config):
    with manager.connect(**device, device_params={'name': 'iosxe'}, allow_agent=False, look_for_keys=False) as m:
        # Send the %3Cedit-config%3E RPC with the configuration to the running datastore
        response = m.edit_config(target="running", config=xml_config)
        return response

# Main program
if __name__ == "__main__":
    # Path to your configuration XML file
    xml_file_path = "nc_conf_change.xml"  # Replace with the path to your XML file

    # Load configuration from the XML file
    config_xml = load_config_from_file(xml_file_path)
    
    # Apply the configuration to the running datastore
    result = apply_config_to_running_datastore(config_xml)

    # Print the NETCONF response
    print(result)
```

#### Stage and Commit Configurations
- Ensure you have enabled  `#netconf-yaml feature candidate`
- You should be able to verify the capability is on using the checking the devices capability list for urn:ietf:params:netconf:capability:candidate:1.0
- Here's a script to get a capability list:
```python
"""devices = {
    "Pod00-CSRv-XE": {"ip": "10.2.100.11", "port": "830", "platform": "csr",},
    "Pod00-XRv-XR":  {"ip": "10.2.100.12", "port": "830", "platform": "iosxr",},
    "Pod00-N9Kv-NX": {"ip": "10.2.100.13", "port": "830", "platform": "nexus",},
    }"""
from ncclient import manager
from prettytable import PrettyTable

# Device credentials
devices = {
    "devnetsandboxiosxe.cisco.com": {"ip": "devnetsandboxiosxe.cisco.com", "port": "830", "platform": "iosxr"}
}

for device, properties in devices.items():
    ip = properties["ip"]
    port = properties["port"]
    platform = properties["platform"]
    print(f"Connecting to device: {device}")

    # Context manager keeping ncclient connection alive for the duration of the context.
    with manager.connect(
        host=ip,                            # IP address of device
        port=port,                          # Port to connect to
        username='admin',                   # SSH Username
        password='C1sco12345',              # SSH Password
        hostkey_verify=False,               # Allow unknown hostkeys not in local store
        device_params={'name': platform}    # Device connection parameters
    ) as m:  
        
        capabilities = []
        
        for capability in m.server_capabilities:
            capabilities.append(capability)

        # Sort the list alphabetically
        capabilities = sorted(capabilities)

        devices[device]["capabilities_long"] = []
        devices[device]["capabilities_short"] = []

        # Build a table to pretty print the capabilities short names.
        table_short = PrettyTable()
        table_short.field_names = [device + ' Capabilities (Short)']
        table_short.align[device + ' Capabilities (Short)'] = 'l'
        table_short.padding_width = 0  # Remove excessive padding

        for capability in capabilities:
            short_capability = capability.split('?')[0]
            devices[device]["capabilities_short"].append(short_capability)
            table_short.add_row([short_capability])

        # Print the short capabilities table.
        print(table_short)

        # Build a table to pretty print the capabilities long names.
        table_long = PrettyTable()
        table_long.field_names = [device + ' Capabilities (Long)']
        table_long.align[device + ' Capabilities (Long)'] = 'l'
        table_long.padding_width = 0  # Remove excessive padding

        for capability in capabilities:
            devices[device]["capabilities_long"].append(capability)
            table_long.add_row([capability])

        # Print the long capabilities table.
        # print(table_long)
```


##### YANG Yet Another Next Generation
- Written in Custom Modelling Language
- Like json, xml or mibs it is structured
	- Containers, Modules, Leafs and supports several data types

#### Netconf Operations
- The `<get>` operation gets the configuration and operational data in a datastore.
	- `<filter>` is a parameter of the `<get>` operation to select a particular subtree in the reply and “type” is the attribute of the parameter.
- The `<get-config>` operation gets only the configuration data in a datastore.
	-  `<filter>` is a parameter of the `<get>` operation to select a particular subtree in the reply and “type” is the attribute of the parameter.
- `<close-session>` Operation – Polite way of disconnecting
- `<kill-session>` Operation – Not so polite way of disconnecting another session. – Releases any locks, aborts any confirmed commit related to session
- `<delete-config>` Operation – Delete a complete data store (not running).
- `<commit>` copy configuration in the `<candidate/>` datastore to the `<running/>` configuration datastore.
- `<edit-config>` Loads all or part of a configuration to the specified configuration data store.
- `<copy-config>` Replace an entire configuration data store with another
- `<lock>` Lock or unlock the entire configuration data store system
- `<validate>` Operation – To validate (check for syntactical and semantic errors) the content of a datastore.

#### XML examples

##### Save run to start
```xml
<?xml version="1.0" encoding="utf-8"?>
<rpc xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" message-id="">
  <cisco-ia:save-config xmlns:cisco-ia="http://cisco.com/yang/cisco-ia"/>
</rpc>
```

##### Lock session to prevent many users editing simultaneously 
```
<rpc message-id="101"
          xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
       <lock>
         <target>
           <running/>
         </target>
       </lock>
</rpc>
```

#### Convert IOS CLI to XML
```
show running-config | format netconf-xml
```

```
R5#sh run | format
<?xml version="1.0" encoding="UTF-8"?><Device-Configuration xmlns="urn:cisco:xml-pi">
<version><Param>15.9</Param></version>
<service><timestamps><debug><datetime><msec/></datetime></debug></timestamps></service>
<service><timestamps><log><datetime><msec/></datetime></log></timestamps></service>
<service operation="delete" ><password-encryption/></service>
<hostname><SystemNetworkName>R5</SystemNetworkName></hostname>
<boot-start-marker></boot-start-marker>
<boot-end-marker></boot-end-marker>
<aaa operation="delete" ><new-model/></aaa>
<mmi><polling-interval><MmiPollingIntervalSecond>60</MmiPollingIntervalSecond></polling-interval></mmi>
<mmi operation="delete" ><auto-configure/></mmi>
<mmi operation="delete" ><pvc/></mmi>
<mmi><snmp-timeout><SNMPTimeoutValueSecond>180</SNMPTimeoutValueSecond></snmp-timeout></mmi>
<ip><cef/></ip>
<ipv6 operation="delete" ><cef/></ipv6>
<multilink><bundle-name><authenticated/></bundle-name></multilink>
<redundancy>
<ConfigRed-Configuration>
</ConfigRed-Configuration>
</redundancy>
<track><TrackedObject>1</TrackedObject><ip><route><IPPrefixPrefixMask>3.3.3.3 255.255.255.255</IPPrefixPrefixMask><reachability>
<ConfigTrack-Configuration>
</ConfigTrack-Configuration>
</reachability>
</route>
</ip>
</track>
<track><TrackedObject>2</TrackedObject><ip><route><IPPrefixPrefixMask>1.2.3.4 255.255.255.255</IPPrefixPrefixMask><reachability>
<ConfigTrack-Configuration>
</ConfigTrack-Configuration>
</reachability>
</route>
</ip>
</track>
<track><TrackedObject>3</TrackedObject><ip><route><IPPrefixPrefixMask>10.0.20.2 255.255.255.255</IPPrefixPrefixMask><reachability>
<ConfigTrack-Configuration>
</ConfigTrack-Configuration>
</reachability>
</route>
</ip>
</track>
<track><TrackedObject>4</TrackedObject><ip><route><IPPrefixPrefixMask>10.0.20.0 255.255.255.0</IPPrefixPrefixMask><reachability>
<ConfigTrack-Configuration>
</ConfigTrack-Configuration>
</reachability>
</route>
</ip>
</track>
<track><TrackedObject>5</TrackedObject><list><boolean><and>
<ConfigTrack-Configuration>
<object><TrackedObjectNumber>1</TrackedObjectNumber></object>
<object><TrackedObjectNumber>2</TrackedObjectNumber></object>
</ConfigTrack-Configuration>
</and>
</boolean>
</list>
</track>
<track><TrackedObject>6</TrackedObject><list><boolean><or>
<ConfigTrack-Configuration>
<object><TrackedObjectNumber>3</TrackedObjectNumber></object>
<object><TrackedObjectNumber>4</TrackedObjectNumber></object>
</ConfigTrack-Configuration>
</or>
</boolean>
</list>
</track>
<track><TrackedObject>7</TrackedObject><ip><sla><EntryNumber>1</EntryNumber></sla></ip></track>
<track><TrackedObject>8</TrackedObject><interface><Param>GigabitEthernet0/0</Param><line-protocol>
<ConfigTrack-Configuration>
</ConfigTrack-Configuration>
</line-protocol>
</interface>
</track>
<interface><Param>GigabitEthernet0/0</Param>
<ConfigIf-Configuration>
<ip operation="delete" ><address/></ip>
<shutdown></shutdown>
<duplex><auto/></duplex>
<speed><auto/></speed>
<media-type><rj45/></media-type>
</ConfigIf-Configuration>
</interface>
<interface><Param>GigabitEthernet0/1</Param>
<ConfigIf-Configuration>
<mac-address><MACAddress>0000.0000.0005</MACAddress></mac-address>
<ip><address><IPAddress>10.10.10.5</IPAddress><IPSubnetMask>255.255.255.0</IPSubnetMask></address></ip>
<glbp><GroupNumber>1</GroupNumber><ip><VirtualIPAddress>10.10.10.254</VirtualIPAddress></ip></glbp>
<glbp><GroupNumber>1</GroupNumber><priority><PriorityValue>105</PriorityValue></priority></glbp>
<glbp><GroupNumber>1</GroupNumber><load-balancing><weighted/></load-balancing></glbp>
<duplex><auto/></duplex>
<speed><auto/></speed>
<media-type><rj45/></media-type>
<vrrp><GroupNumber>1</GroupNumber><ip><VRRPGroupIPAddress>10.10.10.150</VRRPGroupIPAddress></ip></vrrp>
</ConfigIf-Configuration>
</interface>
<interface><Param>GigabitEthernet0/2</Param>
<ConfigIf-Configuration>
<ip operation="delete" ><address/></ip>
<shutdown></shutdown>
<duplex><auto/></duplex>
<speed><auto/></speed>
<media-type><rj45/></media-type>
</ConfigIf-Configuration>
</interface>
<interface><Param>GigabitEthernet0/3</Param>
<ConfigIf-Configuration>
<ip operation="delete" ><address/></ip>
<shutdown></shutdown>
<duplex><auto/></duplex>
<speed><auto/></speed>
<media-type><rj45/></media-type>
</ConfigIf-Configuration>
</interface>
<ip><forward-protocol><nd/></forward-protocol></ip>
<ip operation="delete" ><http><server/></http></ip>
<ip operation="delete" ><http><secure-server/></http></ip>
<ip><sla><EntryNumber>1</EntryNumber>
<ConfigIPSlaEcho-Configuration>
<X-Interface> icmp-echo 1.1.1.1</X-Interface>
</ConfigIPSlaEcho-Configuration>
</sla>
</ip>
<ipv6><ioam><timestamp/></ioam></ipv6>
<control-plane>
<ConfigCp-Configuration>
</ConfigCp-Configuration>
</control-plane>
<banner><exec><CBannerTextCDelimitingCharacter>^C</CBannerTextCDelimitingCharacter></exec></banner>
.........
<line><LineNumber>con 0</LineNumber>
<ConfigLine-Configuration>
</ConfigLine-Configuration>
</line>
<line><LineNumber>aux 0</LineNumber>
<ConfigLine-Configuration>
</ConfigLine-Configuration>
</line>
<line><LineNumber>vty 0 4</LineNumber>
<ConfigLine-Configuration>
<X-Interface> login</X-Interface>
<X-Interface> transport input none</X-Interface>
</ConfigLine-Configuration>
</line>
<scheduler operation="delete" ><allocate/></scheduler>
<end></end>
</Device-Configuration>
```
#### Comments
- You can go in manually via ssh and talk netconf to an IOS xe device but this is only useful exercise if you are learning about NETCONF for the 1st time.
 - Cisco has a Yangsuite tool that can generate the netconf xml so use it!
- The Cisco yangsuite docker container will generate the configuration change xml or help you locate the namespaces for getting device stats back.
- Pyang is a handy CLI tool for printing out the tree structure of a yang models so you can find the right container for a config change or to to collect device information from. e.g.


```module: Cisco-IOS-XE-native
  +--rw native
     +--rw default
     |  +--rw crypto
     |     +--rw ikev2
     |        +--rw proposal?   empty
     |        +--rw policy?     empty
     +--rw bfd
     +--rw version?                 string
     +--rw stackwise-virtual!
     +--rw boot-start-marker?       empty
     +--rw boot
     |  +--rw system
     |     +--rw tftp-path?   string
     |     +--rw tftp?        string
     |     +--rw bootfile
     |     |  +--rw filename-list* [filename]
     |     |     +--rw filename    string
     |     +--rw flash
     |        +--rw flash-list* [flash-leaf]
     |           +--rw flash-leaf    string
     +--rw boot-end-marker?         empty
     +--rw captive-portal-bypass?   empty
     +--rw memory
     |  +--rw statistics
     |  |  +--rw history
     |  |     +--rw table?   uint8
     |  +--rw chunk
     |  |  +--rw siblings
     |  |     +--rw threshold?   uint32
     |  +--rw free
     |  |  +--rw low-watermark
     |  |     +--rw IO?          uint32
     |  |     +--rw processor?   uint32
     |  +--rw lite?         empty
     |  +--rw reserve
     |  |  +--rw critical!
     |  |     +--rw memory-range?   uint32
     |  +--rw sanity!
     |     +--rw all?      empty
     |     +--rw buffer?   empty
     |     +--rw queue?    empty
     +--rw location
     |  +--rw civic-location
     |     +--rw identifier* [identifier]
     |        +--rw identifier    string
     |        +--rw building?     string
     |        +--rw floor?        string
     |        +--rw landmark?     string
     |        +--rw name?         string
     |        +--rw number?       string
     +--rw call-home!
     +--rw hw-module
     |  +--rw uplink
     |  |  +--rw select?   string
     |  |  +--rw mode?     enumeration
 ```


# References
- [[XML]]
- [Sandbox - Cisco DevNet](https://developer.cisco.com/site/sandbox/)
- [GitHub - CiscoDevNet/yangsuite: Cisco YANG Suite provides a set of tools and plugins to learn, test, and adopt YANG programmable interfaces such as NETCONF, RESTCONF, gNMI and more.](https://github.com/CiscoDevNet/yangsuite)
- [YANG Suite Docs- Cisco DevNet](https://developer.cisco.com/docs/yangsuite/welcome-to-cisco-yang-suite/)
- [using the cli NETCONF console ](https://netdevops.me/2020/netconf-console-in-a-docker-container/)
- [If you need to understand yang structure more see here.](http://yang.ciscolive.com/pod0/labs/lab2/lab2-m1)[LTRSDN-2260](http://yang.ciscolive.com/pod0/labs/lab2/lab2-m1)
- [IETF  YangModels Repo](https://github.com/YangModels/yang/tree/main/vendor/cisco)
- [Kirk Byers netconf candidate example](https://pynet.twb-tech.com/blog/netconf/iosxe-candidate-cfg1.html)[IOS-XE and NETCONF Candidate Configuration Testing, Part1](https://pynet.twb-tech.com/blog/netconf/iosxe-candidate-cfg1.html)
- [Cisco Live Intro the ncclient LTRSDN-2260](http://yang.ciscolive.com/pod0/labs/lab4/lab4-m1)
- [Cisco DevNet Learning Labs Center for Netconf and YANG](https://developer.cisco.com/learning/modules/intro-device-level-interfaces/)
- [Programmability Configuration Guide, Cisco IOS XE Dublin 17.12.x - EEM Python Module \[Cisco IOS XE 17\] - Cisco](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/1712/b_1712_programmability_cg/m_1712_prog_eem_python.html)