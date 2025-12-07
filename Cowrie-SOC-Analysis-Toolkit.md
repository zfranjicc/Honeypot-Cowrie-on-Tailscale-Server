**Cowrie SOC Analysis Toolkit** is a script designed for fast and efficient analysis of Cowrie honeypot logs. It allows you to review attacks, IP addresses, brute-force attempts, and executed commands just like a real SOC analyst.




Scripts look like this :

---
<img width="467" height="347" alt="Screenshot 2025-12-07 183356" src="https://github.com/user-attachments/assets/ebe63cb9-d8ee-434d-b6b0-a79b12fc2ee3" />



## How to make?

sudo nano /usr/local/bin/cowview

Past script 
 ```
#!/bin/bash

LOGDIR="/home/cowrie/cowrie/var/log/cowrie"

menu() {
    echo "Cowrie Log Viewer (SOC Toolkit)"
    echo "-----------------------------------"
    echo "[1] Live logs"
    echo "[2] Full cowrie.log"
    echo "[3] Search in logs"
    echo "[4] View rotated logs (*.gz)"
    echo "[5] Show only attacker IPs"
    echo "[6] Brute-force attempt counter"
    echo "[7] Extract attacker commands"
    echo "[8] Top attackers by number of commands"
    echo "[9] Exit"
    echo ""
    read -p "Choose an option: " option
}

show_ips() {
    echo "Unique attacker IPs:"
    grep -hoP '"src_ip": "\K[0-9\.]+' "$LOGDIR"/*.log* | sort -u
}

bruteforce_count() {
    echo " Brute-force attempts:"
    grep -h "login attempt" "$LOGDIR"/cowrie.log* | wc -l
}

extract_commands() {
    echo "Commands executed by attackers:"
    grep -h '"input"' "$LOGDIR"/cowrie.log* | jq -r '.input' | sort -u | less -R
}

top_attackers() {
    echo " Top attackers by number of commands:"
    grep -h '"src_ip"' "$LOGDIR"/cowrie.log* \
        | grep -oP '"src_ip": "\K[0-9\.]+' \
        | sort | uniq -c | sort -nr | head -20
}

menu

case $option in
    1)
        echo " Live log stream..."
        tail -f "$LOGDIR/cowrie.log"
        ;;
    2)
        less -R "$LOGDIR/cowrie.log"
        ;;
    3)
        read -p "Search keyword: " search
        grep -iR "$search" "$LOGDIR" | less -R
        ;;
    4)
        echo " Rotated logs:"
        ls "$LOGDIR"/*.gz 2>/dev/null
        echo ""
        read -p "Pick a file to open: " file
        zcat "$file" | less -R
        ;;
    5)
        show_ips
        ;;
    6)
        bruteforce_count
        ;;
    7)
        extract_commands
        ;;
    8)
        top_attackers
        ;;
    9)
        exit 0
        ;;
    *)
        echo "Invalid option."
        ;;
esac
 ```
Give premission 
 ```
sudo chmod +x /usr/local/bin/cowview
 ```

and run your cowview 

 ```
sudo cowview
 ```
