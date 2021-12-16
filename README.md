# ansible-wireguard-unifi

### Use with caution, not updated for a long time (I don't use this anymore)

Install Wireguard for UniFi USG3, and configure to use as a client for Mullvad.

This is specific to my use case, and sends every bit of traffic on my 192.168.1.0/24 network through Mullvad, documenting here for future reference.

This was inspired by [this Reddit post](https://www.reddit.com/r/Ubiquiti/comments/dltf4e/wireguard_vpn_policy_routing_on_a_usg/)
## Requirements

- User and SSH keys added to your UniFi USG3
- Mullvad VPN account
- Ansible 2.9.x =< (via Ubuntu for Windows)

## Step by step

None of this covers the initial set up of a USG, this is specific to *just* Wireguard.

Log in with your Mullvad VPN credentials and [generate a Wireguard configuration file from here](https://mullvad.net/en/download/wireguard-config/)

Once downloaded, this file will look something like this:
```
[Interface]
PrivateKey = abc123private
Address = 123.456.789.0/32
DNS = 193.138.218.74

[Peer]
PublicKey = abc123public
AllowedIPs = 0.0.0.0/0
Endpoint = 987.654.321.0:51820
```

The values here are then used in a dictionary stored in `inventory/group_vars/usg.yml`, modify them as follows:
```
wireguard_config:
  name: 'Name_of_network_group'
  network: 'Your local subnet (I chose 192.168.1.0/24)'
  address: 'Value from [Interface] -> Address goes here'
  endpoint: 'Value from [Peer] -> Endpoint goes here'
  public_key: 'Value from [Peer] -> PublicKey goes here'
  private_key: 'Value from [Peer] -> PrivateKey goes here'
```

Modify the values found in `inventory/group_vars/all.yml`, `inventory/group_vars/usg.yml` and `inventory/hosts` to verify that they are correct for your use case.

Run the playbook, specifying your unifi SSH username:

`$ ansible-playbook -i inventory/ playbook.yml -e ansible_user=unifi`

The USG3 will install the specified version of Wireguard, the file will be templated out to the location you specify, and the USG will reboot. All being well all traffic will go over Mullvad VPN.

**Note:**
  - It did take a couple of provisions / reboots for this to fully function
  - This was done through Ubuntu for Windows, since my controller is currently on Windows 10
