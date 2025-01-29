```python
import argparse
import os
import subprocess
import sys

class WebRecon:
    def __init__(self, ip_range=None, ip_list=None, threads=10, gowitness=False, osmedeus=False, pre_scanned_urls=None, whatweb=False):
        self.ip_range = ip_range
        self.ip_list = ip_list
        self.threads = threads
        self.run_gowitness = gowitness
        self.run_osmedeus = osmedeus
        self.pre_scanned_urls = pre_scanned_urls
        self.run_whatweb = whatweb
        
        self.output_dir = os.path.join(os.getcwd(), "web_recon")
        self.alive_hosts = os.path.join(self.output_dir, "ips.txt")
        self.nmap_output = os.path.join(self.output_dir, "nmap_scan_results.txt")
        self.formatted_urls = os.path.join(self.output_dir, "formatted_urls.txt")
        self.gowitness_output = os.path.join(self.output_dir, "gowitness")
        self.whatweb_output = os.path.join(self.output_dir, "whatweb")
        self.httpx_results = os.path.join(self.output_dir, "alive_urls.txt")

        if not os.path.exists(self.output_dir):
            os.makedirs(self.output_dir)
        
    def run_command(self, command):
        """Run a shell command and print its output in real time."""
        process = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
        while True:
            output = process.stdout.readline()
            if output == b'' and process.poll() is not None:
                break
            if output:
                print(output.strip().decode())
        rc = process.poll()
        return rc

    def discover_hosts(self):
        if self.ip_range:
            print(f"Starting host discovery with IP range {self.ip_range}\n")
            self.run_command(f"sudo nmap -PA -PE -PP -sn {self.ip_range} | grep 'Nmap scan report for' | awk '{{print $NF}}' > {self.alive_hosts}")
        elif self.ip_list:
            print(f"Using provided list of IPs from {self.ip_list}\n")
            os.system(f'cp {self.ip_list} {self.alive_hosts}')
        os.system(f'sed -i \'s/[()]//g\' {self.alive_hosts}')

    def scan_ports(self):
        if not os.path.exists(self.nmap_output):
            print("Starting nmap port scan on found hosts\n")
            self.run_command(f"sudo nmap -p- --open --min-rate=2750 --min-parallelism=32 -iL {self.alive_hosts} -oN {self.nmap_output}")
        
    def format_urls(self):
        if os.path.getsize(self.nmap_output) > 0:
            print(f"NMAP Scan results have been saved to {self.nmap_output}\n")
            self.run_command(f"awk '/^Nmap scan report for/ {{ip=$NF}} /open/ && /tcp/ {{print \"http://\"ip\":\"$1\"\\nhttps://\"ip\":\"$1}}' {self.nmap_output} | sed 's|/tcp||g' > {self.formatted_urls}")
        else:
            print("Nmap output file is empty or doesn't exist")
            sys.exit(1)

    def probe_ports(self):
        if os.path.getsize(self.formatted_urls) > 0:
            print("Starting httpx on URLs\n")
            self.run_command(f"/home/kali/go/bin/httpx -silent -t 50 -retries 3 -l {self.formatted_urls} -o {self.httpx_results}")
            self.run_command(f"sort -u {self.httpx_results} -o {self.httpx_results}")
            print(f"httpx results have been saved to {self.httpx_results}\n")
        else:
            print("Formatted ports file is empty or doesn't exist")
            sys.exit(1)

    def run_gowitness_scan(self):
        if os.path.getsize(self.httpx_results) > 0:
            print("Starting gowitness scan\n")
            if not os.path.exists(self.gowitness_output):
                os.makedirs(self.gowitness_output)
            self.run_command(f"cat {self.httpx_results} | parallel -j {self.threads} 'gowitness single -f {{}} --delay 10 --destination {self.gowitness_output}'")
            print(f"gowitness scan has finished and results are saved in {self.gowitness_output}\n")
        else:
            print("httpx results file is empty or doesn't exist")
            sys.exit(1)

    def run_whatweb_scan(self):
        if os.path.getsize(self.httpx_results) > 0:
            print("Starting whatweb scan\n")
            if not os.path.exists(self.whatweb_output):
                os.makedirs(self.whatweb_output)
            self.run_command(f"cat {self.httpx_results} | xargs -P {self.threads} -I {{}} whatweb -a 3 {{}} --log-json {self.whatweb_output}/$(echo {{}} | sed 's|https\\?://||;s|[/:]|_|g').json")
            print(f"whatweb scan has finished and results are saved in {self.whatweb_output}\n")
        else:
            print("httpx results file is empty or doesn't exist")
            sys.exit(1)

    def run_osmedeus_scan(self):
        if os.path.getsize(self.httpx_results) > 0:
            print("Starting osmedeus scan. Use docker logs to check progress\n")
            osmedeus_dir = os.path.join(self.output_dir, "osmedeus")
            if not os.path.exists(osmedeus_dir):
                os.makedirs(osmedeus_dir)
            self.run_command(f"sudo docker run -v {osmedeus_dir}:/root/workspaces-osmedeus -v {self.output_dir}:/dev/shm j3ssie/osmedeus scan -f urls -t /dev/shm/{self.httpx_results}")
            self.run_command(f"sudo chmod -R 777 {osmedeus_dir}")
        else:
            print("httpx results file is empty or doesn't exist")
            sys.exit(1)

    def execute(self):
        if self.pre_scanned_urls:
            print("Using pre-scanned URLs\n")
            os.system(f'cp {self.pre_scanned_urls} {self.httpx_results}')
        else:
            self.discover_hosts()
            self.scan_ports()
            self.format_urls()
            self.probe_ports()
        
        if self.run_gowitness:
            self.run_gowitness_scan()
        
        if self.run_whatweb:
            self.run_whatweb_scan()
        
        if self.run_osmedeus:
            self.run_osmedeus_scan()

def main():
    parser = argparse.ArgumentParser(description="Web Recon Script")
    parser.add_argument('-r', '--range', type=str, help='IP range in CIDR notation')
    parser.add_argument('-l', '--list', type=str, help='File with list of IPs')
    parser.add_argument('-t', '--threads', type=int, default=10, help='Number of threads (default: 10 for gowitness)')
    parser.add_argument('-s', '--gowitness', action='store_true', help='Run gowitness scan')
    parser.add_argument('-o', '--osmedeus', action='store_true', help='Run osmedeus scan')
    parser.add_argument('-u', '--urls', type=str, help='File with URLs (use pre-scanned URLs)')
    parser.add_argument('-w', '--whatweb', action='store_true', help='Run whatweb scan')
    
    args = parser.parse_args()

    if not args.range and not args.list and not args.urls:
        parser.print_help()
        sys.exit(1)

    recon = WebRecon(ip_range=args.range, ip_list=args.list, threads=args.threads, gowitness=args.gowitness, osmedeus=args.osmedeus, pre_scanned_urls=args.urls, whatweb=args.whatweb)
    recon.execute()

if __name__ == "__main__":
    main()
```