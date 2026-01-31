# ipv4_calc
Simple IPV4 Calculator

import ipaddress
from math import log2, ceil

def calculate_subnets(base_network, host_counts):
    """Calculate multiple subnets from a base network in sequential order"""
    subnets = []
    current_start = base_network.network_address
    
    for hosts in host_counts:
        # Calculate minimum subnet size needed (power of 2)
        needed_size = 2**(ceil(log2(hosts + 2)))  # +2 for network and broadcast
        prefix = 32 - int(log2(needed_size))
        
        # Create the subnet
        subnet = ipaddress.IPv4Network(f"{current_start}/{prefix}", strict=False)
        
        subnets.append({
            "Hosts Needed": hosts,
            "Network": subnet.network_address,
            "First Host": subnet.network_address + 1,
            "Last Host": subnet.broadcast_address - 1,
            "Broadcast": subnet.broadcast_address,
            "Subnet Mask": subnet.netmask,
            "CIDR": f"/{prefix}"
        })
        
        # Move to next available address
        current_start = subnet.broadcast_address + 1
        
        # Check if we've exceeded the base network
        if current_start > base_network.broadcast_address:
            raise ValueError("Not enough address space for all requested subnets")
    
    return subnets

def main():
    print("Sequential Subnet Calculator")
    print("---------------------------")
    
    # Get base network
    while True:
        try:
            network = input("Enter base IP network (e.g., 200.5.1.0/24): ")
            base_network = ipaddress.IPv4Network(network, strict=False)
            break
        except ValueError:
            print("Invalid network format. Please try again.")
    
    # Get subnet requirements
    host_counts = []
    while True:
        try:
            count = int(input("\nHow many subnets do you need? "))
            if count < 1:
                print("Must have at least 1 subnet.")
                continue
            break
        except ValueError:
            print("Please enter a number.")
    
    for i in range(count):
        while True:
            try:
                hosts = int(input(f"Hosts needed for subnet {i+1}: "))
                if hosts < 1:
                    print("Must have at least 1 host.")
                    continue
                host_counts.append(hosts)
                break
            except ValueError:
                print("Please enter a number.")
    
    # Calculate and display results
    print("\nBase Network Information:")
    print(f"Network: {base_network.network_address}/{base_network.prefixlen}")
    print(f"Subnet Mask: {base_network.netmask}")
    print(f"Total Addresses: {base_network.num_addresses}")
    print(f"Usable Hosts: {base_network.num_addresses - 2}\n")
    
    try:
        subnets = calculate_subnets(base_network, host_counts)
        
        for i, subnet in enumerate(subnets, 1):
            print(f"\nSubnet {i} ({subnet['Hosts Needed']} Hosts):")
            print(f"Network: {subnet['Network']}{subnet['CIDR']}")
            print(f"First Host: {subnet['First Host']}")
            print(f"Last Host: {subnet['Last Host']}")
            print(f"Broadcast: {subnet['Broadcast']}")
            print(f"Subnet Mask: {subnet['Subnet Mask']}")
    except ValueError as e:
        print(f"\nError: {str(e)}")

if __name__ == "__main__":
    main()
