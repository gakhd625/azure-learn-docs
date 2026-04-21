# Mini Project 1 — Secure 3-Tier Network on Azure

> **Day 07 · Week 1 · Mini / Final Project**  
> Build a production-style secure network architecture on Azure using VNets, NSGs, and RBAC.

---

## Architecture Overview

```
                        [ Internet ]
                             |
                       80 / 443 only
                             |
          ┌──────────────────▼──────────────────┐
          │  snet-web  ·  10.0.1.0/24           │
          │  NSG-web: allow 80/443 from Any     │
          │  NSG-web: allow 22 from your IP     │
          │  [ vm-web · Public IP ]             │
          │  RBAC: dev-user = Contributor       │
          └──────────────────┬──────────────────┘
                             |
                  port 8080 · internal only
                             |
          ┌──────────────────▼──────────────────┐
          │  snet-app  ·  10.0.2.0/24           │
          │  NSG-app: allow from 10.0.1.0/24    │
          │  [ vm-app · No public IP ]          │
          └──────────────────┬──────────────────┘
                             |
               3306 / 5432 · internal only
                             |
          ┌──────────────────▼──────────────────┐
          │  snet-db  ·  10.0.3.0/24            │
          │  NSG-db: allow from 10.0.2.0/24     │
          │  [ vm-db · No public IP ]           │
          └─────────────────────────────────────┘

          All subnets live inside:
          vnet-3tier · 10.0.0.0/16
          Resource group: rg-3tier-lab
```

---

## Tasks Checklist

- [ ] Design: VNet + 3 subnets (web, app, database)
- [ ] Deploy VMs in each tier with appropriate NSG rules
- [ ] NSGs: web=80/443 open, app=internal only, db=app-only
- [ ] RBAC: assign dev user as Contributor on web tier only
- [ ] Document the architecture with a diagram

---

## Prerequisites

- An [Azure free account](https://azure.microsoft.com/free/) or Azure for Students
- Access to the [Azure Portal](https://portal.azure.com)
- Basic familiarity with cloud networking concepts

> **Cost note:** Using `Standard_B1s` VMs costs ~$0.008/hr each. Always **stop (deallocate)** VMs when not in use. Delete the entire resource group when done to avoid any charges.

---

## Step 1 — Design the VNet + 3 Subnets

1. Go to [portal.azure.com](https://portal.azure.com) and sign in.
2. Search **Virtual networks** in the top bar → click **+ Create**.
3. Set:
   - **Resource group:** Create new → `rg-3tier-lab`
   - **Name:** `vnet-3tier`
   - **Region:** Pick one close to you (e.g. East Asia)
4. Under the **IP Addresses** tab, set address space to `10.0.0.0/16`.
5. Add 3 subnets by clicking **+ Add subnet** three times:

   | Subnet Name | Address Range |
   |-------------|---------------|
   | `snet-web`  | `10.0.1.0/24` |
   | `snet-app`  | `10.0.2.0/24` |
   | `snet-db`   | `10.0.3.0/24` |

6. Click **Review + Create** → **Create**.

> **Why `/24`?** A /24 subnet gives you 251 usable IPs. Azure reserves 5 addresses per subnet automatically.

---

## Step 2 — Deploy VMs in Each Tier

Repeat the following for each VM. Go to **Virtual machines** → **+ Create → Azure virtual machine**.

| Setting | vm-web | vm-app | vm-db |
|---------|--------|--------|-------|
| Resource group | `rg-3tier-lab` | `rg-3tier-lab` | `rg-3tier-lab` |
| Image | Ubuntu Server 22.04 LTS | Ubuntu Server 22.04 LTS | Ubuntu Server 22.04 LTS |
| Size | `Standard_B1s` | `Standard_B1s` | `Standard_B1s` |
| Subnet | `snet-web` | `snet-app` | `snet-db` |
| Public IP | **Yes** (auto) | **None** | **None** |
| NSG | `nsg-web` (new) | `nsg-app` (new) | `nsg-db` (new) |

> **Important:** `vm-app` and `vm-db` must have **no public IP**. Only `vm-web` faces the internet.

---

## Step 3 — Configure NSG Rules

### nsg-web (inbound)

| Priority | Name | Port | Source | Action |
|----------|------|------|--------|--------|
| 100 | Allow-HTTP | 80 | Any | Allow |
| 110 | Allow-HTTPS | 443 | Any | Allow |
| 120 | Allow-SSH-MyIP | 22 | **Your IP only** | Allow |
| 65500 | DenyAllInBound | Any | Any | Deny *(default)* |

### nsg-web (outbound)

| Priority | Name | Port | Destination | Action |
|----------|------|------|-------------|--------|
| 100 | Allow-To-App | 8080 | `10.0.2.0/24` | Allow |
| 65000 | AllowVnetOutBound | Any | VirtualNetwork | Allow *(default)* |
| 65001 | AllowInternetOutBound | Any | Internet | Allow *(default)* |

### nsg-app (inbound)

| Priority | Name | Port | Source | Action |
|----------|------|------|--------|--------|
| 100 | Allow-Web-To-App | 8080 | `10.0.1.0/24` | Allow |
| 65500 | DenyAllInBound | Any | Any | Deny *(default)* |

> **Note:** Delete any auto-created SSH rule with `Source: Any` on `nsg-app`. The app tier should never be directly reachable from the internet. To manage `vm-app`, SSH into `vm-web` first, then hop from there — this is called a **jump host / bastion pattern**.

### nsg-db (inbound)

| Priority | Name | Port | Source | Action |
|----------|------|------|--------|--------|
| 100 | Allow-App-To-DB | 3306 or 5432 | `10.0.2.0/24` | Allow |
| 65500 | DenyAllInBound | Any | Any | Deny *(default)* |

> **How NSG rules work:** Rules are evaluated from lowest priority number first. Azure has a built-in `DenyAllInbound` at priority 65500 — you only need to explicitly allow what you want.

---

## Step 4 — Assign RBAC Roles

1. Search **Microsoft Entra ID** → **Users → + New user**. Create a user named `dev-user` with a temporary password.
2. Navigate to **vm-web** in the portal → click **Access control (IAM)** in the left menu.
3. Click **+ Add → Add role assignment**.
4. Role: select **Contributor** → click **Next**.
5. Members: click **+ Select members** → search and select `dev-user` → click **Review + assign**.
6. Verify the scope: go to `vm-app` → IAM → **Check access** → type `dev-user` → confirm they show **No access**.

> **Principle of least privilege:** Give each identity only the permissions it needs, on only the resources it needs. Contributor can manage but not delete the resource group itself.

---

## Step 5 — Document the Architecture

1. Go to [app.diagrams.net](https://app.diagrams.net) (draw.io) → **Save to → This device**.
2. Draw a large rectangle for the VNet border. Label it `vnet-3tier (10.0.0.0/16)`.
3. Add 3 subnet rectangles inside: `snet-web`, `snet-app`, `snet-db`.
4. Add VM icons inside each subnet. Use the Azure icon pack: in draw.io, search **Azure** in the shapes panel.
5. Add NSG labels near each subnet. Draw arrows between subnets labeled with port numbers.
6. Add an **Internet** cloud shape connected to `snet-web` only (ports 80/443).
7. Export as PNG: **File → Export as → PNG**.

> **Pro tip:** In draw.io, click the gear icon in the shape panel → **Search Shapes** → search "Azure" to enable the official Azure icon library.

---

## NSG Rule Summary

```
NSG       | Inbound from       | Port        | Action
----------|--------------------|-------------|-------
nsg-web   | Any                | 80, 443     | Allow
nsg-web   | Your IP            | 22          | Allow
nsg-web   | Any other          | *           | Deny
nsg-app   | 10.0.1.0/24        | 8080        | Allow
nsg-app   | Any other          | *           | Deny
nsg-db    | 10.0.2.0/24        | 3306 / 5432 | Allow
nsg-db    | Any other          | *           | Deny
```

---

## Cleanup

When you're done with the lab, delete the entire resource group to avoid ongoing charges:

1. Go to **Resource groups** in the Azure Portal.
2. Select `rg-3tier-lab`.
3. Click **Delete resource group** → type the name to confirm.

This removes all VMs, NSGs, the VNet, and associated resources in one step.

---

## Key Concepts Learned

- **VNet & Subnets** — How to segment a network into isolated tiers using address ranges.
- **NSGs** — Stateful firewall rules evaluated by priority; lowest number wins.
- **Defense in depth** — Each tier only accepts traffic from the tier directly above it.
- **Principle of least privilege** — RBAC scoped to the minimum resource needed.
- **Jump host pattern** — Accessing private VMs by hopping through a bastion/public VM.
- **No public IP = no attack surface** — App and DB tiers are invisible to the internet.

---

## Resources

- [Azure Virtual Network documentation](https://learn.microsoft.com/en-us/azure/virtual-network/)
- [Network Security Groups overview](https://learn.microsoft.com/en-us/azure/virtual-network/network-security-groups-overview)
- [Azure RBAC documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)
- [Azure pricing calculator](https://azure.microsoft.com/en-us/pricing/calculator/)