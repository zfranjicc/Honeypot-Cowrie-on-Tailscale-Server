**Cowrie SOC Analysis Toolkit** is a script designed for fast and efficient analysis of Cowrie honeypot logs. It allows you to review attacks, IP addresses, brute-force attempts, and executed commands just like a real SOC analyst.




Scripts look like this :

---

<img width="600" height="397" alt="Screenshot 2025-12-08 211820" src="https://github.com/user-attachments/assets/c8caff76-fc60-4f3f-baca-1382c7ebc021" />



## How to make?

sudo nano /usr/local/bin/cowview

Past script 
 ```
╔══════════════════════════════════════════════╗
║ Cowrie Log Analysis Toolkit                  ║
║ by zfranjic                                  ║
╚══════════════════════════════════════════════╝

 [1] Live logs (tail -f)
 [2] View current log
 [3] Search all logs
 [4] View rotated logs
 [5] All attacker IPs (top 40)
 [6] Brute-force statistics
 [7] All executed commands
 [8] Top 20 attackers by commands
 [9] Top 25 attack commands + analysis
 [10] Exit

Choose [1-10]^Z
[38]+  Stopped                 sudo cowview
admin@Baldwing:~$ sudo cat /usr/local/bin/cowview
#!/bin/bash
RED='\033[1;31m'
GREEN='\033[1;32m'
YELLOW='\033[1;33m'
BLUE='\033[1;34m'
PURPLE='\033[1;35m'
CYAN='\033[1;36m'
WHITE='\033[1;37m'
NC='\033[0m'
LOG_DIR="/home/cowrie/cowrie/var/log/cowrie"
LOGS="$LOG_DIR/cowrie.json*"

banner() {
  clear
  echo -e "${CYAN}╔══════════════════════════════════════════════╗${NC}"
  echo -e "${CYAN}║ Cowrie Log Analysis Toolkit                  ║${NC}"
  echo -e "${CYAN}║ by zfranjic                                  ║${NC}"
  echo -e "${CYAN}╚══════════════════════════════════════════════╝${NC}"
  echo
}

while true; do
  banner
  echo -e " ${GREEN}[1]${NC} Live logs (tail -f)"
  echo -e " ${GREEN}[2]${NC} View current log"
  echo -e " ${GREEN}[3]${NC} Search all logs"
  echo -e " ${GREEN}[4]${NC} View rotated logs"
  echo -e " ${BLUE}[5]${NC} All attacker IPs (top 40)"
  echo -e " ${YELLOW}[6]${NC} Brute-force statistics"
  echo -e " ${PURPLE}[7]${NC} All executed commands"
  echo -e " ${RED}[8]${NC} Top 20 attackers by commands"
  echo -e " ${RED}[9]${NC} Top 25 attack commands + analysis"
  echo -e " ${WHITE}[10]${NC} Exit"
  echo
  read -p "Choose [1-10]" opt
  echo

  case $opt in

    1) tail -f $LOG_DIR/cowrie.json ;;

    2) less $LOG_DIR/cowrie.json ;;

    3)
      read -p "Search keyword: " kw
      grep -i "$kw" $LOGS 2>/dev/null | less -R
      ;;

    4)
      ls -lh $LOG_DIR/cowrie.json*.gz 2>/dev/null || echo "No rotated logs"
      read -p "File to open: " f
      zless "$LOG_DIR/$f" 2>/dev/null
      read -r
      ;;

    5)
      echo -e "${BLUE}   TOP 40 ATTACKER IPs${NC}"
      echo -e "${BLUE}   ═════════════════════${NC}"
      jq -r 'select(.eventid=="cowrie.session.connect") | .src_ip' $LOGS 2>/dev/null | sort | uniq -c | sort -nr | head -40 | nl
      read -r
      ;;

    6)
      echo -e "${YELLOW}   BRUTE-FORCE STATISTICS (ALL logs)${NC}"
      echo -e "${YELLOW}   ═══════════════════════════════════════${NC}"
      FAILED=$(jq -s '[.. | select(.eventid? == "cowrie.login.failed")] | length' $LOGS 2>/dev/null || echo 0)
      SUCCESS=$(jq -s '[.. | select(.eventid? == "cowrie.login.success")] | length' $LOGS 2>/dev/null || echo 0)
      TOTAL=$((FAILED + SUCCESS))
      printf "   ${RED}Failed attempts     : %'d${NC}\n" "$FAILED"
      printf "   ${GREEN}Successful logins (honeypot): %'d${NC}\n" "$SUCCESS"
      printf "   ${YELLOW}───────────────────────────────────────${NC}\n"
      printf "   ${WHITE}TOTAL SESSIONS      : %'d${NC}\n" "$TOTAL"
      echo
      read -r
      ;;

    7)
      jq -r 'select(.eventid=="cowrie.command.input") | [.timestamp,.src_ip,.input] | @tsv' $LOGS 2>/dev/null | less -R
      ;;

    8)
      echo " TOP 20 ATTACKERS BY EXECUTED COMMANDS"
      echo " ═══════════════════════════════════════════════════════════"
      echo " #      COMMANDS count        IP ADDRESS"
      echo " ───────────────────────────────────────────────────────────"
      jq -r 'select(.eventid=="cowrie.command.input") | .src_ip' $LOGS 2>/dev/null | \
        sort | uniq -c | sort -nr | head -20 | nl | \
        awk '{printf " %2s   %8s      %s\n", $1, $2, $3}'
      echo
      read -r
      ;;
    9)
      echo -e "${PURPLE}   TOP 25 ATTACK COMMANDS + ANALYSIS${NC}"
      echo -e "${YELLOW}   ═════════════════════════════════════════════${NC}"
      echo -e "${YELLOW}   #   COUNT  COMMAND                                     ANALYSIS${NC}"
      echo -e "${YELLOW}   ────────────────────────────────────────────────${NC}"
      jq -r 'select(.eventid=="cowrie.command.input") | .input' $LOGS 2>/dev/null | \
        sort | uniq -c | sort -nr | head -25 | nl | \
        while read -r n c cmd; do
          case "$cmd" in
            *wget*|*curl*|*tftp*) col=$RED;   ana="Payload download – CRITICAL" ;;
            *uname*|*whoami*|*id*) col=$YELLOW; ana="Reconnaissance" ;;
            *sh|*bash|*chmod*) col=$RED;   ana="Shell attempt" ;;
            *rm*|*cat\ /dev/null*) col=$YELLOW; ana="Anti-forensics" ;;
            *) col=$GREEN; ana="Other" ;;
          esac
          printf "   ${WHITE}%2s${NC} ${col}%7s${NC}   %-42.42s  ${col}%s${NC}\n" "$n" "$c" "$cmd" "$ana"
        done
      read -r
      ;;

    10)
      clear
      echo -e "${GREEN}╔══════════════════════════════════════════════╗${NC}"
      echo -e "${GREEN}║        Cowview zatvoren. Hvala!              ║${NC}"
      echo -e "${GREEN}║              by zfranjic                     ║${NC}"
      echo -e "${GREEN}╚══════════════════════════════════════════════╝${NC}"
      exit 0
      ;;

    *) echo -e "${RED}Wrong Command, try again${NC}"; sleep 1 ;;

  esac
done
 ```
Give premission 
 ```
sudo chmod +x /usr/local/bin/cowview
 ```

and run your cowview 

 ```
sudo cowview
 ```
