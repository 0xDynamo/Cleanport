#!/bin/bash

echo -n "Enter the target IP address: "
read target_ip

echo -n "Performing initial port scan: "
sudo nmap -sS -p- $target_ip -oN initial

# Ask for the filename to read
echo -n "Enter the filename to read (e.g., targets.nmap): "
read filename

# Ask for the filename to save the formatted ports
echo -n "Enter the filename to save the formatted ports (e.g., formatted_ports.txt): "
read output_filename

# Read the specified file, extract ports, and format them
ports=$(cat "$filename" | grep -Eo '^[0-9]+/' | cut -d '/' -f 1 | tr '\n' ',')

# Save the formatted ports to the specified output file
echo "$ports" > "$output_filename"

echo "Formatted ports saved to $output_filename"



# Ask for additional information
echo -n "Enter the name for the nmap output file (e.g., nmap_scan): "
read scan_name


# Run UDP Nmap scan in a new tmux pane
tmux new-window -n "UDP Scan" "sudo nmap -sU -T3 --max-retries 3 -p 7,9,13,17,19,21-23,37,42,49,53,67-69,80,88,111,120,123,135-139,158,161-162,177,192,199,200,389,407,427,443,445,464,497,500,514-515,517-518,520,593,623,626,631,664,683,800,989-990,996-999,1001,1008,1019,1021-1034,1036,1038-1039,1041,1043-1045,1049,1068,1419,1433-1434,1645-1646,1701,1718-1719,1782,1812-1813,1885,1900,2000,2002,2048-2049,2148,2222-2223,2967,3052,3130,3283,3389,3456,3659,3703,4000,4045,4444,4500,4672,5000-5001,5060,5093,5351,5353,5355,5500,5632,6000-6001,6346,7938,9200,9876,10000,10080,11487,16680,17185,19283,19682,20031,22986,27892,30718,31337,32768-32773,32815,33281,33354,34555,34861-34862,37444,39213,41524,44968,49152-49154,49156,49158-49159,49162-49163,49165-49166,49168,49171-49172,49179-49182,49184-49196,49199-49202,49205,49208-49211,58002,65024 $target_ip -oN udp_scan"



# SSH audit
if grep -q "22," "$output_filename" || grep -q "2222," "$output_filename"; then
    if grep -q "22," "$output_filename"; then
        echo "Port 22 (SSH) found. Running ssh-audit..."
        # Assuming ssh-audit is installed and available in the PATH
        tmux new-window -n "SSH-audit" "ssh-audit $target_ip > ssh-audit22.txt"
    fi
    if grep -q "2222," "$output_filename"; then
        echo "Port 2222 (SSH) found. Running ssh-audit..."
        # Assuming ssh-audit is installed and available in the PATH
        tmux new-window -n "SSH-audit" "ssh-audit $target_ip > ssh-audit2222.txt"
    fi
fi


# Insert code for running gobuster and nikto here
# Check if port 80, 8080, or 8888 is present and run gobuster and nikto accordingly
if grep -q "80," "$output_filename" || grep -q "8080," "$output_filename" || grep -q "8888," "$output_filename"; then
    if grep -q "80," "$output_filename"; then
        echo "Port 80 (HTTP) found. Running gobuster and nikto..."
        # Assuming gobuster and nikto are installed and available in the PATH
        tmux new-window -n "Gobuster" "gobuster dir -u http://$target_ip:80/  -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -x .php,.txt,.html -o directories80.txt"
        tmux new-window -n "Nikto" "nikto -h http://$target_ip -output nikto80.txt"
    fi
    if grep -q "8080," "$output_filename"; then
        echo "Port 8080 found. Running gobuster and nikto..."
        # Assuming gobuster and nikto are installed and available in the PATH
        tmux new-window -n "Gobuster" "gobuster dir -u http://$target_ip:8080/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -x .php,.txt,.html -o directories8080.txt"
        tmux new-window -n "Nikto" "nikto -h http://$target_ip:8080 -output nikto8080.txt"
    fi
    if grep -q "8888," "$output_filename"; then
        echo "Port 8888 found. Running gobuster and nikto..."
        # Assuming gobuster and nikto are installed and available in the PATH
        tmux new-window -n "Gobuster" "gobuster dir -u http://$target_ip:80/  -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -x .php,.txt,.html -o directories8888.txt"
        tmux new-window -n "Nikto" "nikto -h http://$target_ip:8888 -output nikto8888.txt"
    fi
fi









# Run nmap command with specified options
nmap -sC -sV -A -p "$ports" -oN "$scan_name" "$target_ip"

# Function to prompt for script execution
prompt_for_scripts() {
    local service_name="$1"
    local scripts="$2"

    echo -n "Do you want to run all $service_name scripts? (y/n): "
    read -r choice
    if [ "$choice" == "y" ] || [ "$choice" == "Y" ]; then
        echo "Running all $service_name scripts..."
        nmap "$target_ip" --script "$scripts" -oA "${service_name}_scan" "$target_ip"
    else
        echo "Skipping all $service_name scripts."
    fi
}

# Check if specific ports were found and run corresponding NSE scripts

# FTP NSE scripts
if grep -q "21," "$output_filename"; then
    echo "Port 21 found. Running FTP NSE scripts..."
    prompt_for_scripts "FTP" "ftp-anon.nse,ftp-bounce.nse,ftp-brute.nse,ftp-libopie.nse,ftp-proftpd-backdoor.nse,ftp-syst.nse,ftp-vsftpd-backdoor.nse,ftp-vuln-cve2010-4221.nse,tftp-enum.nse"
fi

# SMTP NSE scripts
if grep -q "25," "$output_filename"; then
    echo "Port 25 (SMTP) found. Running SMTP NSE scripts..."
    prompt_for_scripts "SMTP" "smtp-brute.nse,smtp-commands.nse,smtp-enum-users.nse,smtp-ntlm-info.nse,smtp-open-relay.nse,smtp-strangeport.nse,smtp-vuln-cve2010-4344.nse,smtp-vuln-cve2011-1720.nse,smtp-vuln-cve2011-1764.nse"
fi

# SMB NSE scripts
if grep -q "445," "$output_filename"; then
    echo "Port 445 found. Running SMB NSE scripts..."
    prompt_for_scripts "SMB" "smb2-capabilities.nse,smb2-security-mode.nse,smb2-time.nse,smb2-vuln-uptime.nse,smb-brute.nse,smb-double-pulsar-backdoor.nse,smb-enum-domains.nse,smb-enum-groups.nse,smb-enum-processes.nse,smb-enum-services.nse,smb-enum-sessions.nse,smb-enum-shares.nse,smb-enum-users.nse,smb-flood.nse,smb-ls.nse,smb-mbenum.nse,smb-os-discovery.nse,smb-print-text.nse,smb-protocols.nse,smb-psexec.nse,smb-security-mode.nse,smb-server-stats.nse,smb-system-info.nse,smb-vuln-conficker.nse,smb-vuln-cve2009-3103.nse,smb-vuln-cve-2017-7494.nse,smb-vuln-ms06-025.nse,smb-vuln-ms07-029.nse,smb-vuln-ms08-067.nse,smb-vuln-ms10-054.nse,smb-vuln-ms10-061.nse,smb-vuln-ms17-010.nse,smb-vuln-regsvc-dos.nse,smb-vuln-webexec.nse,smb-webexec-exploit.nse"
fi






# Add more conditions for other ports as needed
# Example:
# if grep -q "80," "$output_filename"; then
#     echo "Port 80 found. Running HTTP NSE scripts..."
#     prompt_for_scripts "HTTP" "http-scripts"
# fi

# Continue with additional port-specific NSE script checks
