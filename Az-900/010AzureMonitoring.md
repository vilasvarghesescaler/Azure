# Vilas Varghese

# Azure Monitor Basics: Metrics, Logs, Alerts vs AWS CloudWatch
# Topic: Azure Monitor Fundamentals, Metrics, Logs, Alerts, and comparison with AWS CloudWatch Duration: ~75 minutes (introductory depth) Audience: Beginners with some AWS exposure, preparing for AZ-900

Learning Objectives: By the end of this session, learners will be able to:

Explain what Azure Monitor is and how it acts as the umbrella observability service
Distinguish Metrics from Logs and explain when to use each
Explain how Alerts are built from Signals, Conditions, and Action Groups
Map Azure Monitor concepts to their AWS CloudWatch equivalents
Answer AZ-900 exam questions confidently on this topic
1. Cold Open + Roadmap
(Timing: 0:00 - 0:05)

Here's a simple truth that applies to every system you'll ever run in production: you cannot manage what you cannot see. If a VM's CPU spikes to 100% at 3 AM and nobody gets notified, does it matter that Azure "detected" it? Not until someone acts on it.

This is the entire purpose of Azure Monitor, the umbrella service that collects, analyzes, and acts on telemetry from virtually everything running in your Azure environment.

If you've touched AWS, you already have a mental hook: Amazon CloudWatch. CloudWatch collects metrics, stores logs, and fires alarms. Azure Monitor does the same three jobs, metrics, logs, alerts, but organizes them slightly differently, and today we'll map every piece precisely.

graph LR
    A[Azure Monitor: The Umbrella] --> B[Metrics - numeric time-series data]
    A --> C[Logs - detailed structured data]
    B --> D[Alerts - notify and act]
    C --> D

Quick visual: the dashboard analogy

+-------------------------------------------------------+
|                     CAR DASHBOARD                       |
|                                                         |
|   [ Speedometer ]   <-- METRICS (numbers, live)         |
|   [ Fuel Gauge   ]                                      |
|                                                         |
|   [ Trip Computer / Service Log ] <-- LOGS (detailed,   |
|                                        queryable)        |
|                                                         |
|   [ Warning Light: Check Engine ] <-- ALERTS (fires     |
|                                        when threshold    |
|                                        is crossed)       |
+-------------------------------------------------------+

Roadmap for today: Azure Monitor big picture, then Metrics, then Logs, then Alerts, then the AWS CloudWatch comparison, then recap and exam prep.

Exam Tip: AZ-900 treats Azure Monitor as a single umbrella platform, not a single "service" with one dashboard. Metrics, Logs, and Alerts are components within it. Don't confuse "Azure Monitor" with just one of its parts.

2. Azure Monitor: The Big Picture
(Timing: 0:05 - 0:15)

2.1 What is Azure Monitor?
Azure Monitor is a full-stack monitoring service that collects, analyzes, and responds to telemetry from:

Azure resources (VMs, App Services, databases, storage accounts)
Applications (via Application Insights, custom telemetry)
Operating systems (via the Azure Monitor Agent collecting OS-level data)
Other clouds/on-premises (hybrid monitoring is supported too)
2.2 The Architecture
graph TD
    subgraph Sources["Data Sources"]
        VM[Virtual Machines]
        APP[Applications]
        AZRES[Azure Resources]
        OS[Operating System / Guest OS]
        CUSTOM[Custom Sources / APIs]
    end

    Sources --> Metrics[("Metrics Database<br/>numeric time-series")]
    Sources --> Logs[("Log Analytics Workspace<br/>structured log data")]

    Metrics --> ME[Metrics Explorer]
    Logs --> KQL[KQL Queries / Log Analytics]

    Metrics --> Alerts[Alert Rules]
    Logs --> Alerts

    Alerts --> AG[Action Groups]
    AG --> Email[Email / SMS / Push]
    AG --> Webhook[Webhook / Function / Logic App]

    ME --> Dashboards[Azure Dashboards / Workbooks]
    KQL --> Dashboards

2.3 The Three Pillars, One-Line Definitions
Pillar	One-Line Definition
Metrics	Lightweight, numerical values collected at regular intervals (e.g., CPU %, requests/sec)
Logs	Detailed, structured or semi-structured records of events, queried using KQL
Alerts	Rules that watch Metrics or Logs and trigger notifications/actions when conditions are met
Think of it like a hospital patient monitor. Metrics are the continuous numeric readouts, heart rate, blood pressure. Logs are the detailed nurse's notes describing what happened and when, in full narrative detail. Alerts are the beeping alarm that goes off when a reading crosses a dangerous threshold.

Exam Tip: A very common AZ-900 distractor is asking you to identify which pillar handles a scenario. Numeric/lightweight/near-real-time → Metrics. Detailed/queryable/text-based → Logs. Notification/automated-response → Alerts.

3. Metrics Deep Dive
(Timing: 0:15 - 0:30)

3.1 What Are Metrics?
Metrics are numerical values describing some aspect of a system at a particular point in time, collected at regular intervals to form a time-series. Examples: CPU percentage, disk IOPS, network bytes in/out, HTTP request count.

3.2 Platform Metrics vs Custom Metrics
Type	Source	Example
Platform Metrics	Automatically emitted by Azure resources, no configuration needed	VM CPU %, Storage account transactions, App Service response time
Custom Metrics	Emitted by your application code or Application Insights SDK	"Number of items added to cart," "Login failures per minute"
Exam Tip: Platform metrics require zero setup, they exist automatically the moment you create a resource. This is a frequently tested fact: metrics are collected "out of the box."

3.3 The Metrics Pipeline
graph LR
    Resource[Azure Resource<br/>e.g. Virtual Machine] --> Emit[Emits Platform Metric<br/>e.g. Percentage CPU]
    Emit --> Store[("Azure Monitor<br/>Metrics Database")]
    Store --> Explorer[Metrics Explorer<br/>chart and visualize]
    Store --> AlertRule[Feed into an Alert Rule]

3.4 Retention Timeline
Day 0 -------------------------------------- Day 93
  |  Metric data actively queryable in       |
  |  Metrics Explorer / used by Alerts       |
  |------------------------------------------|
                                              |
                                     Beyond Day 93:
                                     data no longer available
                                     UNLESS routed to Log
                                     Analytics Workspace via
                                     a Diagnostic Setting

Platform metrics are typically retained for 93 days by default in the metrics database, which is optimized for fast, lightweight, near-real-time queries, not long-term historical analysis (for that, you'd route metrics into Logs via diagnostic settings).

Exam Tip: Metrics are optimized for speed, not long-term storage. If a question mentions "long-term retention and complex querying," the answer is pointing toward Logs, not Metrics.

3.5 Portal Walkthrough: View VM CPU in Metrics Explorer
Sign in to portal.azure.com
Navigate to a running Virtual Machine
In the left menu, under Monitoring, select Metrics
In the Metric dropdown, select Percentage CPU
Choose an Aggregation (Average, Max, Min)
Adjust the time range (last hour, last 24 hours)
Observe the live chart, this is a platform metric being visualized directly, no setup required
4. Logs Deep Dive
(Timing: 0:30 - 0:45)

4.1 What Are Logs in Azure Monitor?
Logs are structured or semi-structured records of events, richer and more detailed than metrics, but heavier and queried rather than charted instantly. They live inside a Log Analytics Workspace, a container where log data is collected and indexed for querying.

4.2 Types of Logs
Log Type	What It Captures	Configuration Required?
Activity Logs	Subscription-level events: who created/modified/deleted a resource, and when (control-plane operations)	No, automatic
Resource Logs (Diagnostic Logs)	Detailed operational data emitted by a specific resource (e.g., a Storage Account's read/write operations, a Key Vault's access requests)	Yes, via Diagnostic Settings
Application Logs	Custom telemetry from your application code, typically via Application Insights	Yes, via SDK/instrumentation
graph TD
    subgraph Log_Sources["Log Sources"]
        Activity[Activity Log<br/>who did what, subscription-level]
        Resource[Resource / Diagnostic Logs<br/>resource-level operational data]
        AppInsights[Application Insights<br/>custom app telemetry]
    end

    Log_Sources --> LAW[("Log Analytics Workspace")]
    LAW --> KQLQuery[KQL Query]
    KQLQuery --> Result[Table / Chart / Dashboard]
    KQLQuery --> LogAlert[Feed into a Log-based Alert Rule]

Exam Tip: Activity Log is enabled automatically for every subscription and free to view for 90 days; it is NOT the same as Resource/Diagnostic Logs, which must be explicitly configured per resource via Diagnostic Settings and routed to a destination like a Log Analytics Workspace.

4.3 KQL: Kusto Query Language
Logs are queried using KQL (Kusto Query Language), a read-only query language built for fast searching and analytics across large log datasets. You do not need to master KQL for AZ-900, but you should recognize its basic shape.

Example: find all Error-level entries in the last 24 hours from a table called AppExceptions:

AppExceptions
| where TimeGenerated > ago(24h)
| where SeverityLevel == "Error"
| project TimeGenerated, ExceptionType, Message
| order by TimeGenerated desc

Read this left to right: start with a table, pipe (|) into a filter, pipe into a projection (choose columns), pipe into a sort order. That's the basic KQL mental model.

graph LR
    T[Table: AppExceptions] --> F["| where TimeGenerated > ago 24h"]
    F --> F2["| where SeverityLevel == Error"]
    F2 --> P["| project columns"]
    P --> O["| order by TimeGenerated desc"]
    O --> R[Result Set]

Exam Tip: AZ-900 will not ask you to write KQL. It may ask you to recognize that KQL is the query language used by Log Analytics/Azure Monitor Logs. Just know the name and the purpose.

4.4 Portal Walkthrough: Run a Basic KQL Query
Navigate to Log Analytics workspaces in the portal (create one first if none exists: + Create, choose a Resource Group and region)
Once inside a workspace, select Logs from the left menu
In the query editor, type a simple query against the built-in Heartbeat table (available if any VM is sending data to this workspace):
Heartbeat
| where TimeGenerated > ago(1h)
| summarize count() by Computer

Click Run
Observe the result table, showing a count of heartbeat signals per connected machine in the last hour
5. Alerts Deep Dive
(Timing: 0:45 - 0:55)

5.1 Alert Rule Anatomy
Every alert rule in Azure Monitor is built from three components:

graph LR
    Signal[Signal<br/>the metric or log being watched] --> Condition[Condition<br/>threshold or logic that triggers the alert]
    Condition --> ActionGroup[Action Group<br/>what happens when triggered]
    ActionGroup --> Notify[Email / SMS / Push / Webhook / Function / Logic App]

Signal + Condition + Action Group = Alert Rule. This formula mirrors the RBAC formula from the Identity lecture (Principal + Role + Scope), Azure loves this three-part composition pattern.

5.2 Types of Signals
Signal Type	Example
Metric signal	"Percentage CPU > 80% for 5 minutes"
Log signal	A KQL query returns more than 10 error rows in the last 15 minutes
Activity Log signal	"A Virtual Machine was deleted"
5.3 Action Groups
An Action Group is a reusable collection of notification and automation actions. Instead of configuring notification details separately for every alert rule, you define an Action Group once (e.g., "Notify DevOps Team": email + SMS + webhook) and attach it to as many alert rules as needed.

graph TD
    AG[Action Group: DevOps-Alerts] --> N1[Email: oncall@company.com]
    AG --> N2[SMS: on-call phone]
    AG --> N3[Webhook: trigger remediation script]
    AG --> N4[Azure Function: auto-scale action]

    Rule1[Alert Rule: High CPU] --> AG
    Rule2[Alert Rule: Failed Logins] --> AG
    Rule3[Alert Rule: VM Deleted] --> AG

Action Type	Example
Notification	Email, SMS, push notification to Azure mobile app, voice call
Automation	Trigger an Azure Function, Logic App, Webhook, or Automation Runbook
5.4 Alert States
stateDiagram-v2
    [*] --> New: Condition triggers
    New --> Acknowledged: Team reviews issue
    Acknowledged --> Closed: Issue resolved
    Closed --> [*]

5.5 Portal Walkthrough: Create a Simple Metric Alert
Navigate to a Virtual Machine → Monitoring → Alerts
Click + Create → Alert rule
Under Scope, confirm the VM is selected
Under Condition, click Add condition, choose Percentage CPU, set threshold Greater than 80, aggregation Average, evaluated over 5 minutes
Under Actions, click Select action group, either choose an existing one or create a new one: name it DevOps-Alerts, add an email notification
Under Alert rule details, give it a name like High CPU Alert, set Severity (0 = Critical to 4 = Verbose)
Click Create alert rule
Now, whenever CPU exceeds 80% for 5 minutes, the configured Action Group notifies the team automatically.

Exam Tip: Remember the exact composition: Alert Rule = Signal + Condition + Action Group. A frequent exam trap: asking what triggers the notification, the answer is the Action Group attached to the rule, not the alert rule itself sending the notification directly.

6. Azure Monitor vs AWS CloudWatch: The Comparison
(Timing: 0:55 - 1:05)

6.1 Structural Overview
graph TB
    subgraph AWS_CW["AWS: CloudWatch"]
        CWM[CloudWatch Metrics] --> CWA[CloudWatch Alarms]
        CWL[CloudWatch Logs] --> CWI[CloudWatch Logs Insights]
        CWL --> CWA
        CWA --> SNS[SNS Topic]
    end

    subgraph Azure_Mon["Azure: Azure Monitor"]
        AZM[Azure Monitor Metrics] --> AZAlert[Alert Rules]
        AZL[Log Analytics / Azure Monitor Logs] --> KQLQ[KQL Queries]
        AZL --> AZAlert
        AZAlert --> AG2[Action Groups]
    end

6.2 Concept-by-Concept Mapping
AWS Concept	Azure Equivalent	Key Difference
CloudWatch Metrics	Azure Monitor Metrics	Both are numeric time-series, automatically collected for native resources
CloudWatch Logs	Log Analytics Workspace (Azure Monitor Logs)	Azure's version is queried using KQL; AWS uses CloudWatch Logs Insights query syntax
CloudWatch Logs Insights	KQL queries in Log Analytics	Different query languages, similar analytical purpose
CloudWatch Alarms	Azure Monitor Alert Rules	Both watch a metric/log and trigger on a threshold/condition
SNS Topic (notification target for an Alarm)	Action Group	Both are reusable notification/automation targets attached to alert/alarm rules
CloudTrail (API/control-plane audit log)	Activity Log	Both automatically record who-did-what at the account/subscription level, no configuration required
CloudWatch Dashboards	Azure Dashboards / Workbooks	Both provide customizable visualization surfaces
6.3 The Most Important Interview/Exam-Ready Line
"Azure Monitor is the umbrella platform combining Metrics, Logs, and Alerts, closely mirroring how CloudWatch combines Metrics, Logs, and Alarms in AWS. The biggest naming difference to remember is that Azure calls its notification mechanism an Action Group, while AWS uses an SNS Topic, and Azure's log query language is KQL, compared to AWS's CloudWatch Logs Insights query syntax."

6.4 Common Misconceptions
Misconception: "Azure Monitor is a single dashboard/service." Reality: it's an umbrella platform composed of multiple sub-features (Metrics, Logs, Alerts, Application Insights, Workbooks).
Misconception: "Metrics and Logs are the same thing, just named differently." Reality: Metrics are lightweight numeric time-series with short retention; Logs are detailed structured records queried via KQL, supporting longer retention and richer analysis.
Misconception: "An Alert Rule sends the notification directly." Reality: the Action Group attached to the Alert Rule is what actually sends notifications or triggers automation.
Exam Tip: If the AZ-900 exam mentions "Azure's equivalent of CloudWatch," expect the answer to be Azure Monitor as the umbrella, not just "Metrics" or just "Logs" alone.

7. Recap: The Complete Mental Model
(Timing: 1:05 - 1:08)

graph TD
    Q1[Question: What is happening right now, numerically?] --> Metrics[Metrics<br/>lightweight, near real-time, 93-day retention]
    Q2[Question: What exactly happened, in detail?] --> Logs[Logs<br/>Log Analytics Workspace + KQL, longer retention]
    Q3[Question: Who should be notified, and how?] --> Alerts[Alerts<br/>Signal + Condition + Action Group]

    Metrics --> Alerts
    Logs --> Alerts
    Alerts --> Action[Notification / Automated Response]

Return to the hospital monitor analogy one final time: Metrics are the continuous vital sign readouts, Logs are the detailed nurse's notes, Alerts are the alarm that goes off and pages the on-call doctor (the Action Group).

8. Exam Tips Quick-Fire Summary
(Timing: 1:08 - 1:10)

Azure Monitor is the umbrella platform, Metrics, Logs, and Alerts are components within it, not separate standalone services.
Platform metrics require zero configuration; they're collected automatically the moment a resource exists.
Metrics are optimized for speed and short-term retention (93 days); Logs support longer retention and richer, structured querying.
Activity Log is automatically enabled per subscription (control-plane, who-did-what); Resource/Diagnostic Logs must be explicitly configured via Diagnostic Settings.
Logs are queried using KQL (Kusto Query Language); AZ-900 expects you to recognize the name and purpose, not write queries.
Alert Rule = Signal + Condition + Action Group.
The Action Group is what actually sends the notification or triggers automation, not the Alert Rule itself.
Azure Monitor is the direct conceptual equivalent of AWS CloudWatch; Action Group maps to SNS Topic, Alert Rule maps to CloudWatch Alarm.
9. Interview Questions (Consolidated Q&A)
(Take-home reference)

Q1: What is Azure Monitor, and how does it relate to Metrics, Logs, and Alerts? A: Azure Monitor is the umbrella observability platform in Azure. Metrics, Logs, and Alerts are its core components: Metrics provide lightweight numeric time-series data, Logs provide detailed structured records queryable via KQL, and Alerts watch both to trigger notifications or automated responses.

Q2: When would you use Metrics versus Logs to diagnose an issue? A: Use Metrics for fast, near-real-time visibility into numeric trends, like confirming CPU spiked at a certain time. Use Logs when you need the detailed "why," such as querying error messages, stack traces, or specific request details around that spike.

Q3: Explain the three components of an Azure Monitor Alert Rule. A: Signal (the metric or log being watched), Condition (the threshold or logic that determines when to trigger), and Action Group (what happens when the condition is met, such as sending an email or triggering a Function).

Q4: What is an Action Group, and why is it designed as a reusable component? A: An Action Group is a reusable bundle of notification/automation actions (email, SMS, webhook, Function, etc.). It's reusable so teams don't have to reconfigure notification details for every single alert rule, one Action Group can be attached to many alert rules across many resources.

Q5: What is the difference between Activity Logs and Resource/Diagnostic Logs? A: Activity Logs automatically capture subscription-level control-plane events (who created, modified, or deleted a resource) with zero configuration. Resource/Diagnostic Logs capture detailed operational data emitted by a specific resource (like Storage Account read/write operations) and must be explicitly configured via Diagnostic Settings to route to a destination like a Log Analytics Workspace.

Q6: How does Azure Monitor compare structurally to AWS CloudWatch? A: Both are umbrella observability platforms combining metrics, logs, and alerting. Azure Monitor's Alert Rule is CloudWatch's Alarm; Azure's Action Group is CloudWatch's SNS Topic; Azure's Log Analytics/KQL is CloudWatch Logs/Logs Insights; Azure's Activity Log is roughly CloudTrail's control-plane audit role combined into the same monitoring umbrella.

Q7: Why might a metric-based alert not be sufficient for a complex production issue, and what would you use instead? A: Metric-based alerts only see numeric thresholds crossing a value; they can tell you CPU is high but not why. For root-cause diagnosis, you'd use a Log-based alert or manual KQL query against Log Analytics to inspect detailed application or system logs correlating with that time window.

Q8: What is KQL and why does Azure use it for Logs instead of a metrics-style query? A: KQL (Kusto Query Language) is a read-only query language optimized for fast searching and analytics across large volumes of structured/semi-structured log data. It's used for Logs because log data is far richer and more varied than the simple numeric time-series structure of Metrics, requiring a more expressive query language to filter, join, and aggregate.

10. Exam-Style Practice Questions (AZ-900 Format)
Question 1: Which Azure Monitor component provides lightweight, numerical, time-series data collected automatically from Azure resources?

A) Logs B) Metrics C) Action Groups D) Activity Log

Answer: B. Metrics are the lightweight numeric time-series data automatically emitted by platform resources. Logs (A) are structured/detailed data; Action Groups (C) are notification mechanisms; Activity Log (D) is a specific type of log for control-plane events.

Question 2: What query language is used to query data in a Log Analytics Workspace?

A) SQL B) KQL C) PromQL D) GraphQL

Answer: B. KQL (Kusto Query Language) is the language used across Azure Monitor Logs/Log Analytics.

Question 3: What are the three components that make up an Azure Monitor Alert Rule?

A) Metric, Dashboard, Workbook B) Signal, Condition, Action Group C) Resource, Policy, Scope D) User, Role, Permission

Answer: B. Signal + Condition + Action Group is the exact formula for building an alert rule.

Question 4: Which Azure Monitor feature automatically records subscription-level events like resource creation, modification, and deletion, without any configuration?

A) Resource Logs B) Application Insights C) Activity Log D) Metrics Explorer

Answer: C. Activity Log captures control-plane, who-did-what events automatically for every subscription.

Question 5: In the AWS-to-Azure comparison, which Azure concept is closest to an AWS SNS Topic used with a CloudWatch Alarm?

A) Log Analytics Workspace B) Action Group C) Metrics Explorer D) Activity Log

Answer: B. Both are reusable notification/automation targets attached to alerting rules.

Question 6: A team needs to investigate detailed error messages and stack traces from an application over the past week. Which Azure Monitor pillar should they use?

A) Metrics B) Logs C) Action Groups D) Alert Rules alone, without any signal source

Answer: B. Logs provide detailed, structured, queryable data suitable for deep investigation, and support longer retention than Metrics.

Question 7: What must be explicitly configured before Resource/Diagnostic Logs from a specific Azure resource are sent to a Log Analytics Workspace?

A) Nothing, it happens automatically like Activity Log B) Diagnostic Settings C) A Conditional Access policy D) An RBAC role assignment

Answer: B. Diagnostic Settings must be explicitly configured per resource to route Resource/Diagnostic Logs to a destination like a Log Analytics Workspace, unlike Activity Log which is automatic.

Question 8: Which statement best describes the relationship between Azure Monitor and AWS CloudWatch?

A) They are unrelated services with no comparable features B) Azure Monitor is the umbrella platform for Metrics, Logs, and Alerts, directly comparable to how CloudWatch combines Metrics, Logs, and Alarms in AWS C) CloudWatch only handles alerting, while Azure Monitor only handles metrics D) Azure Monitor requires AWS CloudWatch integration to function

Answer: B. Both are umbrella observability platforms with directly comparable sub-components, as detailed in the mapping table in Section 6.

The Azure Database Services note (Azure SQL Database, Cosmos DB, PostgreSQL/MySQL vs RDS, DynamoDB) still has a pending clarifying question: should it include the Interview Q&A section like the rest of your series, or stay exam-focused only? Let me know, or say "generate now" to proceed with the Interview Q&A included by default.
