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

## Automation script part1--Set up

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


## Automation script part2--Authentication

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


## Automation script part3--Load csv input

    // ---------------------------------------------------------
    // LOAD INPUT CSV
    // ---------------------------------------------------------
    private static void processCsv() throws Exception {
        System.out.println("Reading user data");

        CSVReader reader = new CSVReader(new FileReader(csvPath));
        List<String[]> rows = reader.readAll();
        reader.close();

        String[] header = rows.get(0);
        rows.remove(0);

        int success = 0;
        int failed = 0;

        for (String[] row : rows) {
            Map<String, String> user = mapRow(header, row);

            System.out.println("\n" + user.get("DisplayName"));

            try {
                String userId = createUser(user);
                String[] groups = user.get("Groups").split(",");

                for (String g : groups) {
                    String groupId = ensureGroupExists(g.trim());
                    addUserToGroup(userId, groupId);
                }

                success++;

            } catch (Exception ex) {
                System.out.println("Loading user data failed: " + ex.getMessage());
                failed++;
            }
        }

        System.out.println("\nDone.");
        System.out.println(success + "  users loaded correctly");
        System.out.println(failed + "  users loaded incorrectly");
    }

    private static Map<String, String> mapRow(String[] header, String[] row) {
        Map<String, String> map = new HashMap<>();
        for (int i = 0; i < header.length; i++) {
            map.put(header[i], row[i]);
        }
        return map;
    }

## Automation script part4--provision users

    // ---------------------------------------------------------
    // PROVISION USERS
    // ---------------------------------------------------------
    private static String createUser(Map<String, String> u) throws Exception {

        String mailNickname = u.get("UserPrincipalName").split("@")[0];

        ObjectNode body = mapper.createObjectNode();
        body.put("accountEnabled", true);
        body.put("displayName", u.get("DisplayName"));
        body.put("mailNickname", mailNickname);
        body.put("userPrincipalName", u.get("UserPrincipalName"));
        body.put("department", u.get("Department"));
        body.put("jobTitle", u.get("JobTitle"));

        ObjectNode pwd = mapper.createObjectNode();
        pwd.put("forceChangePasswordNextSignIn", true);
        pwd.put("password", u.get("Password"));
        
        body.set("passwordProfile", pwd);

        Request request = new Request.Builder()
                .url("https://graph.microsoft.com/v1.0/users")
                .post(RequestBody.create(body.toString(), MediaType.parse("application/json")))
                .addHeader("Authorization", "Bearer " + accessToken)
                .build();

        Response response = client.newCall(request).execute();
        if (!response.isSuccessful()) {
            throw new RuntimeException("Failure during User creation: " + response.message());
        }

        Map<String, Object> json = mapper.readValue(response.body().string(), Map.class);
        System.out.println("User " + u.get("DisplayName") + " created with ID " + (String) json.get("id"));
        return (String) json.get("id");
    }

## Automation script part5--provision groups

    // ---------------------------------------------------------
    // PROVISION GROUPS
    // ---------------------------------------------------------
    private static String ensureGroupExists(String groupName) throws Exception {

        String url = "https://graph.microsoft.com/v1.0/groups?$filter=displayName eq '" + groupName + "'";

        Request request = new Request.Builder()
                .url(url)
                .get()
                .addHeader("Authorization", "Bearer " + accessToken)
                .build();

        Response response = client.newCall(request).execute();
        Map<String, Object> json = mapper.readValue(response.body().string(), Map.class);

        List<Map<String, Object>> value = (List<Map<String, Object>>) json.get("value");

        if (value.size() > 0) {
            return (String) value.get(0).get("id");
        }

        ObjectNode body = mapper.createObjectNode();
        body.put("displayName", groupName);
        body.put("mailEnabled", false);
        body.put("mailNickname", groupName.replace(" ", ""));
        body.put("securityEnabled", true);

        Request createReq = new Request.Builder()
                .url("https://graph.microsoft.com/v1.0/groups")
                .post(RequestBody.create(body.toString(), MediaType.parse("application/json")))
                .addHeader("Authorization", "Bearer " + accessToken)
                .build();

        Response createResp = client.newCall(createReq).execute();
        Map<String, Object> created = mapper.readValue(createResp.body().string(), Map.class);
        System.out.println("Group " + groupName + " created with ID " + (String) created.get("id"));
        return (String) created.get("id");
    }

## Automation script part6--Group assignment

    // ---------------------------------------------------------
    // ASSIGN USER TO GROUP
    // ---------------------------------------------------------
    private static void addUserToGroup(String userId, String groupId) throws Exception {

        ObjectNode body = mapper.createObjectNode();
        body.put("@odata.id", "https://graph.microsoft.com/v1.0/directoryObjects/" + userId);

        Request request = new Request.Builder()
                .url("https://graph.microsoft.com/v1.0/groups/" + groupId + "/members/$ref")
                .post(RequestBody.create(body.toString(), MediaType.parse("application/json")))
                .addHeader("Authorization", "Bearer " + accessToken)
                .build();

        Response response = client.newCall(request).execute();

        if (!response.isSuccessful()) {
            throw new RuntimeException("Failed to add user to group: " + response.message());
        }

        System.out.println("user " + userId + " assigned to group " + groupId);
    }


 ## 🧠 Skills Demonstrated

- Microsoft Entra ID administration and identity management  
- App registration and service principal configuration  
- Microsoft Graph API (Users & Groups endpoints)  
- Scripting and automation best practices  
- CSV‑driven bulk provisioning workflows  
- Robust error handling and logging  
- Role-Based Access Control (RBAC) fundamentals  
