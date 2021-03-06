![logo](https://raw.githubusercontent.com/P0cL4bs/WiFi-Pumpkin/master/icons/logo.png)

WiFi-Pumpkin
---
[![build](https://travis-ci.org/P0cL4bs/WiFi-Pumpkin.svg)](https://travis-ci.org/P0cL4bs/WiFi-Pumpkin/)

Framework for Rogue Wi-Fi Access Point Attack
### Description
WiFi-Pumpkin is an open source security tool that provides the Rogue access point to Man-In-The-Middle and network attacks.
### Installation

- Python 2.7
```sh
 git clone https://github.com/P0cL4bs/WiFi-Pumpkin.git
 cd WiFi-Pumpkin
 ./installer.sh --install
```
or download [.deb](https://github.com/P0cL4bs/WiFi-Pumpkin/releases) file to install
``` sh
sudo dpkg -i wifi-pumpkin-0.8.4-all.deb

```

refer to the wiki for [Installation](https://github.com/P0cL4bs/WiFi-Pumpkin/wiki/Installation)

### Features
* Rogue Wi-Fi Access Point
* Deauth Attack Clients AP 
* Probe Request Monitor
* DHCP Starvation Attack
* Credentials Monitor
* Transparent Proxy
* Windows Update Attack
* Phishing Manager
* Partial Bypass HSTS protocol
* Support beef hook
* ARP Poison 
* DNS Spoof 
* Patch Binaries via MITM
* Karma Attacks (support hostapd-mana)
* LLMNR, NBT-NS and MDNS poisoner (Responder)
* Pumpkin-Proxy (ProxyServer (mitmproxy API))
* Capture images on the fly
* TCP-Proxy


### Plugins
| Plugin | Description | 
|:-----------|:------------|
[dns2proxy](https://github.com/LeonardoNve/dns2proxy) | This tools offer a different features for post-explotation once you change the DNS server to a Victim.
[sslstrip2](https://github.com/LeonardoNve/sslstrip2) | Sslstrip is a MITM tool that implements Moxie Marlinspike's SSL stripping attacks based version fork @LeonardoNve/@xtr4nge.
[sergio-proxy](https://github.com/supernothing/sergio-proxy) | Sergio Proxy (a Super Effective Recorder of Gathered Inputs and Outputs) is an HTTP proxy that was written in Python for the Twisted framework.
[BDFProxy-ng](https://github.com/davinerd/BDFProxy-ng) | Patch Binaries via MITM: BackdoorFactory + mitmProxy, bdfproxy-ng is a fork and review of the original BDFProxy @secretsquirrel.
[Responder](https://github.com/lgandx/Responder) | Responder an LLMNR, NBT-NS and MDNS poisoner. Author: Laurent Gaffie

### Transparent Proxy
![proxy](https://raw.githubusercontent.com/P0cL4bs/WiFi-Pumpkin/master/docs/proxyscenario.png)

 Transparent proxies(mitmproxy) that you can use to intercept and manipulate HTTP traffic modifying requests and responses, that allow to inject javascripts into the targets visited.  You can easily implement a module to inject data into pages creating a python file in directory "plugins/extension/" automatically will be listed on Pumpkin-Proxy tab.
#### Plugins Example Dev

 ``` python
from mitmproxy.models import decoded # for decode content html
from plugins.extension.plugin import PluginTemplate

class Nameplugin(PluginTemplate):
    meta = {
        'Name'      : 'Nameplugin',
        'Version'   : '1.0',
        'Description' : 'Brief description of the new plugin',
        'Author'    : 'by dev'
    }
    def __init__(self):
        for key,value in self.meta.items():
            self.__dict__[key] = value
        # if you want set arguments check refer wiki more info. 
        self.ConfigParser = False # No require arguments 

    def request(self, flow):
        print flow.__dict__
        print flow.request.__dict__ 
        print flow.request.headers.__dict__ # request headers
        host = flow.request.pretty_host # get domain on the fly requests 
        versionH = flow.request.http_version # get http version 
        
        # get redirect domains example
        # pretty_host takes the "Host" header of the request into account,
        if flow.request.pretty_host == "example.org":
            flow.request.host = "mitmproxy.org"
            
        # get all request Header example 
        self.send_output.emit("\n[{}][HTTP REQUEST HEADERS]".format(self.Name))
        for name, valur in flow.request.headers.iteritems():
            self.send_output.emit('{}: {}'.format(name,valur))
            
        print flow.request.method # show method request 
        # the model printer data
        self.send_output.emit('[NamePlugin]:: this is model for save data logging')

    def response(self, flow):
        print flow.__dict__
        print flow.response.__dict__
        print flow.response.headers.__dict__ #convert headers for python dict
        print flow.response.headers['Content-Type'] # get content type
         
        #every HTTP response before it is returned to the client
        with decoded(flow.response):
            print flow.response.content # content html
            flow.response.content.replace('</body>','<h1>injected</h1></body>') # replace content tag 
       
        del flow.response.headers["X-XSS-Protection"] # remove protection Header
        
        flow.response.headers["newheader"] = "foo" # adds a new header
        #and the new header will be added to all responses passing through the proxy
```
#### About plugins
[plugins](https://github.com/P0cL4bs/WiFi-Pumpkin/wiki/Plugins) on the wiki 

#### TCP/UDP Proxy
A proxy that you can place between in a TCP stream. It filters the request and response streams with ([scapy](http://www.secdev.org/projects/scapy/) module) and actively modify packets of a TCP protocol that gets intercepted by WiFi-Pumpkin. this plugin uses modules to view or modify the intercepted data that possibly easiest implementation of a module, just add your custom module on  "plugins/analyzers/" automatically will be listed on TCP/UDP Proxy tab.

``` python
from scapy.all import *
from scapy_http import http # for layer HTTP
from default import PSniffer # base plugin class

class ExamplePlugin(PSniffer):
    _activated     = False
    _instance      = None
    meta = {
        'Name'      : 'Example',
        'Version'   : '1.0',
        'Description' : 'Brief description of the new plugin',
        'Author'    : 'your name',
    }
    def __init__(self):
        for key,value in self.meta.items():
            self.__dict__[key] = value

    @staticmethod
    def getInstance():
        if ExamplePlugin._instance is None:
            ExamplePlugin._instance = ExamplePlugin()
        return ExamplePlugin._instance

    def filterPackets(self,pkt): # (pkt) object in order to modify the data on the fly
        if pkt.haslayer(http.HTTPRequest): # filter only http request 
        
            http_layer = pkt.getlayer(http.HTTPRequest) # get http fields as dict type
            ip_layer = pkt.getlayer(IP)# get ip headers fields as dict type
            
            print http_layer.fields['Method'] # show method http request
            # show all item in Header request http
            for item in http_layer.fields['Headers']:
                print('{} : {}'.format(item,http_layer.fields['Headers'][item]))
            
            print ip_layer.fields['src'] # show source ip address 
            print ip_layer.fields['dst'] # show destiny ip address 
            
            print http_layer # show item type dict
            print ip_layer # show item type dict
            
            return self.output.emit({'name_module':'send output to tab TCP-Proxy'}) 

```
#### About TCP/UDP Proxy
[TCP/UDPProxy](https://github.com/P0cL4bs/WiFi-Pumpkin/wiki/TCP-UDPProxy) on the wiki 

### Screenshots
[Screenshot](https://github.com/P0cL4bs/WiFi-Pumpkin/wiki/Screenshots) on the wiki 

### FAQ
[FAQ](https://github.com/P0cL4bs/WiFi-Pumpkin/wiki/FAQ) on the wiki

### Contact Us
Whether you want to report a [bug](https://github.com/P0cL4bs/WiFi-Pumpkin/issues/new), send a patch or give some suggestions on this project, drop us or open [pull requests](https://github.com/P0cL4bs/WiFi-Pumpkin/pulls) 

### Donate
##### paypal:
[![donate](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=PUPJEGHLJPFQL)

Via BTC: 1HBXz6XX3LcHqUnaca5HRqq6rPUmA3pf6f

Happy MITM!