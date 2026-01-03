# Goal
Check Linux logs to find:

❌ Failed SSH login

👤 Username used

🌍 Attacker IP

⏰ Time of attack

📂 STEP 1: Check if log file exists

ls /var/log/auth.log


✅ If file name appears → OK
❌ If not → use journalctl

🔍 STEP 2: See recent login logs
sudo tail -n 20 /var/log/auth.log


📌 Shows last 20 security events
📸 Screenshot
🔐 STEP 3: Find FAILED SSH logins



sudo grep "Failed password" /var/log/auth.log
📌 This shows:

Username

IP address

Time

📸 Screenshot
👉 This is your MAIN project proof


✅ CREATE FAILED SSH LOG (SAFE PRACTICE)
🔹 Step 1: Check SSH service
sudo systemctl status ssh


If not running:

sudo systemctl start ssh

🔹 Step 2: Find your IP
ip a


Look for:

inet 192.168.x.x

🔹 Step 3: Attempt WRONG SSH login (IMPORTANT)

From same machine, run:

ssh wronguser@localhost


When asked password:
❌ Enter wrong password 2–3 times
❌ Then exit

You will see:

Permission denied

🔹 Step 4: Now check failed SSH logs (IT WILL WORK)
sudo grep "Failed password" /var/log/auth.log


✅ You WILL see output now

🔍 What ::1 means
::1 is the IPv6 loopback address.

Address	Meaning
127.0.0.1	IPv4 localhost
::1	IPv6 localhost

👉 Because you did:

ssh wronguser@localhost
Linux used IPv6, so it logged:

from ::1

The source IP ::1 indicates a locally simulated SSH attack using IPv6 loopback for controlled testing.


✅ If you WANT an IPv4 address

Run this to force IPv4:

ssh -4 wronguser@127.0.0.1


Enter wrong password 2–3 times.

Then check logs again:

sudo journalctl -u ssh -o cat | grep "Failed password"


Now you will see:

from 127.0.0.1

✅ USE REGEX (SOC analysts use this):
sudo journalctl -u ssh | grep "Failed password" | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'

✔ Output:
127.0.0.1

✅ Step 3: Extract username (SIMPLE & WORKING)
sudo journalctl -u ssh | grep "Failed password" | awk '{print $9}'


Expected:

wronguser
🔹 Step 4: Count Repeated Attack Attempts (Brute Force)
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr


📌 Shows most aggressive attacker IP
🔐 PART 2: Privilege Escalation Detection
🔹 Step 6: Detect sudo Attempts
sudo grep "sudo" /var/log/auth.log


📌 Look for:

authentication failure

user NOT in sudoers

🔹 Step 7: Failed Root Access Attempts
sudo grep "authentication failure" /var/log/auth.log

📜 PART 3: Using journalctl (Advanced SOC Skill)
🔹 Step 8: SSH Logs via journalctl
sudo journalctl -u ssh --no-pager

🔹 Step 9: Filter Failed Logins
sudo journalctl | grep "Failed password"

🔹 Step 10: Time-Based Investigation
sudo journalctl --since "1 hour ago"


📌 Helps identify attack window
















