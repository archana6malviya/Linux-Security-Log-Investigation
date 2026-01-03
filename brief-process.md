# Goal

Check Linux logs to find:

❌ Failed SSH login

👤 Username used

🌍 Attacker IP

⏰ Time of attack


## 📂 STEP 1: Check if log file exists

      ls /var/log/auth.log

               ✅ If file name appears → OK
               ❌ If not → use journalctl

## 🔍 STEP 2: See recent login logs

      sudo tail -n 20 /var/log/auth.log

           📌 Shows last 20 security events
      
## 🔐 STEP 3: Find FAILED SSH logins

📌 This shows:

- Username

- IP address

- Time

**But for Failed SSH login we have to create Failed logins**

### ✅ CREATE FAILED SSH LOG

#### 🔹 1: Check SSH service

               sudo systemctl status ssh

                       If not running:

               sudo systemctl start ssh

#### 🔹 2: Find your IP

               ip a

                     Look for:

                inet 192.168.x.x

#### 🔹 3: Attempt WRONG SSH login

From same machine, run:

                ssh username@localhost

When asked password:
❌ Enter wrong password 2–3 times
❌ Then exit

You will see:

                Permission denied

## 🔹 Step 3: Now check failed SSH logs

                sudo grep "Failed password" /var/log/auth.log

✅ You WILL see output now-
                Failed password for invalid user wronguser from ::1 port 22 ssh2


### 🔍 What ::1 means

::1 is the IPv6 loopback address.

Address	Meaning
127.0.0.1	IPv4 localhost
::1	      IPv6 localhost

👉 Because you did:

                  ssh username@localhost
                  
Linux used IPv6, so it logged: from ::1

The source IP ::1 indicates a locally simulated SSH attack using IPv6 loopback for controlled testing.


### ✅ If  WANT an IPv4 address

Run this to force IPv4:

               ssh -4 username@127.0.0.1

Enter wrong password 2–3 times.

Then check logs again:

              sudo journalctl -u ssh -o cat | grep "Failed password"


Now you will see: Ip address



## ✅ step 4: USE REGEX
 
              sudo journalctl -u ssh | grep "Failed password" | grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}'

✔ Output:
              127.0.0.1

## ✅ Step 5: Extract username

              sudo journalctl -u ssh | grep "Failed password" | awk '{print $9}'

Expected:
              username

## 🔹 Step 6: Count Repeated Attack Attempts (Brute Force)

              sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr


📌 Shows most aggressive attacker IP

# 🔐 PART 2: Privilege Escalation Detection

## 🔹 Step 7: Detect sudo Attempts

             sudo grep "sudo" /var/log/auth.log


📌 Look for:

             authentication failure

             user NOT in sudoers

## 🔹 Step 8: Failed Root Access Attempts

              sudo grep "authentication failure" /var/log/auth.log

📜 PART 3: Using journalctl

## 🔹 Step 9: SSH Logs via journalctl

              sudo journalctl -u ssh --no-pager

## 🔹 Step 10: Filter Failed Logins

              sudo journalctl | grep "Failed password"

## 🔹 Step 11: Time-Based Investigation

              sudo journalctl --since "1 hour ago"


📌 Helps identify attack window
















