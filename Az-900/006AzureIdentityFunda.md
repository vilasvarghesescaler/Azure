# Written and maintained Vilas Varghese

Session Overview
Topic: Azure Identity Fundamentals, Microsoft Entra ID (Azure AD), Azure RBAC, Conditional Access, and comparison with AWS IAM Duration: ~120 minutes Audience: Beginners with some AWS exposure, preparing for AZ-900 and DevOps/Cloud interviews

Learning Objectives: By the end of this session, learners will be able to:

Explain the role of Microsoft Entra ID (Azure AD) as Azure's identity backbone
Describe how Azure RBAC controls "what" an identity can do, and at what scope
Describe how Conditional Access controls "under what conditions" access is granted
Map Azure identity concepts to their AWS IAM equivalents (and know where there is NO equivalent)
Explain MFA, SSPR, PIM, Managed Identities, and Service Principals
Answer AZ-900 exam questions and interview questions confidently on this topic
1. Cold Open + Session Roadmap
(Timing: 0:00 - 0:05)

Let's start with a question. In any cloud, before you can do anything, the system needs to answer two questions about you:

Who are you? (Authentication)
What are you allowed to do? (Authorization)
Everything we discuss in the next two hours is really just cloud vendors answering these two questions in different ways.

If you've touched AWS, you already know IAM, Identity and Access Management, handles both. AWS bundles users, groups, roles, and policies into one service called IAM.

Azure splits this responsibility across three distinct services, and that split is exactly why this topic confuses beginners and trips people up in interviews. Today we untangle it.

Here's our roadmap for the next 120 minutes:

graph LR
    A[Identity Fundamentals] --> B[Microsoft Entra ID - WHO]
    B --> C[Azure RBAC - WHAT]
    C --> D[Conditional Access - UNDER WHAT CONDITIONS]
    D --> E[MFA / SSPR / PIM / Managed Identity / Service Principal]
    E --> F[AWS IAM vs Azure Comparison]
    F --> G[Recap + Exam + Interview Prep]

Think of it like airport security. Entra ID checks your passport (who you are). RBAC checks your boarding pass and ticket class (what you're allowed to access, economy vs business lounge). Conditional Access is the extra security check that triggers only if certain conditions are met, flying from a high-risk country, using a new device, traveling at an odd hour. All three work together, but they are separate systems doing separate jobs.

Keep that airport analogy in your back pocket. We'll return to it throughout.

Exam Tip: AZ-900 loves testing whether you know that Entra ID, RBAC, and Conditional Access are three separate services with different jobs. A very common wrong-answer trap is treating them as interchangeable.

2. Identity Fundamentals: Core Concepts
(Timing: 0:05 - 0:15)

Before diving into Azure-specific services, let's lock down vocabulary that examiners and interviewers expect you to use precisely.

Authentication vs Authorization
Concept	Question it Answers	Azure Example	AWS Example
Authentication (AuthN)	Who are you?	Signing into Entra ID with username/password + MFA	Signing into AWS Console with IAM user credentials
Authorization (AuthZ)	What are you allowed to do?	Azure RBAC role assignment	IAM policy attached to user/role
A simple way to remember: AuthN comes first (you must prove identity), AuthZ comes second (system checks your permissions). You cannot authorize someone whose identity you haven't authenticated.

Identity Provider (IdP) and Directory Service
An Identity Provider is a system that stores identities and can authenticate them. A Directory Service stores information about users, groups, devices, and organizational structure.

Microsoft Entra ID is both: it's Azure's directory service AND its identity provider. This is a critical distinction from AWS.

Important Beginner Point (AWS comparison): AWS IAM is NOT a directory service. IAM only manages access to AWS resources; it doesn't manage a company's employee directory, devices, or organizational structure the way Entra ID does. If your organization uses AWS and wants a "directory" like Entra ID, you'd typically bring in AWS IAM Identity Center (formerly AWS SSO) or integrate an external directory. This is one of the biggest structural differences between the two clouds, and we will return to it in Section 7.

The Security Principal Concept
In Azure, anything that can request access to a resource is called a security principal. There are four types:

graph TD
    SP[Security Principal] --> U[User - a person]
    SP --> G[Group - collection of users]
    SP --> SVP[Service Principal - identity for an app]
    SP --> MI[Managed Identity - auto-managed app identity]

We'll explore Service Principals and Managed Identities in depth in Section 6, but plant the seed now: not every identity in Azure is a human.

Exam Tip: If a question asks "which of the following can be assigned an RBAC role," the answer set almost always includes User, Group, Service Principal, and Managed Identity, not just "User."

3. Microsoft Entra ID (Azure AD) Deep Dive
(Timing: 0:15 - 0:35)

3.1 What is Microsoft Entra ID?
Microsoft Entra ID, formerly called Azure Active Directory (Azure AD), is Microsoft's cloud-based identity and access management service. Microsoft rebranded Azure AD to Microsoft Entra ID in 2023, but the underlying service and the AZ-900 exam content are unchanged; you will see both names used interchangeably in the wild and on the exam.

Think of Entra ID as the "master phonebook plus bouncer" for your entire organization's cloud identity. It:

Stores your organization's users, groups, and devices
Authenticates users signing into Microsoft 365, Azure Portal, and thousands of SaaS apps (Salesforce, Slack, ServiceNow, etc. via SSO)
Issues security tokens that other services trust
3.2 The Tenant Concept
A tenant is a dedicated, isolated instance of Entra ID that an organization gets when it signs up for a Microsoft cloud service. One tenant usually equals one organization.

graph TD
    Tenant[Entra ID Tenant - contoso.onmicrosoft.com] --> Users[Users]
    Tenant --> Groups[Groups]
    Tenant --> Apps[App Registrations / Enterprise Apps]
    Tenant --> Devices[Registered Devices]
    Tenant --> Domains[Custom Domains]

    Users --> InternalUser[Internal Employee - Member]
    Users --> GuestUser[External Guest - B2B]

Key facts about tenants:

A tenant has a unique Tenant ID (a GUID) and typically a default domain like yourorg.onmicrosoft.com
One Azure subscription trusts exactly one Entra ID tenant for authentication, but one tenant can be trusted by many subscriptions
Multiple tenants can exist for the same company (e.g., separate dev/test tenants), but they are fully isolated from each other unless you explicitly configure federation or B2B collaboration
Exam Tip: A very common AZ-900 question: "How many Azure AD tenants can a single Azure subscription be associated with?" Answer: exactly one. But one tenant can be linked to multiple subscriptions.

3.3 Users and Groups
Users represent individual identities. They can be:
Member (internal): employees created directly in your tenant, or synced from on-premises Active Directory using Azure AD Connect / Entra Connect
Guest (external/B2B): users invited from another organization's Entra ID tenant or even a personal Microsoft/Google account, used for secure collaboration without creating a full internal account
Groups simplify management. Instead of assigning permissions to 50 individual users, you assign them to one group.
Security Groups: used for granting access to resources
Microsoft 365 Groups: used for collaboration (shared mailbox, calendar, files) in addition to access
3.4 External Identities: B2B and B2C
Feature	B2B (Business to Business)	B2C (Business to Consumer)
Purpose	Collaborate with partner organizations	Let external customers sign in to your app
Who signs in	Employees of another company (guest users)	End customers, public users
Example	Inviting a vendor to your Teams project	A retail app letting customers sign in with Google/Facebook/email
Exam Tip: Remember B2B = Business partners collaborating; B2C = Consumer facing app sign-in. This distinction shows up almost every AZ-900 sitting.

3.5 Entra ID Editions/Licensing
Edition	Key Capabilities
Free	Basic user/group management, on-prem AD sync
Microsoft 365 Apps	Included with M365 subscriptions, some premium features
Premium P1	Conditional Access, dynamic groups, hybrid identities
Premium P2	Everything in P1 + Identity Protection + Privileged Identity Management (PIM)
Exam Tip: Conditional Access requires at least Entra ID P1. PIM requires Entra ID P2. This licensing detail is a frequently tested exam fact, keep it memorized.

3.6 Portal Walkthrough: Create a User and a Group
Step-by-step (follow along in Azure Portal):

Sign in to portal.azure.com
In the search bar, type Microsoft Entra ID and open it
In the left menu, select Users → New user → Create new user
Fill in User principal name (e.g., jane.doe@yourtenant.onmicrosoft.com), Display name, and password options
Click Review + Create, then Create
Now go back to Microsoft Entra ID → Groups → New Group
Choose Group type: Security, give it a name like DevOps-Engineers
Under Members, click No members selected, search for jane.doe, add her
Click Create
You've now created an identity (WHO) and a container for it. Note: at this point, Jane can log in, but she has zero permissions to any Azure resource. That's the job of RBAC, coming up next.

4. Azure RBAC Deep Dive
(Timing: 0:35 - 0:55)

4.1 What is RBAC?
Role-Based Access Control (RBAC) is Azure's authorization system. It answers: now that we know WHO you are (thanks, Entra ID), WHAT are you allowed to do?

RBAC works through three building blocks combined into a role assignment:

graph LR
    SP[Security Principal<br/>User, Group, Service Principal, Managed Identity] --> RA((Role Assignment))
    RD[Role Definition<br/>e.g. Contributor, Reader, Owner] --> RA
    SC[Scope<br/>Management Group, Subscription, Resource Group, Resource] --> RA
    RA --> Result[Access granted at that scope]

Role Assignment = Security Principal + Role Definition + Scope

This is the single most important formula in this entire lecture. If you remember one sentence from RBAC, remember this one.

4.2 Scope: Where Does the Role Apply?
Azure resources are organized in a hierarchy, and RBAC role assignments can happen at any level of it:

graph TD
    MG[Management Group] --> SUB[Subscription]
    SUB --> RG[Resource Group]
    RG --> RES[Individual Resource<br/>e.g. a VM or Storage Account]

Roles assigned higher in the hierarchy are inherited downward. If you're made Contributor at the Subscription level, you automatically get Contributor on every Resource Group and Resource underneath it.

Exam Tip: "Inheritance flows downward, never upward." A role assigned at a Resource Group does NOT give access to the Subscription or other Resource Groups. This exact wording appears often on the exam.

4.3 Built-in Roles You Must Know
Role	What it Allows
Owner	Full access to all resources, INCLUDING the ability to grant access to others (manage RBAC itself)
Contributor	Full access to manage resources, but CANNOT grant access to others
Reader	View resources only, no modifications
User Access Administrator	Manage user access to Azure resources (assign roles), but not manage the resources themselves
A simple comparison table to nail this down:

Capability	Owner	Contributor	Reader
View resources	Yes	Yes	Yes
Create/modify/delete resources	Yes	Yes	No
Grant access to other users	Yes	No	No
Exam Tip: The classic trick question: "A user needs to manage VMs but should NOT be able to grant access to other users. Which role?" Answer: Contributor, not Owner.

4.4 Custom Roles
If built-in roles don't fit (too broad or too narrow), you can create a custom role defining exact permitted Actions and NotActions (explicitly excluded actions), scoped to specific resource types.

This is conceptually similar to writing a custom IAM Policy in AWS with a precise JSON permission set, except in Azure it's called a Role Definition, and it's reusable across scopes.

4.5 Deny Assignments (brief mention)
Azure also supports Deny Assignments, which explicitly block actions regardless of what role assignments grant. These are rare and mostly created by Azure Blueprints/Managed Apps, but know that they exist and that Deny always wins over Allow.

Exam Tip: Just like in AWS IAM (explicit Deny always overrides Allow), Azure RBAC Deny Assignments always override Allow role assignments. This "Deny wins" principle is common to both clouds and a favorite short exam question.

4.6 Portal Walkthrough: Assign a Role
In portal.azure.com, navigate to a Resource Group
Click Access control (IAM) in the left menu (yes, Azure also uses the term "IAM" in the portal for this blade, more on this naming collision in Section 7)
Click + Add → Add role assignment
Under Role, search and select Contributor
Under Members, click + Select members, search for the group DevOps-Engineers created earlier, select it
Click Review + assign
Now every member of DevOps-Engineers, including Jane, can create/modify resources in this Resource Group, but cannot grant that access to anyone else.

Exam Tip: Notice the portal blade is literally called "Access control (IAM)". This is Microsoft's own internal naming; it does NOT mean Azure has a separate "IAM service" like AWS. It's the RBAC configuration screen. Don't let the wording confuse you on the exam or in interviews.

5. Conditional Access Deep Dive
(Timing: 0:55 - 1:10)

5.1 Why Conditional Access Exists
RBAC decides WHAT you can do. But what if the SAME user, with the SAME permissions, is logging in from an unmanaged personal laptop at 3 AM from a country your company has no offices in? Should that be treated the same as logging in from a company laptop, on the corporate network, during business hours?

Conditional Access (CA) is Entra ID's policy engine that adds a layer of contextual, risk-based control on top of authentication. It answers: under what conditions should access be granted, blocked, or challenged further?

5.2 The Signal-Decision-Enforcement Model
graph TD
    subgraph Signals
        S1[User / Group]
        S2[Location / IP]
        S3[Device Platform & Compliance]
        S4[Application being accessed]
        S5[Real-time Sign-in Risk]
    end

    Signals --> Decision{Conditional Access<br/>Policy Evaluation}

    Decision -->|Conditions Met| Controls[Enforce Access Controls]
    Decision -->|Conditions Not Met / High Risk| Block[Block Access]

    Controls --> C1[Require MFA]
    Controls --> C2[Require Compliant Device]
    Controls --> C3[Require Password Change]
    Controls --> C4[Grant Access]

A Conditional Access policy is essentially an if-then statement:

IF [these signals are true] THEN [enforce these controls]

Example in plain English: "IF a user outside the corporate network tries to access the Azure Portal, THEN require MFA."

5.3 Common Signals (Conditions)
User or group membership
Sign-in risk level (calculated by Entra ID Identity Protection, requires P2)
Device platform (Windows, iOS, Android) and compliance state (managed by Intune or not)
Location (trusted IP ranges, countries)
Client application (browser vs mobile app vs legacy authentication protocols)
5.4 Common Controls (Enforcement)
Require multi-factor authentication (MFA)
Require the device to be marked compliant
Require a hybrid Azure AD joined device
Block access entirely
Require terms of use acceptance
Session controls (limit sign-in frequency, restrict downloads in cloud apps)
5.5 Licensing Requirement
Exam Tip: Conditional Access requires Microsoft Entra ID P1 (or P2, since P2 includes everything P1 has). It is NOT available in the Free tier. This is one of the most tested licensing facts on AZ-900.

5.6 Conditional Access vs MFA: A Common Confusion
Beginners often think "Conditional Access" and "MFA" are the same thing. They are not.

MFA is a specific control, one possible outcome/enforcement action
Conditional Access is the policy engine/decision framework that can require MFA (among many other controls) based on conditions
Think of Conditional Access as the judge, and MFA as one possible sentence the judge can hand down, but not the only one.

5.7 Portal Walkthrough: Build a Conditional Access Policy
Navigate to Microsoft Entra ID → Security → Conditional Access
Click + New policy
Name it: Require MFA for all admins
Under Users, select Select users and groups, choose the built-in Directory roles option, select Global Administrator
Under Cloud apps or actions, select All cloud apps
Under Conditions, optionally set locations (e.g., "any location except trusted IPs")
Under Access controls → Grant, check Require multi-factor authentication
Set Enable policy to Report-only first (best practice: test before enforcing), then click Create
Exam Tip: Best practice is to run new Conditional Access policies in Report-only mode before turning them on, so you can see the impact without locking anyone out. Exam questions sometimes test this operational best practice.

6. MFA, SSPR, PIM, Managed Identities, and Service Principals
(Timing: 1:10 - 1:25)

We've covered the three pillars. Now let's quickly nail the supporting cast, features that frequently show up as AZ-900 distractor options or interview rapid-fire questions.

6.1 Multi-Factor Authentication (MFA)
Requires two or more verification methods: something you know (password), something you have (phone/authenticator app), something you are (biometrics). Reduces risk from compromised passwords. Can be enforced directly (per-user MFA, legacy) or, the recommended modern way, via Conditional Access policies.

6.2 Self-Service Password Reset (SSPR)
Lets users reset their own forgotten passwords without calling IT/helpdesk, using verification methods like email, phone, or security questions. Reduces helpdesk load and downtime.

Exam Tip: SSPR requires the user to have registered authentication methods beforehand, and requires at least Entra ID P1 for full functionality (though a limited version exists for cloud-only admin accounts in the Free tier).

6.3 Privileged Identity Management (PIM)
PIM enables Just-In-Time (JIT) privileged access. Instead of a user having Global Administrator rights permanently (a standing security risk), PIM lets them activate the role only when needed, for a limited time window, often requiring approval and MFA.

graph LR
    Eligible[User is "Eligible" for Global Admin] --> Request[User requests activation]
    Request --> Approval{Approval Required?}
    Approval -->|Yes| Approve[Approver grants]
    Approval -->|No| Auto[Auto-activated]
    Approve --> Active[Role Active for limited time e.g. 8 hours]
    Auto --> Active
    Active --> Expire[Role automatically expires]

Exam Tip: PIM requires Entra ID P2. This is a very frequently tested licensing fact, pair it mentally with "Conditional Access = P1, PIM = P2."

6.4 Service Principals
A Service Principal is the identity an application uses to access Azure resources, as opposed to a human user. When you register an application in Entra ID (an "App Registration"), Azure automatically creates a Service Principal for it in the tenant. You then assign RBAC roles to that Service Principal, just like you would to a user.

The challenge: Service Principals authenticate using a secret (password) or a certificate, and someone has to create, store, and rotate that secret securely. This is a common security risk if mismanaged, similar to hardcoding an AWS IAM user's access key in code.

6.5 Managed Identities
A Managed Identity solves the Service Principal secret-management problem. It's an identity automatically managed by Azure for an Azure resource (like a VM or Function App), with no credentials for you to store or rotate, Azure handles it behind the scenes.

Two types:

Type	Lifecycle	Reusable?
System-assigned	Tied to the resource; deleted when resource is deleted	No, one-to-one with the resource
User-assigned	Created as an independent Azure resource	Yes, can be attached to multiple resources
graph TD
    Identity[Azure Identity Types] --> User[User Identity - human, interactive login]
    Identity --> SP[Service Principal - app identity, manual secret/cert management]
    Identity --> MI[Managed Identity - app identity, Azure auto-manages credentials]

    MI --> SAMI[System-Assigned<br/>1:1 with resource lifecycle]
    MI --> UAMI[User-Assigned<br/>Standalone, reusable across resources]

Exam Tip and Interview Favorite: "Why would you use a Managed Identity instead of a Service Principal?" Answer: to eliminate the need to manually store and rotate credentials/secrets. This is one of the highest-value one-liners you can give in an interview.

AWS Comparison: A Managed Identity is conceptually very similar to an AWS IAM Role attached to an EC2 instance (Instance Profile), no hardcoded credentials, temporary credentials automatically supplied by the platform. If you remember EC2 Instance Profiles from AWS, you already understand 80% of Managed Identities.

7. Azure Identity Trio vs AWS IAM: The Big Comparison
(Timing: 1:25 - 1:40)

This is the section that will make interview answers sharp and precise. Let's directly map concepts.

7.1 Structural Difference First
The single biggest conceptual difference: AWS bundles identity + authorization into one service (IAM). Azure splits it into three services (Entra ID for identity, RBAC for authorization, Conditional Access for contextual policy).

graph TB
    subgraph AWS_IAM["AWS: Single Bundled Service"]
        AU[IAM Users/Groups/Roles] --> AP[IAM Policies - JSON]
        AP --> AAccess[Access Decision]
    end

    subgraph Azure_Split["Azure: Three Separate Services"]
        AAD[Entra ID - Identity] --> ARBAC[RBAC - Authorization]
        ARBAC --> ACA[Conditional Access - Contextual Policy]
        ACA --> AAccess2[Access Decision]
    end

7.2 Concept-by-Concept Mapping Table
AWS Concept	Closest Azure Equivalent	Key Difference
IAM User	Entra ID User	Entra ID user also serves as an organization-wide directory identity (email, M365 login, SSO to SaaS apps), not just cloud-resource access
IAM Group	Entra ID Group (Security Group)	Similar concept, Entra ID groups also drive M365 collaboration features
IAM Role (assumed by resources/services)	Managed Identity	AWS IAM Role can be assumed by users too; Azure Managed Identity is specifically for Azure resources/workloads
IAM Role (assumed by external apps/federated identity)	Service Principal	Both represent non-human/application identities
IAM Policy (JSON permission document)	RBAC Role Definition	Azure organizes permissions into named reusable roles; AWS lets you write ad-hoc inline policies OR named managed policies
Permission Boundaries / SCPs (Service Control Policies)	Azure Policy (different from RBAC) + Management Group hierarchy	Not a 1:1 equivalent; Azure Policy governs resource configuration/compliance rules, while SCPs are guardrails on what IAM permissions are usable at all
AWS has NO direct equivalent	Conditional Access	AWS has no single unified "contextual access" engine; similar outcomes require combining IAM policy conditions (like aws:SourceIp), MFA enforcement, and tools like AWS Organizations/SSO
AWS Organizations + IAM Identity Center (SSO)	Microsoft Entra ID (tenant)	Closest match to "a directory covering many accounts/subscriptions with centralized identity"
IAM Identity Center Permission Sets	RBAC Role Assignments at Management Group scope	Both let you grant access consistently across many accounts/subscriptions from one place
7.3 The Most Important Interview Line
If asked "how does Azure's identity model differ from AWS IAM," a strong answer sounds like this:

"AWS bundles identity and authorization into a single IAM service using users, roles, and JSON policies. Azure separates these concerns: Microsoft Entra ID handles identity and authentication as a full directory service, Azure RBAC handles authorization through role assignments scoped across a resource hierarchy, and Conditional Access adds a contextual policy layer, like requiring MFA from untrusted locations, that AWS doesn't have a single unified equivalent for. Azure's Managed Identity is the rough equivalent of an AWS IAM Role attached to an EC2 instance profile, since both eliminate the need for hardcoded credentials."

Memorize a version of this in your own words. It signals strong cross-cloud understanding.

7.4 Common Misconceptions to Avoid
Misconception: "Azure has an IAM service like AWS." Reality: the portal blade is named "Access control (IAM)," but it configures RBAC, it is not a separate standalone service the way AWS IAM is.
Misconception: "Conditional Access is just Azure's version of MFA." Reality: MFA is one possible enforcement action within Conditional Access, not the whole feature.
Misconception: "RBAC roles and Entra ID roles are the same thing." Reality: Entra ID roles (like Global Administrator, User Administrator) control administration of the directory itself (creating users, resetting passwords). Azure RBAC roles (like Owner, Contributor) control access to Azure resources (VMs, storage, networking). These are two separate role systems.
graph LR
    subgraph Directory_Level["Entra ID Roles - manage the directory"]
        GA[Global Administrator]
        UA[User Administrator]
    end
    subgraph Resource_Level["Azure RBAC Roles - manage resources"]
        OW[Owner]
        CO[Contributor]
        RE[Reader]
    end

Exam Tip: This Entra ID role vs Azure RBAC role distinction is one of the most commonly missed points on AZ-900. A Global Administrator does NOT automatically have Owner access to Azure subscriptions (though they can elevate themselves to gain it, a specific documented toggle).

8. Recap: The Complete Mental Model
(Timing: 1:40 - 1:45)

Here is the one diagram that ties everything from today together. If you remember only one visual from this entire session, remember this:

graph TD
    Q1[Question: WHO are you?] --> Entra[Microsoft Entra ID<br/>Authentication + Directory]
    Q2[Question: WHAT can you do?] --> RBAC[Azure RBAC<br/>Role Assignment = Principal + Role Definition + Scope]
    Q3[Question: UNDER WHAT CONDITIONS?] --> CA[Conditional Access<br/>Signals to Decision to Enforcement]

    Entra --> RBAC
    RBAC --> CA
    CA --> Final[Secure, Contextual Access Granted]

    Support[Supporting Cast] --> MFA[MFA - a control CA can enforce]
    Support --> SSPR[SSPR - self-service password reset]
    Support --> PIM[PIM - Just-In-Time privileged access, P2]
    Support --> MI[Managed Identity - passwordless app identity]
    Support --> SP[Service Principal - app identity needing secrets]

Return to our airport analogy one final time:

Entra ID = Passport control (who are you)
RBAC = Your boarding pass and ticket class (what zones you can access)
Conditional Access = The extra security check triggered by risk signals (where are you flying from, is your documentation unusual)
9. Exam Tips Quick-Fire Summary
(Timing: 1:45 - 1:50)

Rapid recap of every Exam Tip mentioned today:

Entra ID, RBAC, and Conditional Access are three separate services with distinct jobs, don't treat them as interchangeable.
A single Azure subscription trusts exactly one Entra ID tenant; one tenant can serve multiple subscriptions.
B2B = collaborating with partner businesses; B2C = customer-facing sign-in.
RBAC inheritance flows downward only (Management Group → Subscription → Resource Group → Resource), never upward.
Contributor can manage resources but cannot grant access to others; Owner can do both.
Deny Assignments always override Allow role assignments.
The "Access control (IAM)" blade in the portal configures RBAC, it is not a separate Azure "IAM service."
Conditional Access requires Entra ID P1 (or P2).
PIM requires Entra ID P2.
Run new Conditional Access policies in Report-only mode before enforcing them.
Managed Identity removes the need to manually manage credentials, unlike a Service Principal.
Entra ID roles (Global Admin) govern the directory; Azure RBAC roles (Owner/Contributor) govern resources, they are not the same system.
10. Interview Questions (Consolidated Q&A)
(Timing: 1:50 - 2:00, or take-home reference)

Q1: What is the difference between Azure AD (Entra ID), Azure RBAC, and Conditional Access? A: Entra ID is the identity and directory service (authentication, WHO you are). RBAC is the authorization system (WHAT you can do, via role assignments scoped to resources). Conditional Access is a policy engine that adds contextual, risk-based conditions (WHEN/HOW access is granted, e.g. requiring MFA from untrusted locations). They are separate services that work together.

Q2: How does Azure RBAC differ from AWS IAM policies? A: AWS IAM policies are JSON documents attached directly to users, groups, or roles, defining allowed/denied actions on resources. Azure RBAC uses reusable named Role Definitions (like Contributor) combined with a security principal and a scope in the resource hierarchy to form a Role Assignment. Azure's model emphasizes scope inheritance across Management Groups, Subscriptions, Resource Groups, and Resources more explicitly than AWS's account/OU structure.

Q3: What is a Managed Identity and why would you use it over a Service Principal? A: A Managed Identity is an Azure AD identity automatically created and managed by Azure for a resource (like a VM or Function App), with credentials fully handled by the platform, no secrets to store or rotate. A Service Principal also represents an application identity but requires you to manage a client secret or certificate yourself. Managed Identities are preferred whenever the workload runs on Azure infrastructure, since they eliminate credential management risk.

Q4: Explain the difference between Entra ID roles and Azure RBAC roles. A: Entra ID roles (e.g., Global Administrator, User Administrator) control permissions over the directory itself, managing users, groups, and tenant-wide settings. Azure RBAC roles (e.g., Owner, Contributor, Reader) control permissions over Azure resources like VMs, storage accounts, and networking. They are two independent role systems; having one does not automatically grant the other.

Q5: What licensing tier is required for Conditional Access, and what about Privileged Identity Management? A: Conditional Access requires Microsoft Entra ID Premium P1 (or P2). Privileged Identity Management (PIM) requires Entra ID Premium P2.

Q6: How would you design least-privilege access for a CI/CD pipeline deploying to Azure? A: Create a Service Principal (or preferably a User-Assigned Managed Identity if the pipeline runs on Azure infrastructure) scoped with a custom RBAC role granting only the specific permissions needed (e.g., deploy to a specific Resource Group), rather than assigning Contributor at the subscription level. Combine with Conditional Access policies restricting sign-ins to trusted IP ranges if applicable.

Q7: What happens if a user has Contributor at the subscription level and Reader at a specific resource group under it? A: Because RBAC is additive (the most permissive combination of assignments wins) and there's no explicit deny here, the user effectively gets Contributor access even at that resource group unless there's an explicit Deny Assignment. Azure RBAC doesn't let a lower-scope assignment "downgrade" a higher-scope one; you'd need a Deny Assignment to truly restrict it.

Q8: What's the difference between a system-assigned and user-assigned managed identity? A: System-assigned is created and tied directly to a single Azure resource's lifecycle; it's deleted automatically when the resource is deleted, and cannot be shared. User-assigned is created as its own independent Azure resource, can be attached to multiple resources simultaneously, and persists independently of any single resource's lifecycle.

Q9: How does Conditional Access support a Zero Trust security model? A: Zero Trust assumes no implicit trust based on network location alone; every access request must be verified explicitly. Conditional Access operationalizes this by evaluating real-time signals (user, device compliance, location, sign-in risk) on every access attempt and enforcing appropriate controls (MFA, blocking, requiring compliant devices), rather than trusting users just because they're on the corporate network.

Q10: If your company uses both AWS and Azure, how might you unify identity management across both? A: Use Microsoft Entra ID as the central identity provider and federate it with AWS IAM Identity Center (or configure SAML-based federation directly with IAM roles) so employees use a single set of corporate credentials (single sign-on) to access both clouds, rather than maintaining separate credential sets in each provider's native identity store.

Q11: What is the principle of least privilege, and how do RBAC scopes support it? A: Least privilege means granting only the minimum permissions necessary to perform a task. Azure RBAC scopes support this by letting you assign roles at the most specific level needed (a single resource or resource group) instead of broadly at the subscription level, limiting the blast radius if credentials are compromised.

Q12: Why can't you assign a Conditional Access policy to a Service Principal the same way as a user in older policy versions, and how is that changing? A: Historically, Conditional Access focused primarily on user sign-ins. Microsoft has since extended Conditional Access to support workload identities (Service Principals and Managed Identities) as a distinct policy target, allowing organizations to apply location/risk-based conditions to non-human identities as well, closing a historic gap in coverage.

11. Exam-Style Practice Questions (AZ-900 Format)
(Take-home practice)

Question 1: Which service provides Azure's directory and authentication capabilities?

A) Azure RBAC B) Azure Policy C) Microsoft Entra ID D) Azure Key Vault

Answer: C. Entra ID is the directory and identity/authentication service. RBAC handles authorization (B and D are unrelated services for resource compliance and secrets management respectively).

Question 2: A user is assigned the Contributor role at the Subscription scope. What can they do at a Resource Group within that subscription?

A) Nothing, scope does not inherit downward B) They automatically have Contributor access there too C) They automatically get Owner access there D) They must be separately assigned Contributor at the Resource Group

Answer: B. RBAC role assignments inherit downward through the resource hierarchy. A is incorrect because inheritance does flow downward. C is incorrect, the role does not escalate. D is incorrect, no separate assignment is needed.

Question 3: What is required to enable Conditional Access in your tenant?

A) Entra ID Free tier B) Entra ID Premium P1 or higher C) Azure RBAC Owner role D) A Global Administrator account

Answer: B. Conditional Access is a premium feature requiring at least P1 licensing. C and D relate to permissions, not licensing, and are not sufficient on their own.

Question 4: Which identity type is automatically managed by Azure and does not require you to handle secrets or certificates?

A) Service Principal B) IAM User C) Managed Identity D) Guest User

Answer: C. Managed Identities have Azure-managed credentials. Service Principals (A) require manual secret/certificate management. B is an AWS concept. D refers to external collaboration users, unrelated to credential management.

Question 5: What best describes the relationship between Azure RBAC and Conditional Access?

A) They are the same service under different names B) RBAC controls what actions a user can perform; Conditional Access controls the conditions under which access is granted C) Conditional Access replaces the need for RBAC D) RBAC only applies to Entra ID roles, not Azure resources

Answer: B. This is the core distinction covered throughout the lecture. A, C, and D misrepresent the relationship.

Question 6: Which of the following is true about Azure AD B2B collaboration?

A) It allows external partner organization users to access your resources as guest users B) It is used exclusively for public consumer-facing applications C) It requires the partner organization to migrate their users into your tenant permanently D) It is a feature only available with Entra ID P2

Answer: A. B2B is for inviting external partner/business users as guests. B describes B2C. C is false, guests remain separate identities. D is false, B2B is broadly available without requiring P2.

Question 7: A company wants users to complete MFA only when signing in from outside the corporate network. What should they configure?

A) A Conditional Access policy with a location condition requiring MFA when outside trusted IP ranges B) Per-user MFA enabled for all users regardless of location C) An RBAC custom role D) A Deny Assignment

Answer: A. This is a textbook Conditional Access use case combining a location signal with an MFA control. B applies MFA universally, not conditionally. C and D are unrelated authorization mechanisms.

Question 8: What is the minimum Entra ID licensing tier required for Privileged Identity Management (PIM)?

A) Free B) Microsoft 365 Apps C) Premium P1 D) Premium P2

Answer: D. PIM specifically requires P2, distinguishing it from Conditional Access, which only requires P1.

Question 9: In the AWS-to-Azure identity mapping, which Azure concept is closest to an AWS IAM Role attached to an EC2 instance profile?

A) Entra ID Group B) Azure RBAC custom role C) Managed Identity D) Conditional Access policy

Answer: C. Both eliminate the need for hardcoded/stored credentials by providing automatically managed, temporary access to the underlying compute resource.

Question 10: Which statement correctly distinguishes Entra ID roles from Azure RBAC roles?

A) They are the same role system with different names B) Entra ID roles manage directory-level administration; Azure RBAC roles manage access to Azure resources C) Azure RBAC roles can only be assigned to Service Principals D) Entra ID roles automatically grant Owner access to all subscriptions

Answer: B. This is the exact distinction emphasized in Section 7.4 as a commonly missed exam point. A, C, and D are all misconceptions explicitly called out during the lecture.
