🏗️ AZURE CLOUD LEARNING PROJECT — FROM MONOLITHIC TO 3-TIER ARCHITECTURE

---

# 🏗️ UPDATED GUIDE (Web + DB on Azure — FIXED)

In Microsoft Azure, this is a 2-tier architecture (Web + Database).

---

## 🧱 STEP 1: Create VNet

VNet: VNet-Lab-01  
CIDR: 10.0.0.0/16  

Subnets:
- web → 10.0.1.0/24  
- db → 10.0.2.0/24  

✔ Done

---

## 🌐 STEP 2: Deploy VMs

### ✔ Web VM
- Subnet: web
- Public IP: YES
- Ubuntu Linux

### ✔ DB VM
- Subnet: db
- Public IP: NO
- Private IP only

✔ Correct (important security practice)

---

## 🔥 STEP 3: NSG RULES (SIMPLIFIED)

### 🌐 Web NSG

Allow SSH:

- Source: My IP
- Port: 22
- Protocol: TCP
- Action: Allow
- Priority: 100

---

### 🗄️ DB NSG

Allow Web → DB:

- Source: 10.0.1.0/24
- Destination port: 3306
- Protocol: TCP
- Action: Allow
- Priority: 100

---

❌ Do NOT add:
- deny rules (unnecessary in this lab)
- SSH rules for DB (not needed externally)
- VirtualNetwork blocking rules

---

## 🛠️ STEP 4: Install MySQL (DB VM)

```bash
sudo apt update
sudo apt install mysql-server -y
```

---

## ⚙️ STEP 5: FIX MySQL (CRITICAL LESSON)

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Change:

```
bind-address = 127.0.0.1 ❌
```

TO:

```
bind-address = 0.0.0.0 ✔
```

Restart MySQL:

```bash
sudo systemctl restart mysql
```

Verify:

```bash
sudo ss -tulnp | grep 3306
```

✔ Expected:

```
0.0.0.0:3306
```

---

## 🧪 STEP 6: TEST CONNECTION

From Web VM:

```bash
nc -zv 10.0.2.4 3306
```

✔ Expected:

```
succeeded
```

---

## 🔐 STEP 7: ARCHITECTURE FLOW

```
Laptop → Web VM (public IP)
Web VM → DB VM (private IP)
Laptop → DB VM ❌ blocked (correct security design)
```

---