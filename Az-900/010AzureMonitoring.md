# Vilas Varghese

Session Overview
Topic: Azure Monitor Basics: Metrics, Logs, Alerts (introductory level) vs AWS CloudWatch Duration: ~75 minutes Audience: Beginners with some AWS exposure, preparing for AZ-900

Learning Objectives: By the end of this session, learners will be able to:

Explain what Azure Monitor is and how it acts as the umbrella monitoring service for Azure
Distinguish Metrics from Logs and know when each is used
Explain how Alerts are structured and triggered
Map Azure Monitor concepts to their AWS CloudWatch equivalents
Answer AZ-900 exam questions confidently on this topic
1. Cold Open + Roadmap
(Timing: 0:00 - 0:05)

Here's a simple truth that applies to every system you'll ever run in production: you cannot manage what you cannot see. A VM can be silently running at 98% CPU, a web app can be throwing errors every few minutes, a database can be one disk-full-error away from an outage, and if nobody is watching, nobody knows until a customer complains.

Monitoring is how the cloud gives you eyes on your infrastructure. If you've touched AWS, you've probably heard of CloudWatch, AWS's monitoring backbone. Azure's equivalent is called Azure Monitor, and today we unpack it.

graph LR
    A[Why Monitoring Matters] --> B[Azure Monitor - The Umbrella Service]
    B --> C[Metrics - the numbers]
    C --> D[Logs - the detailed records]
    D --> E[Alerts - the notifications]
    E --> F[Azure Monitor vs AWS CloudWatch]

Think of running a car. Metrics are your dashboard gauges, speed, fuel level, engine temperature, quick numeric readouts updated constantly. Logs are your car's detailed trip computer/service history, a rich record of everything that happened, which you can query later ("show me every time the engine temperature spiked above 110 degrees in the last month"). Alerts are your dashboard warning lights, they watch the gauges or the trip computer and flag you the moment something crosses a threshold you care about.

Keep that dashboard analogy in mind, we'll revisit it throughout.

Exam Tip: AZ-900 tests whether you know Azure Monitor is the umbrella service, Metrics, Logs, and Alerts are components/features within it, not separate standalone products.

2. Azure Monitor: The Big Picture
(Timing: 0:05 - 0:15)

2.1 What is Azure Monitor?
Azure Monitor is Azure's full-stack monitoring service. It collects, analyzes, and acts on telemetry data from your cloud and on-premises environments, helping you understand how your applications and infrastructure are performing, and respond automatically to issues.

It's not a single tool, it's a platform made up of a few core capabilities:

graph TD
    Sources[Data Sources] --> AM[Azure Monitor]

    subgraph Sources_List["Data Sources"]
        S1[Azure Resources - VMs, App Services, Databases]
        S2[Applications - via Application Insights]
        S3[Operating System - Guest OS, via Agents]
        S4[Custom Sources - REST API, Diagnostics]
    end

    AM --> Metrics[Metrics<br/>Numeric time-series data]
    AM --> Logs[Logs<br/>Structured event records]

    Metrics --> Visualize1[Metrics Explorer / Dashboards]
    Logs --> Visualize2[Log Analytics / KQL Queries / Workbooks]

    Metrics --> Alerts[Alerts]
    Logs --> Alerts
    Alerts --> Actions[Action Groups<br/>Email, SMS, Webhook, Function, Logic App]

2.2 The Two Fundamental Data Types
Azure Monitor stores collected telemetry as one of two data types:

Data Type	Nature	Example
Metrics	Numerical values, collected at regular time intervals, lightweight, optimized for near-real-time analysis	CPU percentage every minute
Logs	Structured or unstructured event records, richer detail, queried using a query language	"VM rebooted at 3:42 AM due to Windows Update"
This is the foundational split you must internalize before going deeper: Metrics are for "what's the number right now/over time," Logs are for "what exactly happened and why."

Exam Tip: A common exam distractor asks you to pick the right tool for a scenario. If the question emphasizes "near real-time performance graph," think Metrics. If it emphasizes "investigate root cause of an incident" or "correlate events across multiple resources," think Logs.

3. Metrics Deep Dive
(Timing: 0:15 - 0:30)

3.1 What are Metrics?
Metrics are lightweight, numerical values describing some aspect of a system at a particular point in time, collected at regular intervals to form a time series. Because they're just numbers (not full event records), they're cheap to store and fast to query, making them ideal for near-real-time dashboards and alerting.

3.2 Platform Metrics vs Custom Metrics
Type	Source	Example
Platform Metrics	Automatically emitted by Azure resources, no configuration needed	VM: CPU percentage, Network In/Out; Storage Account: transactions, latency
Custom Metrics	Emitted by your application code or collected via Application Insights/agents	Number of items in a shopping cart, custom business KPI
Exam Tip: Remember that platform metrics require zero setup, they're available the moment you create the resource. This is a frequently tested "what's included by default" fact.

3.3 The Metrics Pipeline
graph LR
    Resource[Azure Resource<br/>e.g. Virtual Machine] --> Emit[Emits Platform Metric<br/>e.g. CPU %]
    Emit --> Store[Stored in Azure Monitor Metrics Database]
    Store --> Explorer[Metrics Explorer<br/>chart & analyze]
    Store --> AlertRule[Used by Metric Alert Rules]
    Store --> Dashboard[Pinned to Azure Dashboards]

3.4 Retention
Platform metrics are retained for 93 days by default in the Azure Monitor metrics store, sufficient for recent trend analysis but not long-term historical archiving. If you need longer retention, you route metrics into a Log Analytics Workspace or storage account via diagnostic settings.

Exam Tip: 93 days is a specific number that has appeared on exams. Don't just remember "a few months," remember the number.

3.5 Portal Walkthrough: View VM CPU Metric in Metrics Explorer
Navigate to a Virtual Machine resource in portal.azure.com
In the left menu, under Monitoring, click Metrics
In the Metric dropdown, select Percentage CPU
Choose an Aggregation (Average, Max, Min)
Adjust the time range (e.g., last 24 hours) using the time picker at the top
Optionally click Pin to dashboard to keep this chart visible on your Azure Dashboard
You now have a live, near-real-time view of CPU utilization, exactly like checking the speedometer on our car dashboard analogy.

4. Logs Deep Dive
(Timing: 0:30 - 0:45)

4.1 What are Logs?
Logs in Azure Monitor are records of events, with structured or semi-structured data containing much richer detail than a metric's single number. Logs answer "what happened, in what order, and why," which metrics alone cannot tell you.

4.2 Log Analytics Workspace
A Log Analytics Workspace is the container where log data is collected, stored, and queried. Think of it as a dedicated database purpose-built for log analytics. Multiple resources (VMs, App Services, Activity Logs) can all send their log data into the same workspace, letting you correlate across resources in a single query.

graph TD
    subgraph Log_Sources["Log Sources"]
        L1[Activity Logs - subscription-level operations]
        L2[Resource Logs / Diagnostic Logs - resource-level operations]
        L3[Agent-collected OS logs - VM performance/events]
        L4[Application Insights - app telemetry]
    end

    Log_Sources --> LAW[Log Analytics Workspace]
    LAW --> KQL[KQL Queries]
    KQL --> Results[Tables, Charts, Workbooks]
    LAW --> LogAlert[Used by Log Search Alert Rules]

4.3 Three Categories of Logs You Must Know
Log Type	What it Captures	Example
Activity Logs	Subscription-level control-plane operations, "who did what, when"	"User X created a VM at 10:03 AM"
Resource Logs (Diagnostic Logs)	Detailed operations performed inside or by a specific resource	"SQL Database query took 4.2 seconds"
Agent-collected Logs	OS-level or application-level events from inside a VM, via the Azure Monitor Agent	"Windows Event Log: Service X failed to start"
Exam Tip: Activity Log is enabled automatically for every subscription and retained for 90 days by default, no setup required. Resource Logs require you to explicitly configure a Diagnostic Setting to send them somewhere (Log Analytics Workspace, Storage Account, or Event Hub). This "automatic vs must-configure" distinction is a classic exam trap.

4.4 KQL: Kusto Query Language (Brief Intro)
Logs are queried using KQL (Kusto Query Language), a powerful, readable query language purpose-built for log/telemetry analysis. You don't need to master KQL for AZ-900, but you should recognize its basic shape.

Simple example query, "show me the 10 most recent Activity Log entries":

AzureActivity
| order by TimeGenerated desc
| take 10

Read it left to right like a pipeline: start with the table AzureActivity, then pipe (|) the results through steps, order by time descending, then take the top 10.

Exam Tip: AZ-900 will not ask you to write KQL. It may ask you to recognize that KQL is the query language used in Log Analytics, that's the extent of depth needed.

4.5 Portal Walkthrough: Run a Basic KQL Query
Navigate to Log Analytics Workspaces in the portal, select your workspace (or navigate to a VM → Logs under Monitoring)
In the query editor, type:
AzureActivity
| order by TimeGenerated desc
| take 10

Click Run
Review the results table, note columns like OperationName, Caller, TimeGenerated
Optionally click Chart to visualize results, or Pin to dashboard
5. Alerts Deep Dive
(Timing: 0:45 - 0:55)

5.1 Why Alerts Exist
Metrics and Logs are passive, they sit there until someone looks at them. Alerts make monitoring proactive: they watch metrics or log query results continuously and notify you (or trigger automation) the moment a defined condition is met, without you having to stare at a dashboard.

5.2 Anatomy of an Alert Rule
Every Azure Monitor alert rule is built from three parts:

graph LR
    Signal[Signal<br/>Metric or Log Query] --> Condition[Condition<br/>Threshold / Logic]
    Condition --> ActionGroup[Action Group<br/>Who/What gets notified]
    ActionGroup --> Notify[Email / SMS / Webhook / Function / Logic App / ITSM]

Alert Rule = Signal + Condition + Action Group

This mirrors the RBAC formula from our Identity lecture (Principal + Role + Scope), same pattern: three building blocks combine into one enforceable rule.

Component	Meaning	Example
Signal	What is being measured	"Percentage CPU" metric, or a KQL query result
Condition	The threshold/logic that triggers the alert	"Average CPU > 80% over 5 minutes"
Action Group	A reusable set of notification/automation actions	Send email to on-call engineer + trigger an Azure Function to auto-scale
5.3 Alert Types
Type	Based On	Example
Metric Alert	Metrics (near real-time)	CPU > 90%
Log Search Alert	A scheduled KQL query against Logs	"More than 5 failed login attempts in 10 minutes"
Activity Log Alert	Specific control-plane events	"Alert me whenever someone deletes a Resource Group"
5.4 Action Groups
An Action Group is a reusable, named collection of notification and automation actions that you attach to one or many alert rules, avoiding the need to reconfigure notification details every time. A single Action Group might include: an email to a distribution list, an SMS to an on-call phone, and a webhook that triggers an automated remediation script.

5.5 Alert States
New: condition just triggered, not yet acknowledged
Acknowledged: someone has seen it and is working on it
Closed: issue resolved
5.6 Portal Walkthrough: Create a Simple Metric Alert
Navigate to a Virtual Machine → Monitoring → Alerts
Click + Create → Alert rule
Under Scope, confirm the VM is selected
Under Condition, click Add condition, choose signal Percentage CPU
Set Threshold: Static, Operator: Greater than, Threshold value: 80, Aggregation granularity: 5 minutes
Click Done
Under Actions, click Select action groups → Create action group
Name it OnCall-Notify, add an email notification with your address
Click Review + create, then Create
Now, any time this VM's CPU exceeds 80% for 5 minutes, you get an email automatically, no manual dashboard-watching required.

Exam Tip: Remember the layered relationship: an Alert Rule monitors a Signal using a Condition, and fires an Action Group. If asked "what notifies the user when an alert fires," the answer is Action Group, not the alert rule itself.

6. Azure Monitor vs AWS CloudWatch: The Comparison
(Timing: 0:55 - 1:05)

6.1 Structural Comparison
Both platforms follow a strikingly similar structure: collect metrics, collect logs, define alerts on top of both. The naming and some architectural details differ.

graph TB
    subgraph Azure["Azure Monitor"]
        AM_M[Metrics] --> AM_A[Alert Rules]
        AM_L[Logs / Log Analytics Workspace + KQL] --> AM_A
        AM_A --> AM_AG[Action Groups]
    end

    subgraph AWS["AWS CloudWatch"]
        CW_M[CloudWatch Metrics] --> CW_A[CloudWatch Alarms]
        CW_L[CloudWatch Logs + Logs Insights] --> CW_A
        CW_A --> SNS[SNS Topics]
    end

6.2 Concept-by-Concept Mapping
Azure Monitor Concept	AWS CloudWatch Equivalent	Key Difference
Azure Monitor (umbrella service)	Amazon CloudWatch	Both are the "umbrella" monitoring service for their respective clouds
Metrics	CloudWatch Metrics	Both are numeric time-series; both offer default platform-level metrics automatically
Metrics Explorer	CloudWatch Metrics Dashboard	Similar charting/visualization capability
Logs / Log Analytics Workspace	CloudWatch Logs / Log Groups	Azure centralizes logs into a queryable Workspace; AWS organizes into Log Groups/Streams
KQL (Kusto Query Language)	CloudWatch Logs Insights Query Language	Both are purpose-built query languages for log analysis, syntax differs
Activity Log	AWS CloudTrail	Subtle but important: Activity Log in Azure is actually closer to CloudTrail (who did what) than to CloudWatch Logs; AWS splits "audit trail" (CloudTrail) from "operational logs" (CloudWatch Logs) into two separate services, Azure keeps both under one Azure Monitor umbrella
Alert Rule (Metric Alert / Log Search Alert)	CloudWatch Alarm	Both follow Signal + Threshold + Notification pattern
Action Group	SNS Topic (often paired with Lambda/Email/SMS subscriptions)	Azure bundles multiple notification channels into one reusable Action Group; AWS typically routes an Alarm to an SNS Topic, which fans out to subscribers
93-day metric retention	15-month CloudWatch metric retention (with resolution reducing over time)	AWS retains raw high-resolution metrics for a shorter window too, but overall retains longer at reduced granularity; exact numbers differ, know Azure's 93 days specifically for the exam
6.3 The Most Important Interview/Exam Line
"Azure Monitor is the umbrella service for observability in Azure, just like CloudWatch is for AWS. Both split telemetry into Metrics (lightweight numeric time series) and Logs (rich structured event data), and both let you define Alerts on top of either. The key difference is that Azure also folds subscription-level audit data, the Activity Log, into the same Azure Monitor umbrella, whereas AWS separates that concern into CloudTrail, keeping CloudWatch focused purely on operational metrics and logs."

6.4 Common Misconceptions to Avoid
Misconception: "Azure Monitor is just for VMs." Reality: Azure Monitor collects telemetry from virtually every Azure resource type, plus applications (via Application Insights) and even on-premises servers.
Misconception: "Metrics and Logs are the same kind of data, just displayed differently." Reality: they are fundamentally different data types (lightweight numeric time-series vs rich structured events) with different retention, cost, and query mechanisms.
Misconception: "An Alert Rule sends the notification directly." Reality: the Alert Rule only detects the condition; the actual notification/automation is delegated to an Action Group.
7. Recap: The Complete Mental Model
(Timing: 1:05 - 1:08)

graph TD
    Q1[Question: What's the number right now?] --> Metrics[Metrics<br/>lightweight, near real-time, 93-day retention]
    Q2[Question: What exactly happened and why?] --> Logs[Logs<br/>Log Analytics Workspace, queried via KQL]
    Q3[Question: Notify me automatically when something crosses a threshold] --> Alerts[Alerts<br/>Signal + Condition + Action Group]

    Metrics --> Alerts
    Logs --> Alerts
    Alerts --> Final[Proactive, automated response]

Back to our car dashboard analogy one final time:

Metrics = the dashboard gauges (speed, fuel, temperature), quick numeric glance
Logs = the trip computer / service history, detailed queryable record of everything that happened
Alerts = the warning lights, triggered automatically when a gauge or a logged event crosses a line you've defined
8. Exam Tips Quick-Fire Summary
(Timing: 1:08 - 1:10)

Azure Monitor is the umbrella service; Metrics, Logs, and Alerts are components within it, not separate products.
Platform metrics are collected automatically with zero configuration; custom metrics require app-level instrumentation.
Metrics are retained for 93 days by default in the Azure Monitor metrics store.
Logs require a Log Analytics Workspace and are queried using KQL.
Activity Log is enabled automatically per subscription (control-plane, "who did what"); Resource Logs require configuring a Diagnostic Setting.
An Alert Rule = Signal + Condition + Action Group.
The Action Group, not the Alert Rule itself, delivers the actual notification/automation.
Metric Alerts, Log Search Alerts, and Activity Log Alerts are the three alert types; know which signal source each uses.
Azure's Activity Log is functionally closer to AWS CloudTrail than to CloudWatch Logs, even though it lives inside Azure Monitor.
9. Interview Questions (Consolidated Q&A)
Q1: What is Azure Monitor, and how does it relate to Metrics, Logs, and Alerts? A: Azure Monitor is the umbrella monitoring platform for Azure. Metrics, Logs, and Alerts are core capabilities within it: Metrics provide lightweight numeric time-series data, Logs provide detailed structured event records queryable via KQL, and Alerts let you define conditions on either that trigger notifications or automated actions through Action Groups.

Q2: When would you use Metrics versus Logs to diagnose a problem? A: Use Metrics when you need a fast, near-real-time view of a numeric trend, like "is CPU spiking right now." Use Logs when you need to investigate root cause, correlate multiple events, or answer detailed "what happened and why" questions, since logs carry much richer contextual detail than a single number.

Q3: What is a Log Analytics Workspace, and why would you centralize multiple resources into one? A: It's the storage and query engine for log data in Azure Monitor. Centralizing multiple resources into one workspace lets you write a single KQL query that correlates events across VMs, applications, and platform logs simultaneously, useful for tracing an incident that spans multiple components.

Q4: Explain the three components of an Alert Rule. A: Signal (what's being measured, a metric or log query), Condition (the threshold or logic that must be met), and Action Group (what happens when the condition is met, like sending an email or triggering automation). All three combine to form a complete, functioning alert rule.

Q5: What's the difference between Activity Logs and Resource (Diagnostic) Logs? A: Activity Logs capture subscription-level control-plane operations (who created/deleted/modified a resource) and are enabled automatically. Resource Logs capture detailed operations happening inside or by a specific resource (like query execution times in a database) and require explicitly configuring a Diagnostic Setting to route them somewhere.

Q6: How does Azure Monitor compare structurally to AWS CloudWatch? A: Both are umbrella observability services splitting telemetry into Metrics and Logs, with Alerts/Alarms built on top. The notable difference is that Azure folds subscription-level audit data (Activity Log) into Azure Monitor, while AWS separates that concern into CloudTrail, keeping CloudWatch focused on operational metrics and logs.

Q7: What is an Action Group, and why is it designed as a reusable object rather than being configured per alert? A: An Action Group is a reusable, named bundle of notification/automation actions (email, SMS, webhook, function, etc.). It's designed as a standalone reusable object so multiple alert rules across different resources can share the same notification configuration, avoiding duplicated setup and making updates (like changing the on-call email) a single change in one place instead of many.

Q8: If a company needs to retain metric data for over a year for compliance/trend analysis, what should they do? A: Since Azure Monitor's built-in metrics store only retains data for 93 days, they should configure a Diagnostic Setting to route metrics (and/or logs) into a Log Analytics Workspace or a Storage Account for long-term retention beyond the default window.

Q9: What's a practical example of choosing a Log Search Alert over a Metric Alert? A: A Metric Alert works well for simple numeric thresholds like "CPU > 80%." A Log Search Alert is better suited for more complex conditional logic that requires querying event data, such as "alert me if there are more than 5 failed login attempts within a 10-minute window," which requires parsing and counting log entries rather than a single numeric signal.

Q10: How would you design an alerting strategy for a production web application on Azure? A: Combine Metric Alerts for fast infrastructure signals (CPU, memory, response time thresholds), Log Search Alerts for application-level anomalies (error rate spikes, failed authentication attempts), and Activity Log Alerts for critical control-plane events (like accidental deletion of a production resource group), all routed through a shared Action Group so the on-call team gets consistent, centralized notifications across all three alert types.

10. Exam-Style Practice Questions (AZ-900 Format)
Question 1: Which Azure service acts as the umbrella platform for collecting metrics, logs, and triggering alerts?

A) Azure Advisor B) Azure Monitor C) Azure Policy D) Microsoft Defender for Cloud

Answer: B. Azure Monitor is the umbrella observability service. The others serve different purposes (recommendations, governance, and security respectively).

Question 2: What type of data are Azure Monitor Metrics best suited for?

A) Rich, structured event records requiring complex queries B) Lightweight numerical values collected at regular time intervals C) Subscription-level audit trails of administrative actions D) Long-term compliance archives

Answer: B. Metrics are lightweight numeric time-series data, ideal for near-real-time dashboards. A describes Logs; C describes Activity Logs; D requires routing data elsewhere since built-in retention is limited.

Question 3: By default, how long are platform metrics retained in the Azure Monitor metrics store?

A) 30 days B) 93 days C) 6 months D) Indefinitely

Answer: B. 93 days is the default retention period for platform metrics in the built-in metrics store.

Question 4: Which query language is used to analyze data in a Log Analytics Workspace?

A) SQL B) KQL (Kusto Query Language) C) PromQL D) GraphQL

Answer: B. KQL is the purpose-built query language for Log Analytics in Azure Monitor.

Question 5: What are the three components that make up an Azure Monitor Alert Rule?

A) Metric, Dashboard, Notification B) Signal, Condition, Action Group C) Resource, Policy, Effect D) Workspace, Query, Chart

Answer: B. An Alert Rule is composed of a Signal (what's measured), a Condition (threshold/logic), and an Action Group (what happens when triggered).

Question 6: Which Azure Monitor log type is enabled automatically for every subscription without additional configuration?

A) Resource Logs B) Diagnostic Logs C) Activity Log D) Custom application logs

Answer: C. The Activity Log is automatically enabled per subscription and captures control-plane operations. Resource/Diagnostic Logs require explicit configuration via a Diagnostic Setting.

Question 7: What actually delivers the email or SMS notification when an alert condition is met?

A) The Alert Rule itself B) The Metrics Explorer C) The Action Group D) The Log Analytics Workspace

Answer: C. The Action Group is the reusable object that defines and delivers notification/automation actions when an alert fires. The Alert Rule only detects the condition.

Question 8: Which AWS service is most functionally comparable to Azure Monitor as a whole?

A) AWS Config B) Amazon CloudWatch C) AWS CloudTrail D) AWS Trusted Advisor

Answer: B. CloudWatch is AWS's umbrella observability service for metrics, logs, and alarms, directly comparable to Azure Monitor. CloudTrail is closer to Azure's Activity Log specifically, not the whole umbrella service.

