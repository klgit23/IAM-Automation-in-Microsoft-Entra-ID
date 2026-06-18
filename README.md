# IAM-Automation-in-Microsoft-Entra-ID
Streamline identity lifecycle tasks by automatically provisioning users, generating security groups, and assigning memberships

This project automates user and security group management in Microsoft Entra ID using Java and the Microsoft Graph API. It liberates administrators from repetitive portal-based tasks.

🚩 Bulk user creation by ingesting a structured CSV file  
🚩 Automatic security group creation  
🚩 Accurate group membership assignment for each user  


```

██ Workflow Demonstrated ██

The workflow is: token authentication -> reading csv input -> user creation -> security group creation -> user group assignment

User Input in CSV
        →
Automation Script
        →
Microsoft Graph API
        → Create Users:  POST /v1.0/users                        
        → Check Existing Group:  GET  /v1.0/groups                       
        → Create Groups:  POST /v1.0/groups                       
        → Assign User to Group:  POST /v1.0/groups/{id}/members/$ref     

```



 ## 🧠 Skills Demonstrated

- Microsoft Entra ID administration and identity management  
- App registration and service principal configuration  
- Microsoft Graph API (Users & Groups endpoints)  
- Scripting and automation best practices  
- CSV‑driven bulk provisioning workflows  
- Robust error handling and logging  
- Role-Based Access Control (RBAC) fundamentals  
