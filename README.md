# ASI

## Part 1: Build the network

## Part 2: Configure inital device settings

### Step 1: Configure R1 Basic Settings and Device Hardening

#### a. Configure basic settings

1. Prevent the router from attempting to resolve incorrectly entered commands as domain names.

```
enable
conf term
no ip domain lookup
```

2. Configure the R1 hostname.

```
hostname R1
```

3. Configure an appropriate MOTD banner.

```
banner motd #Unauthorized Acces is Prohibited#
```

#### b. Configure password security.

1. Configure the console password and enable connections.

```
line console 0
password cisco
login
exit
```

2. Configure an enable secret password.

```
enable secret cisco
```

3. Encrypt all clear text passwords.

```
service password-encryption
```

4. Set the minimum length of newly created passwords to 10
   characters.

```
security passwords min-length 10
```

#### c. Configure SSH.

1. Create an administrative user in the local user database

- Username: admin
- Encrypted Password: admin1pass

```
username admin secret admin1pass
```

2. Configure the domain name as ccna-ptsa.com

```
ip domain-name ccna-ptsa.com
```

3. Create an RSA crypto key with a modulus of 1024 bits.

```
crypto key generate rsa general-keys modulus 1024
```

4. Ensure that more secure version of SSH will be used.

```
ip ssh version 2
```

5. Configure the vty lines to authenticate logins against the local user database.

```
line vty 0 15
login local
```

6. Configure the vty lines to only accept connections over SSH.

```
transport input ssh
exit
```

### Step 2: Configure router interfaces.

#### a. Configure R1 with a loopback interface. Configure the loopback0 with IPv4 and IPv6 addressing according to the addressing table.

```
int lo0
ip address 209.165.201.1 255.255.255.224
ipv6 address 2001:db8:acad:209::1/64
ipv6 address fe80::1 link-local
description Simulated Internet
exit
```

#### b. Configure Router Subinterfaces

1. Prepare the router to be configured with IPv6 addresses on its interfaces.

```
ipv6 unicast-routing
```

2. Use the information in the Addressing Table and VLAN Table to configure subinterfaces on R1:

- Interfaces should be configured with IPv4 and IPv6 addressing.
- All addressed interfaces should use fe80::1 as the link local address.
- Use the VLAN table to assign VLAN membership to the subinterfaces.

```
int g0/0/1.2
encapsulation dot1q 2
description Bikes
ip address 10.19.8.1 255.255.255.192
ipv6 address 201:db8:acad:a::1/64
ipv6 address fe80::1 link-local

int g0/0/1.3
encapsulation dot1q 3
description Trikes
ip address 10.19.8.65 255.255.255.224
ipv6 address 2001:db8:acad:b::1/64
ipv6 address fe80::1 link-local

int g0/0/1.4
encapsulation dot1q 4
description Management
ip address 10.19.8.97 255.255.255.248
ipv6 address fe80::1 link-local

int g0/0/1.6
encapsulation dot1q 6
description Native

int g0/0/1
no shutdown
```

3. Be sure to configure the native VLAN interface.
4. Configure descriptions for all interfaces.

### Step 3: Configure S1 and S2 with Basic Settings and Device Hardening.

#### a. Configure Basic Settings on S1 and S2

1. Prevent the switches from attempting to resolve incorrectly entered commands as domain names.int

```
conf term
no ip domain lookup
```

2. Configure the S1 or S2 hostname.

```
hostname S1
```

3. Configure an appropriate MOTD banner on both switches.

```
banner motd %Unauthorized Access is Prohibited%
```

#### b. Configure Device Hardening on S1 and S2

1. Configure the console password and enable connections.

```
line console 0
password cisco
login
exit
```

2. Configure an enable secret password.

```
enable secret cisco
```

3. Encrypt all clear text passwords.

```
service password-encryption
```

#### c. Configure SSH on S1 and S2

1. Create an administrative user in the local user database.

- Username: admin
- Password: admin1pass

```
username admin secret admin1pass
```

2. Configure the domain name as ccna-ptsa.com

```
ip domain-name ccna-ptsa.com
```

3. Create an RSA crypto key with a modulus of 1024 bits.

```
crypto key generate rsa general-keys modulus 1024
```

4. Ensure that more secure version of SSH will be used.

```
ip ssh version 2
```

5. Configure the vty lines to authenticate logins against the local user database.

```
line vty 0 15
login local
```

6. Configure the vty lines to accept connections over SSH only.

```
transport input ssh
exit
```

### Step 4: Configure SVIs on S1 and S2

#### a. Use the information in the Addressing Table to configure SVIs on S1 and S2 for the Management VLAN.

#### b. Configure the switch so that the SVI can be reached from other networks over the Management VLAN.

```
int vlan 4
ip address 10.19.8.98 255.255.255.248
description Management SVI
no shutdown
exit
ip default-gateway 10.19.8.97
```

## Part 3: Configure Network Infrastructure Settings (VLANs, Trunking, EtherChannel)

### Step 1: Configure VLANs and Trunking.

#### a. Create the VLANs according to the VLAN table.

```
vlan 2
name Bikes
vlan 3
name Trikes
vlan 4
name Management
vlan 5
name Parking
vlan 6
name Native
exit
```

#### b. Create 802.1Q VLAN trunks on ports F0/1 and F0/2. On S1, F0/5 should also be configured as a trunk. Use VLAN 6 as the native VLAN.

```
int range f0/1-2
switchport mode trunk
switchport trunk native vlan 6
exit

int f0/5
switchport mode trunk
switchport trunk native vlan 6
exit
```

### Step 2: Configure Etherchannel.

Create Layer 2 EtherChannel port group 1 that uses interfaces F0/1 and F0/2 on S1 and S2. Both ends of the channel should negotiate the LACP link.<br/>

```
int range f0/1-2
channel-group 1 mode active
exit
```

`show etherchannel summary` to check

### Step 3: Configure Switchports.

#### a. On S1, configure the port that is connected to the host with static access mode in VLAN 2.

```
int f0/6
switchport mode access
switchport access vlan 2
exit
```

#### b. On S2, configure the port that is connected to the host with static access mode in VLAN 3.

#### c. Configure port security on the S1 and S2 active access ports to accept only three learned MAC addresses.

```
int f0/6
switchport port-security
switchport port-security maximum 3
switchport port-security mac-address sticky
exit
```

#### d. Assign all unused switch ports to VLAN 5 on both switches and shut down the ports.

```
int range f0/3-4,f0/-24,g0/1-2
switchport mode access
switchport access vlan 5
```

#### e. Configure a description on the unused ports that is relevant to their status.

```
description Unused Port
shutdown
exit
```

**REPEAT FOR S2**

## Part 4: Configure Host Support

### Step 1: Configure Default Routing on R1

### a. Configure an IPv4 default route that uses the Lo0 interface as the exit interface.

```
enable
conf term
ip route 0.0.0.0 0.0.0.0 lo0
```

### b. Configure an IPv6 default route that uses the Lo0 interface as the exit interface.

```
ipv6 route ::/0 lo0
```

### Step 2: Configure IPv4 DHCP for VLAN 2

#### a. On R1, create a DHCP pool called CCNA-A that consists of the last 10 host addresses in the VLAN 2 subnet only.

/26 --> 62 hosts 255.255.255.192<br/>

```
ip dhcp excluded-address 10.19.8.1 10.19.8.52
ip dhcp pool CCNA-A
network 10.19.8.0 255.255.255.192
```

#### b. Configure the correct default gateway address in the pool.

```
default-router 10.19.8.1
```

#### c. Configure the domain name of ccna-a.net.

```
domain ccna-a.net
exit
```

### Step 3: Configure IPv4 DHCP for VLAN 3

#### a. On R1, create a DHCP pool called CCNA-B that consists of the last 10 host addresses in the VLAN 3 subnet only.

/27 --> 30 hosts 255.255.255.224

```
enable
conf term
ip dhcp excluded-address 10.19.8.65 10.19.8.84
ip dhcp pool CCNA-B
network 10.19.8.64 255.255.255.224
```

#### b. Configure the correct default gateway address in the pool.

```
default-router 10.19.8.65
```

#### c. Configure the domain name of ccna-b.net.

```
domain ccna-b.net
exit
```

### Step 4: Configure host computers.

#### a. Configure the host computers to use DHCP for IPv4 addressing.

go to PCA ip config and select dhcp<br>
go to PCB ip config and select dhcp

#### b. Statically assign the IPv6 GUA and default gateway addresses using the values in the Addressing Table.

click on PCA and add static ipv6 address<br/>
same for PCB<br/>
to test ssh `ssh -l admin 10.19.8.1`
