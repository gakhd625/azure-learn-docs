# What I Learned & Realizations — Azure Storage (Blob, File Share, SAS)

---

## 1. Understanding Azure Storage Accounts

At first, I thought a storage account was just a simple place to upload files like Google Drive.  
But I realized it’s actually the main foundation for all storage services in Azure.

Everything (Blob storage, file shares, SAS tokens, lifecycle rules) all depend on it.

What I learned:
- It’s not just storage, it’s a full storage system
- Choosing LRS affects durability and cost
- Everything starts from this one resource

💡 Realization: A storage account is basically the “base system” for cloud storage in Azure.

---

## 2. Blob Storage Confusion (Then Understanding)

At the beginning, I was a bit confused about containers and blobs.

I thought folders worked like normal file systems, but Azure doesn’t really work that way.

What I learned:
- Containers are just logical groups
- Blobs are the actual files
- There is no real folder structure

I also made mistakes like using the wrong URL or trying to access the root instead of the container.

💡 Realization: Blob storage is more like object storage, not a normal file system.

---

## 3. Using PowerShell vs Portal

I started with the Azure Portal because it was easier to understand visually.

But later I saw that PowerShell (Azure CLI) is way more powerful.

What I learned:
- Portal = good for learning
- CLI = better for real automation
- Commands are reusable and faster once you understand them

💡 Realization: In real jobs, CLI matters more than clicking in the portal.

---

## 4. SAS Tokens (This Was Interesting)

SAS tokens confused me at first, but after testing them, I understood their purpose.

What I learned:
- SAS gives temporary access to storage
- It has an expiry time
- After expiry, access is denied automatically

I also noticed small mistakes like:
- Wrong URL format breaks access
- Root URL doesn’t work, you need container or file path

💡 Realization: SAS tokens are like temporary keys you can share safely.

---

## 5. Azure File Share (Z: Drive)

This part was interesting because it felt like connecting a cloud drive to my PC.

What I learned:
- Azure File Share behaves like a network drive
- You can map it as Z: in Windows
- It uses a storage key for authentication

But I also faced issues:
- Sometimes mounting fails in sandbox environments
- Network ports can block access

💡 Realization: Cloud storage can behave like a local drive if configured correctly.

---

## 6. Lifecycle Rules (Cost Control)

Before this, I didn’t really think about storage costs.

Now I understand Azure can automatically manage storage tiers.

What I learned:
- Files can move from Hot → Cool → Archive automatically
- Rules can delete or move old data
- This helps reduce cost in real environments

💡 Realization: Cloud storage is not just about storing data — it’s also about managing cost.

---

## 7. Errors & Debugging Experience

I ran into several errors like:
- AuthenticationFailed (expired SAS)
- InvalidQueryParameterValue (wrong URL)
- Mounting issues with file share

But I learned that most of these come from:
- Incorrect URLs
- Expired tokens
- Missing permissions

💡 Realization: Most cloud errors are actually small configuration mistakes.

---

## Final Thoughts

This lab helped me understand how Azure storage actually works in real scenarios.

Now I see that:
- Storage accounts are the base of everything
- Blob storage is object-based, not folder-based
- SAS tokens control secure access
- File shares act like network drives
- Lifecycle rules help manage cost automatically

---

## Final Takeaway

Instead of thinking of Azure Storage as “just file storage,”  
I now see it as a **complete system for storing, securing, and managing data in the cloud**.