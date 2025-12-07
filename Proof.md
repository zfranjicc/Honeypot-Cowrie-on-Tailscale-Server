
## 1. Attacker Connection

The screenshot below shows a remote machine connecting to the server over SSH.  
The attacker successfully logs into the fake environment (Cowrie), believing it is a real Linux server.

Cowrie presents a real-looking shell prompt:

    admin@svr04:~$

    
## 2. Cowrie Captures All Commands

On the right side, Cowrie logs every command the attacker enters.
In this test, the attacker typed:

    Hello GitHub THIS IS ATTACKER PC

Cowrie immediately logged:
- the full command
- source IP address
- session ID
- timestamp
- protocol (SSH)


## This is live logs!


  <img width="1907" height="995" alt="PROOF" src="https://github.com/user-attachments/assets/63c8478e-c2fe-43ca-930e-4317b045fc0e" />
