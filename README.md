# Firewall-Policy-Like BIG-IP Forward Proxy Implementation (PoC code)
This is a SAMPLE configuration for the BIG-IP Forward Proxy topology with the concept of access control like Firewall-Policy.  
It is intended for PoC (Proof of Concept) and is NOT intended for production use.

**This PoC code is very dangerous because it does not adequately implement error handling.**


## Function
* Allow and deny actions corresponding to the policy order  

* Choosing URL pattern method  

* Switching User Authentication & No Authentication  

* Switching SSL Intercept & Bypass  


## Not Supported
* Destination field is Only FQDNs and Address are supported.  
  Subnet (e.g. 192.168.0.0/24) is not supported.
  URL (e.g. http://www.samplecorp.test/path/index.html) is not supported.

* Control by Authenticated User/Group is not supported.  
  For example, you can authenticate user and group, but access to the sales system (e.g. www.sale-system.test) cannot be restricted to sales group only.  


## Methodology

### Concept  
Implement access control policy in table form, like a typical Firewall.  

* Image Format  
  <img src=.\images\Firewall-Policy-Like_Data_Group.jpg>    

* Table Format  
  | Policy Order | Action | Source         | URL Pattern Method | Destination       | Auth or No Auth  | SSL Bypass or SSL Intercept | * Memo *                                             |
  |--------------|--------|----------------|--------------------|-------------------|------------------|-----------------------------|------------------------------------------------------|
  | 0000         | aclact | src            | pattern            | dst               | auth or no-auth  | ssl-bypass or ssl-intercept | Policy Order 0000 is Column. This record is ignored. |
  | 0010         | deny   | ANY            | equals             | bad-site.test     | *                | *                           |                                                      |
  | 0011         | deny   | ANY            | equals             | malware-site.test | *                | *                           |                                                      |
  | 0012         | deny   | 192.168.0.123  | *                  | ANY               | *                | *                           |                                                      |
  | 0013         | deny   | ANY            | matches_regex      | .*\\.eicar.org    | *                | *                           |                                                      |
  | 0100         | allow  | 10.0.0.0/8     | equals             | www.example.com   | no-auth          | ssl-intercept               |                                                      |
  | 0101         | allow  | 172.16.0.0/12  | equals             | bank.test         | auth             | ssl-bypass                  |                                                      |
  | 0102         | allow  | 192.168.0.0/16 | equals             | 1.1.1.1           | auth             | ssl-bypass                  |                                                      |
  | 0103         | allow  | 192.168.0.0/16 | *                  | ANY               | no-auth          | ssl-bypass                  |                                                      |
  | 0104         | allow  | ANY            | equals             | www.google.com    | auth             | ssl-intercept               |                                                      |

### Implementation  
This idea do implement to using Data Group.
Data Group is a Key-Value Store(KVS) that can store user-defined data.
But, KVS is not table form. Therefore add a twist.  

* Normal Data Group Structure  

  | String (Key) | Value   |
  |--------------|---------|
  | Key#1        | Value#1 |
  | Key#2        | Value#2 |
  | Key#3        | Value#3 |

* Abnormal Data Group Structure  
  Split Value to multiple user-defined fields.  
  Its used comma (,) separated.

  | String (Key) | Value                            |
  |--------------|----------------------------------|
  | Key#1        | Filed#1-1,Field#1-2,Field#1-3    |
  | Key#2        | Filed#2-1,Field#2-2,Field#2-3    |
  | Key#3        | Filed#3-1,Field#3-2,Field#3-3    |

  <img src=.\images\BIG-IP_WebUI_Data_Group.jpg>   

## Notes
**HTTP** Forward Proxy Virtual Server is **always** forwarded to **SSL** Forward Proxy Virtual Server to allow centralized application of policies.
This concept is inspired by the SSL Orchestrator (SSLO) module.


## Caution
In this implementation, access control policy is managed not by standard BIG-IP functions, but by user-created functions (iRules), which complicates operation.
Therefore, this concept is a bad example that does not take into account operation and maintainability.


## Environment
I tested this concept in F5 BIG-IP Virtual Edition Ver. 17.0.0.


## Prerequisite
* Enable LTM & APM modules.

* Create HTTP & SSL Forward Proxy Virtual Server.

* Set Access Profile to the **HTTP** Forward Proxy Virtual Server.


## Settings
1. Open SSL Client Profile for SSL Forward Proxy Virtual Server.  
   * Change Configuration drop down list: Basic => Advanced  
   * Change Non-SSL Connections: Disabled => Enabled  
     <img src=.\images\SSL_Client_Profile_Non-SSL_Connections.jpg> 
   * Change SSL Forward Proxy Bypass: Disabled => Enabled
     <img src=.\images\SSL_Client_Profile_SSL_Forward_Proxy_Bypass.jpg> 

2. Add Data Group.  
   Data Group Name: Firewall-Policy-Like_Data_Group  
   Change the Data Group Name in the source code as necessary.  

3. Add iRules.  
   * Create iRules: Rule_01_For_HTTP_Forward_Proxy  
   * Create iRules: Rule_02_For_SSL_Forward_Proxy  

4. Apply iRules to HTTP & SSL Forward Proxy Virtual Server  
   * Apply "Rule_01_For_HTTP_Forward_Proxy" to **HTTP** Forward Proxy Virtual Server  
   * Apply "Rule_02_For_SSL_Forward_Proxy" to **SSL** Forward Proxy Virtual Server  


## Disclaimer
In no event shall the author be liable for any claim, damages or other liability.  
The author will not provide any support.  

If you are going to use this code, you should know what you are doing.  
If you can't figure it out, you shouldn't use it.  


## Author
MyHomeNWLab: https://github.com/myhomenwlab  
