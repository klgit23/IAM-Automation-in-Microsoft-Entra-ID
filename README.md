# IAM-Automation-in-Microsoft-Entra-ID
Streamline identity lifecycle tasks by automatically provisioning users, generating security groups, and assigning memberships

This project automates user and security group management in Microsoft Entra ID using Java and the Microsoft Graph API. It liberates administrators from repetitive portal-based tasks.

🚩 Bulk user creation by ingesting a structured CSV file  
🚩 Automatic security group creation  
🚩 Accurate group membership assignment for each user  


```

██ Workflow Demonstrated ██

The workflow is: token authentication -> reading csv input -> user creation -> security group creation -> user group assignment

User Input in CSV (including DisplayName, mailNickname, UserPrincipalName, Department, JobTitle)
        →
Automation Script (see attached)
        →
Microsoft Graph API
        → Create Users:  POST /v1.0/users                        
        → Check Existing Group:  GET  /v1.0/groups                       
        → Create Groups:  POST /v1.0/groups                       
        → Assign User to Group:  POST /v1.0/groups/{id}/members/$ref     

```

## Automation script part1
<img width="955" height="821" alt="image" src="https://github.com/user-attachments/assets/ab654c4b-20a2-48b0-b97f-cc38e9633ba2" />

    private static final OkHttpClient client = new OkHttpClient();
    private static final ObjectMapper mapper = new ObjectMapper();

    private static String tenantId;
    private static String clientId;
    private static String clientSecret;
    private static String csvPath;
    private static String accessToken;

    public static void main(String[] args) throws Exception {

        if (args.length < 4) {
            System.out.println("Usage: java AutoProvision input takes 4 parameters: <TenantId> <ClientId> <ClientSecret> <CsvPath>");
            return;
        }

        tenantId = args[0];
        clientId = args[1];
        clientSecret = args[2];
        csvPath = args[3];

        authenticate();
        processCsv();
    }


## Automation script part2
<img width="1182" height="660" alt="image" src="https://github.com/user-attachments/assets/77487113-30fe-49c0-84dd-aaa60d818923" />

    // ---------------------------------------------------------
    // AUTHENTICATION
    // ---------------------------------------------------------
    private static void authenticate() throws IOException {
        System.out.println("Authentication");

        RequestBody formBody = new FormBody.Builder()
                .add("grant_type", "client_credentials")
                .add("scope", "https://graph.microsoft.com/.default")
                .add("client_id", clientId)
                .add("client_secret", clientSecret)
                .build();

        Request request = new Request.Builder()
                .url("https://login.microsoftonline.com/" + tenantId + "/oauth2/v2.0/token")
                .post(formBody)
                .build();

        Response response = client.newCall(request).execute();
        if (!response.isSuccessful()) {
            throw new RuntimeException("Authentication failed: " + response.message());
        }

        Map<String, Object> json = mapper.readValue(response.body().string(), Map.class);
        accessToken = (String) json.get("access_token");

        System.out.println("Token retrieved");
    }


## Automation script part3
<img width="1182" height="857" alt="image" src="https://github.com/user-attachments/assets/c2c66cda-d396-4b3b-b06f-d4cd1aae4e07" />

## Automation script part4
<img width="1182" height="302" alt="image" src="https://github.com/user-attachments/assets/b4d4d119-7587-4e93-82d0-0bbf01d435bf" />

## Automation script part5
<img width="1182" height="836" alt="image" src="https://github.com/user-attachments/assets/e8fc7c2c-2337-49d1-98de-57c633d33879" />

## Automation script part6
<img width="1182" height="841" alt="image" src="https://github.com/user-attachments/assets/a156ae7c-c170-4ef4-950b-f6cc6c775f07" />

## Automation script part7
<img width="1182" height="516" alt="image" src="https://github.com/user-attachments/assets/de540158-2566-4f51-b977-9e3a2b1eb36a" />


 ## 🧠 Skills Demonstrated

- Microsoft Entra ID administration and identity management  
- App registration and service principal configuration  
- Microsoft Graph API (Users & Groups endpoints)  
- Scripting and automation best practices  
- CSV‑driven bulk provisioning workflows  
- Robust error handling and logging  
- Role-Based Access Control (RBAC) fundamentals  
