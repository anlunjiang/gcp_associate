# Managing Network Resources 

* Resizing a subnet
* Reserving internal and external ip address

# CIDR Notation
Each ipv4 address is 32 its in size, or 4 bytes, one byte for each number
 - 0.0.0.0/0 all ip ranges  
 The mask is the number of bits that are locked from left to right
* So if a mask is 32 bits there is only one ip address 
* 29 bits mean that it only has 3 bits to work with meaning 2^3=8 possible ip addresses

`ipaddress/<bits filtered from left to right>`

## Resize a subnet

You can then resize an existing subnet in a VPC by chanign the cidr notation of the ip address range

## Reserving static ip

* You can reserve a static internal ip address from the console for the VPC, can be automatically chosen or custom
    * good for migration 

* Creating a static external ip address is similar by clicking on the external ip addresses tab on the console and click reserve static ip address - 
### this is automatically generated for us

Google doesnt charge for any reserved ip address as long as you are using it, if you are NOT using it, then you get charged