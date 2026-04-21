#  Azure RBAC Hands-On Lab — Roles, Scope, and Assignments

## 📌 Overview
This lab walks through how to implement Role-Based Access Control (RBAC) in Azure using built-in roles, custom roles, and scoped assignments.

You will:
- Understand built-in roles (Owner, Contributor, Reader)
- Create a test user in Entra ID
- Assign RBAC roles at the resource group level
- Validate access restrictions
- Create and test a custom RBAC role

---

##  Prerequisites
- Active Azure subscription
- Access to Azure Portal (https://portal.azure.com)
- Role: Owner or User Access Administrator

---

##  Lab Architecture

- Resource Group: `rg-rbac-lab`
- Test User: `rbacuser01`
- Role Assignment: Reader (Resource Group scope)
- Custom Role: VM Operator (limited permissions)

---

##  Task 1 — Understand Built-in Roles

| Role        | Permissions |
|------------|------------|
| Owner       | Full access + can assign roles |
| Contributor | Full access (cannot assign roles) |
| Reader      | View-only |

📌 RBAC = **Role + Scope + Assignment**

---

##  Task 2 — Create a Test User

1. Go to **Microsoft Entra ID**
2. Navigate to **Users → New user**
3. Create:
   - Username: `rbacuser01`
   - Name: `RBAC Test User`
   - Auto-generate password
4. Save credentials

---

##  Task 3 — Create Resource Group

1. Go to **Resource Groups**
2. Click **Create**
3. Configure:
   - Name: `rg-rbac-lab`
   - Region: Any (e.g., East US)

---

##  Task 4 — Assign Reader Role

1. Open `rg-rbac-lab`
2. Go to **Access Control (IAM)**
3. Click **Add → Add role assignment**
4. Configure:
   - Role: `Reader`
   - Assign access to: User
   - Select: `rbacuser01`
5. Click **Review + assign**

📌 Scope used: **Resource Group**

---

##  Task 5 — Verify Access

Login as `rbacuser01` (use incognito/private browser)

Test actions:

| Action              | Expected Result |
|---------------------|---------------|
| View resources       | ✅ Allowed |
| Create resources     | ❌ Denied |
| Delete resources     | ❌ Denied |

---

##  Task 6 — Create Custom RBAC Role

1. Go to **Subscriptions → Access Control (IAM)**
2. Click **Roles → + Create custom role**

### Basic Info
- Name: `Custom VM Operator`
- Description: Can view and start VMs

### Permissions
Add the following:

```json
{
  "Actions": [
    "Microsoft.Compute/virtualMachines/read",
    "Microsoft.Compute/virtualMachines/start/action"
  ],
  "NotActions": []
}
```

### Assignable Scope
- Subscription or `rg-rbac-lab`

##  Task 7 — Assign Custom Role

1. Go to rg-rbac-lab
2. Open Access Control (IAM)
3. Click Add role assignment
4. Select:
    - Role: Custom VM Operator
    - Assign to: rbacuser01

##  Task 8 - Test Custom Role
Login again as `rbacuser01`

```bash
Management Group
    ↓
Subscription
    ↓
Resource Group
    ↓
Resource
```
- Permissions inherit downward
- Follow least privilege principle
- Prefer group-based assignments over individual users (best practice)

## Cleanup
Delete:
- Resource Group: rg-rbac-lab
- Test User: rbacuser01