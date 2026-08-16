# Adding a Third Vulnerable Target to the Home Lab (DVWA via Docker)

## Objective

Expand the home lab from two machines (1. Ubuntu Server as *hardened target*, 2. Kali as *attacker*) to three by adding an intentionally vulnerable target(DVWA).

This third machine gives the lab a real system to scan, enumerate, and later exploit.

## Steps Performed

1. **Install and run Metasploitable 2.** Attempted to import the official Metasploitable 2 VirtualBox image as the vulnerable target.
2. **Architecture blocker.** VirtualBox refused to power on the VM, returning:
   `This virtual machine cannot be powered on because it requires the x86 machine architecture, which is incompatible with this ARM machine architecture host.`

   *Metasploitable 2 ships as a fixed x86 disk image with no ARM build, and my host machine is Apple Silicon (ARM64). No workaround at the image level exists.*


4. **Switched to DVWA via Docker.** Chose DVWA (Damn Vulnerable Web Application) as the replacement target (*official images exist with multi-arch (including arm64) support*) avoiding the architecture issue entirely.
5. **Pulled an ARM-compatible image.** Used `ghcr.io/digininja/dvwa:latest`, confirmed to support arm64.

6. **Ran the container**, mapping it to port 8080 on the host:

`
   docker run -d -p 8080:80 --name dvwa ghcr.io/digininja/dvwa:latest
`

7. **Verified locally first.** Loaded `http://localhost:8080` on the host, confirmed the DVWA login page rendered, logged in with default credentials, and initialized the database (Create/Reset Database).

8. **Confirmed reachability from Kali.** From the Kali VM, ran:
   `
   nmap -p 8080 172.16.237.1
   `
   **Result**: port 8080/tcp open (http-proxy), confirming the host-run container is reachable from the attacker VM on the lab network. Also loaded the DVWA login page directly in Kali's browser to confirm end-to-end access.

   <img width="2070" height="1676" alt="day1 nmap scan 8080" src="https://github.com/user-attachments/assets/77f87774-d3bf-479a-917d-72c46426ed25" />


## Problems Encountered

Metasploitable 2's image is x86-only and cannot run on an Apple Silicon (ARM64) host under VirtualBox — VirtualBox errors out immediately at power-on rather than emulating. 

Pivoted to a target with native ARM64 support: DVWA running as a Docker container. This kept the lab goal intact (a reachable, intentionally vulnerable target on the network) while sidestepping the incompatibility.


