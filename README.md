# What we will learn

In this tutorial, you will learn how to install, configure, and use Cowrie, a medium-interaction SSH/Telnet honeypot. Cowrie allows you to capture real attacker activity, log every command they execute, and mislead them using realistic fake files and a simulated filesystem.

This setup runs Cowrie on a Tailscale Zero-Trust server, meaning your real system remains hidden from the public internet. Only the honeypot ports are exposed on your public IP, allowing you to safely observe attacks without risking your actual server.

When Cowrie is stopped, the exposed ports close and your server becomes hidden again. 

Later in this repository, additional tools will be added, including:

- an automatic log analysis script (extract IPs, commands, keywords, payloads)

- a systemd auto-start service to run Cowrie on every reboot

- optional enhancements for threat intelligence and easier monitoring


