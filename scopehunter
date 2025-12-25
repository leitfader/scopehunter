#!/usr/bin/env python3

import os
import time
import subprocess
import sys
import re
import threading

# update-all is a simple script i made to update everything in a single command.
# you can either comment out the update part if you think its unnecessary
banner = r"""
                               __                __           
   ______________  ____  ___  / /_  __  ______  / /____  _____
  / ___/ ___/ __ \/ __ \/ _ \/ __ \/ / / / __ \/ __/ _ \/ ___/
 (__  ) /__/ /_/ / /_/ /  __/ / / / /_/ / / / / /_/  __/ /    
/____/\___/\____/ .___/\___/_/ /_/\__,_/_/ /_/\__/\___/_/     
               /_/                       

					Version 1.0 by leitfader 

"""

for line in banner.splitlines():
    print(line)
    sys.stdout.flush()
    time.sleep(0.02)


process = subprocess.Popen(
    ["sudo", "update-all"],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    universal_newlines=True,
    bufsize=1
)

spinner = ["⠋","⠙","⠹","⠸","⠼","⠴","⠦","⠧","⠇","⠏"]
i = 0
print("Updating… summoning the background Oompa-Loompas...\n")

while True:
    # Non-blocking read of a single line
    line = process.stdout.readline()
    if not line and process.poll() is not None:
        break

    sys.stdout.write(f"\r{spinner[i % len(spinner)]} Doing system wizardry… ")
    sys.stdout.flush()
    i += 1
    time.sleep(0.10)
code = process.wait()
print("\nSystem is up to date!")
time.sleep(2)
os.system('clear')
# Ask user if domains will come from files or manually
file_mode = input("Load domains from file? (y/n): ").strip().lower()

# Toolchains
def save_domains():
    domains = []
    wildcards = []
    print('Specify in scope targets (type END when finished)')
    while True:
        domain = input('Target: ')
        if domain == "END":
            break
        if domain.startswith("*."):
            wildcards.append(domain[2:])
        else:
            domains.append(domain)
    
    with open("domains.txt", "w") as file:
        for domain in domains:
            file.write(domain + '\n')
    
    with open("wildcards.txt", "w") as file:
        for wildcard in wildcards:
            file.write(wildcard + '\n')

    print('Scope saved. \n')

def sub_httpx():
    # If wildcards.txt exists, use subfinder and append to domains.txt
    if os.path.exists('wildcards.txt'):
        os.system('subfinder -dL wildcards.txt -all | anew domains.txt')
    # Then run httpx on domains.txt as before
    os.system('cat domains.txt | httpx | anew hosts.txt')
    os.system('cat domains.txt | httpx -wc -sc -cl -ct -web-server -asn -o detailed-hosts.txt -p 8080,8000,8443,443,80,8008,3000,5000,9090,900,7070,9200,15672,9000 -threads 75 -location -hae 96c8e9cf-4d77-45f5-bc3c-dd5d14919358')

# Saving hosts in Acunetix format for easy import as i usually use it.
def acu():
    os.system('scp hosts.txt acu.csv')
    with open('acu.csv', 'r') as file:
        urls = file.readlines()
    urls_with_commas = [url.strip() + ',' for url in urls]
    with open('acu.csv', 'w') as file:
        file.write('\n'.join(urls_with_commas) + '\n')

def nuclei():
    os.system("nuclei -list hosts.txt -nmhe -rl 5 -nh -ldp -c 150 -markdown-export nuclei-out")

# Also a tool that I made that's also available on my Github.
def waybackurls():
    os.system('veybekci')

# Paramspider is actually optional I usually find my cdx search tool enough paramspider just seems a waste of time and traffic for me but some find it usefull
def paramspider():
    os.system('paramspider -l domains.txt')


# Decision logic for scope source
if file_mode == 'y':
    domains_exists = os.path.exists("domains.txt")
    wildcards_exists = os.path.exists("wildcards.txt")

    if domains_exists or wildcards_exists:
        print("Using existing scope files:")
        if domains_exists:
            print("- domains.txt")
        if wildcards_exists:
            print("- wildcards.txt (will be used with subfinder)")
        # No need to call save_domains, we'll just proceed
    else:
        print("No domains.txt or wildcards.txt found! Switching to manual input...\n")
        save_domains()
else:
    save_domains()

print('Scope confirmed!')
time.sleep(2)

# Main flow
sub_httpx()
acu()
paramspider()
waybackurls()
nuclei()
