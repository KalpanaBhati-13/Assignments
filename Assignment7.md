**1.Create a project with different user groups and implement group policies.**

Creating a project with different user groups and implementing group policies is a common task in system administration, DevOps, and cloud platform management (e.g., AWS, Azure, Google Cloud). Here's a general approach, with an example using Linux (for system-level policies) and Azure Active Directory (for cloud-based group policies).



Project: User Group Management with Policies

Objective:

Create a structured user management system where different user groups have defined permissions and access levels.



Environment Options (Choose One):

Option 1: Linux/Unix Server (e.g., Ubuntu)



Option 2: Azure Active Directory (Cloud IAM)



Option 3: Windows with Group Policy Management



I'll detail Option 1: Linux, and briefly mention Option 2: Azure AD at the end.



Option 1: Linux Project - User Groups and Policies

Step 1: Create User Groups

sudo groupadd dev\_team

sudo groupadd qa\_team

sudo groupadd hr\_team

Step 2: Create Users and Assign to Groups

\# Developers

sudo useradd -m -G dev\_team dev\_user1

sudo useradd -m -G dev\_team dev\_user2



\# QA

sudo useradd -m -G qa\_team qa\_user1



\# HR

sudo useradd -m -G hr\_team hr\_user1

Step 3: Create Project Directory Structure

sudo mkdir -p /project/dev

sudo mkdir -p /project/qa

sudo mkdir -p /project/hr

Step 4: Set Permissions (Group Policies)

\# Assign ownership

sudo chown :dev\_team /project/dev

sudo chown :qa\_team /project/qa

sudo chown :hr\_team /project/hr



\# Set permissions

sudo chmod 770 /project/dev

sudo chmod 770 /project/qa

sudo chmod 770 /project/hr

Only group members can access their respective directories.



Step 5: Policy Example - Read-Only for QA

If you want QA team to only read files from dev team:

\# Add qa\_user1 to dev\_team group as secondary

sudo usermod -a -G dev\_team qa\_user1



\# Set read-only for group

sudo chmod 750 /project/dev

Step 6: Audit Logs (Optional - Track Usage)

Install and configure auditd:

sudo apt install auditd

sudo auditctl -w /project/dev -p war -k dev\_audit



Option 2: Azure AD – Group Policies (Brief Overview)

1\. Create Groups in Azure AD:

DevTeam, QATeam, HRTeam



2\. Assign Users to Groups via Azure Portal or PowerShell

powershell

Add-AzureADGroupMember -ObjectId <GroupID> -RefObjectId <UserID>



3\. Configure Access Policies:

Use Azure Role-Based Access Control (RBAC)



Assign roles (Reader, Contributor) to groups for resources like VMs, Key Vault, Storage



Deliverables for Project Submission:

Documented group creation commands



List of users and their groups



Permissions table



Screenshots (or CLI output)



Optional: Audit logs or monitoring setup.



**2.Apply branch policies such that only the project administrator can access the master branch; contributors cannot.**

To apply branch policies where only the project administrator can access (push to) the master branch and contributors cannot, you typically use Git-based version control systems along with Git hosting platforms like:



GitHub



GitLab



Azure DevOps



Bitbucket



Below is how you can implement this policy, with step-by-step instructions for GitHub, followed by equivalent notes for GitLab and Azure DevOps.



Goal:

master (or main) branch should be write-protected.



Only project admins/maintainers can push or merge into it.



Contributors must work on feature branches and create Pull Requests.



Option 1: GitHub – Restrict master branch access

Step 1: Create Roles

Ensure:



You (admin) have Owner or Admin role in the repository.



Contributors are added with the Write or Triage permission level (not Admin).



Step 2: Enable Branch Protection Rules

Go to your GitHub repository.



Navigate to Settings > Branches.



Under Branch protection rules, click "Add rule".



Fill in:



Branch name pattern: master (or main)



Enable the following options:



Require pull request reviews before merging



Require status checks to pass before merging (optional)



Include administrators (keep unchecked if admin should bypass protection)



Restrict who can push to matching branches



Under Restrict who can push to matching branches, choose:



Only admin user or group



Click Create.



Now:

Contributors cannot push directly to master.



They must create a PR (Pull Request).



Admins can review, approve, and merge.



Option 2: GitLab

Go to your project > Settings > Repository > Protected Branches.



For the master branch:



Set Allowed to push: Maintainers (or specific users)



Set Allowed to merge: Maintainers



Save changes.



Option 3: Azure DevOps

Navigate to Repos > Branches.



Click on the three dots next to master → Branch policies.



Configure:



Require a minimum number of reviewers



Check for linked work items (optional)



Limit merge types



Restrict push access (only project admin)



You can also configure permissions from Project Settings > Repositories > Security.



Test Policy (Recommended)

Log in as a contributor



Try: git push origin master



Should fail with permission error



Contributor pushes to feature/new-ui



Admin reviews and merges via Pull Request.



**3.Apply branch security and locks.**

To apply branch security and locks, you can restrict access and lock branches to prevent accidental or unauthorized changes—critical for protecting the master or main branch in any Git workflow.



What You’ll Achieve:

Prevent force-push, deletion, or unauthorized access.



Apply branch locks to freeze a branch completely.



Set granular permissions using your Git hosting platform.



I'll guide you through how to do this in the three most common platforms:



GitHub – Apply Branch Security and Locks

Step 1: Enable Branch Protection

Go to your repo → Settings > Branches.



Under Branch protection rules, click Add rule.



Set the branch name pattern to: master (or main).



Enable Security Features:

Check the following boxes:



Require pull request reviews before merging



Require status checks to pass before merging



Include administrators (if even admins must follow rules)



Restrict who can push to matching branches



Choose only Admins or a specific team



Lock Branch (Hard Lock)

GitHub doesn’t support true branch locking through UI, but you can simulate it:



\# Lock the branch to prevent any changes

git branch --set-upstream-to=origin/master master

git push --force --delete origin master   # Dangerous! Use only if resetting.

Instead, use the GitHub API or CLI to lock the branch:



gh api \\

&nbsp; -X PUT \\

&nbsp; -H "Accept: application/vnd.github+json" \\

&nbsp; /repos/OWNER/REPO/branches/master/protection \\

&nbsp; -f required\_status\_checks='{"strict":true,"contexts":\[]}' \\

&nbsp; -f enforce\_admins=true \\

&nbsp; -f required\_pull\_request\_reviews='{"required\_approving\_review\_count":1}' \\

&nbsp; -f restrictions='{"users":\["admin-username"],"teams":\[]}'

GitLab – Protect \& Lock Branch

Go to Settings > Repository > Protected Branches



Find master branch



Set:



Allowed to push: No one or Maintainers



Allowed to merge: Maintainers



To lock the branch completely:



Set "No one" for both push and merge.



Azure DevOps – Branch Security and Locks

Go to Repos > Branches



Select the master branch → Click ... > Branch policies



Enable:



Require a minimum number of reviewers



Check for linked work items



Limit merge types



Prevent deletion



Block direct pushes



To lock a branch:



Go to Project Settings > Repos > Security



Select the repo → Permissions



Set Deny on Contribute, Force push, and Delete for everyone except admins.



**4.Apply branch filters and path filters.**

Applying branch filters and path filters is essential for fine-tuning automation workflows, CI/CD pipelines, and access control in your version control system. They help you trigger actions or enforce rules only when certain branches or files/folders are involved.



Filter Type	Purpose

Branch Filters	Target specific branches (e.g., only master, or all feature/\*)

Path Filters	Trigger actions only when changes occur in specific files or folders



Common Platforms That Support This:

GitHub Actions



Azure DevOps Pipelines



GitLab CI/CD



Bitbucket Pipelines



I'll show examples for the most used systems.



1\. GitHub Actions — Apply Branch \& Path Filters

Sample: .github/workflows/deploy.yml

name: Deploy



on:

&nbsp; push:

&nbsp;   branches:

&nbsp;     - master           # Branch filter

&nbsp;     - 'release/\*'      # Wildcard pattern

&nbsp;   paths:

&nbsp;     - 'src/\*\*'         # Path filter: only if changes are in /src

&nbsp;     - '!src/test/\*\*'   # Exclude test files



jobs:

&nbsp; deploy:

&nbsp;   runs-on: ubuntu-latest

&nbsp;   steps:

&nbsp;     - uses: actions/checkout@v4

&nbsp;     - run: echo "Deploying changes..."

2\. Azure DevOps Pipelines — Branch \& Path Filters

In azure-pipelines.yml

trigger:

&nbsp; branches:

&nbsp;   include:

&nbsp;     - master

&nbsp;     - release/\*

&nbsp; paths:

&nbsp;   include:

&nbsp;     - src/\*\*

&nbsp;     - README.md

&nbsp;   exclude:

&nbsp;     - docs/\*\*



pr:

&nbsp; branches:

&nbsp;   include:

&nbsp;     - master

&nbsp;     - feature/\*

You can also configure branch/path filters in the Azure DevOps UI under:

Pipelines > Edit > Triggers > Path filters.



3\. GitLab CI/CD — Branch \& Path Filters

Use only / rules:

deploy-job:

&nbsp; script: echo "Deploying app..."

&nbsp; rules:

&nbsp;   - if: '$CI\_COMMIT\_BRANCH == "main"'    # Branch filter

&nbsp;   - changes:

&nbsp;       - src/\*\*

&nbsp;       - config/prod.yml                  # Path filter

Bonus: Use with Branch Security Rules

In GitHub, you can combine these with protected branches, so that:



A job runs only on master



And triggers only when specific files are modified



And merges are blocked unless a job passes.



**5.Apply a pull request.**

To apply a pull request (PR) means to review, approve, and merge changes from one branch (typically a feature branch) into another (usually main or master). This is a collaboration workflow step in Git-based systems like GitHub, GitLab, Azure DevOps, etc.



Steps to Apply a Pull Request (PR)

I'll break it down for the three major platforms:



1\. GitHub – Apply a Pull Request

Step 1: Create a PR

Navigate to your GitHub repository



Click "Pull requests" > "New pull request"



Select:



Base: the target branch (e.g., master)



Compare: your feature branch (e.g., feature/login-page)



Click "Create pull request"



Add title \& description



Step 2: Review and Approve

Admins/reviewers check:



Code diffs



CI/CD checks (GitHub Actions, status checks)



Click "Approve" or request changes (if needed)



Step 3: Merge the PR

After approval:



Click "Merge pull request"



Choose merge type:



Merge commit



Squash \& merge



Rebase \& merge



Click Confirm merge



Now the PR is applied! The changes are part of master.



2\. GitLab – Merge a Merge Request (MR)

Go to Merge Requests



Click your MR (same as a PR in GitLab)



Review, comment, and approve



Click “Merge”



You can configure:



Approvers



Pipeline success required before merge



3\. Azure DevOps – Apply a Pull Request

Go to Repos > Pull requests



Click the relevant PR



Review the code



Ensure policies are met (reviewers, pipeline passed)



Click "Complete" to merge



You can also enable auto-complete to merge automatically when checks pass.



Pull Request via Command Line (Git)

If you prefer command line with GitHub:

\# Push your feature branch

git push origin feature/login



\# Create a pull request via GitHub CLI

gh pr create --base master --head feature/login --title "Login Feature" --body "Adds login functionality"

To merge:

gh pr merge 123 --merge    # Use PR number.



**6.Apply branch policy, Apply branch security.**

Great! You're asking to apply branch policy and branch security — both are critical for protecting production branches like master or main from unwanted changes.



Here’s a complete breakdown of how to apply them on GitHub, Azure DevOps, and GitLab.



1\. What is a Branch Policy?

A branch policy defines rules or conditions for modifying a branch.



Common policies:



Require pull request (PR) before merge



Require reviewer approval



Require successful builds/tests



Restrict who can push



Prevent force-push \& deletion



2\. What is Branch Security?

Branch security focuses on permissions:



Who can read, write, or delete the branch



Preventing unauthorized push or force-push



Preventing branch deletion



A. GitHub – Apply Branch Policy \& Security

Step 1: Open Repo Settings

Go to your repo → Settings → Branches → Add a Branch Protection Rule.



Step 2: Add Branch Protection Rule for master or main

Enable These:

Require pull request before merging



Require approvals (e.g., 1–2 reviewers)



Require status checks (CI tests) to pass



Include administrators (optional but recommended)



Restrict who can push:



Only admins or specific users/teams



Prevent force pushes



Prevent branch deletion



This secures the branch and enforces policies.



B. Azure DevOps – Branch Policies \& Security

Step 1: Go to Repo → Branches

Click ... next to master



Choose "Branch Policies"



Step 2: Set Policies

Require a minimum number of reviewers (e.g., 2)



Check for linked work items (optional)



Limit merge types (e.g., squash only)



Require successful build before merging



Automatically include reviewers (optional)



Step 3: Apply Security

Go to:



Project Settings > Repositories > Security



Choose your repository



Select Contributors group or individuals



Set these permissions for master:



Deny: Contribute



Deny: Force push



Deny: Delete branch



C. GitLab – Protected Branches

Step 1: Settings → Repository → Protected Branches

Find master or main



Set:



Allowed to push: Maintainers only



Allowed to merge: Maintainers only



Lock the branch (optional)



Summary Table

Platform	Require PR	Restrict Push	Require CI	Prevent Deletion	Reviewer Approvals

GitHub		Yes	Yes	Yes	Yes

Azure DevOps	Yes	Yes	Yes	Yes	Yes

GitLab		Yes	Yes	Yes	Yes



Example Scenario Applied

Branch: master



Only admins can push



PR with 2 reviewers required



Tests must pass



No force push or delete allowed



Contributors can only work via PR to master.



**7.Apply triggers in build and release.**

Applying triggers in build and release pipelines helps you automate workflows in CI/CD — so builds and deployments run when specific events happen (e.g., a push, PR, or schedule).



Here's how to apply triggers for both Build and Release pipelines on common platforms:



Platforms Supported

We'll focus on:



Azure DevOps



GitHub Actions



(Optional: GitLab CI/CD, on request)



A. Azure DevOps – Triggers in Build \& Release Pipelines

1\. Build Pipeline Triggers

Triggers go in azure-pipelines.yml.



Trigger on Branch Push (default)

trigger:

&nbsp; branches:

&nbsp;   include:

&nbsp;     - master

&nbsp;     - develop

Exclude Branches

trigger:

&nbsp; branches:

&nbsp;   exclude:

&nbsp;     - feature/\*

Trigger on Pull Requests

pr:

&nbsp; branches:

&nbsp;   include:

&nbsp;     - master

&nbsp;     - develop

Trigger on File Path Changes

trigger:

&nbsp; paths:

&nbsp;   include:

&nbsp;     - src/\*\*

&nbsp;     - web/\*\*

&nbsp;   exclude:

&nbsp;     - docs/\*\*

Scheduled Build

schedules:

&nbsp; - cron: "0 3 \* \* 1"  # Every Monday at 3 AM UTC

&nbsp;   displayName: Weekly Build

&nbsp;   branches:

&nbsp;     include:

&nbsp;       - master

&nbsp;   always: true

2\. Release Pipeline Triggers

Release pipelines are GUI-based in Azure DevOps, but here's how you can trigger them:



Trigger from a Build Pipeline

Go to Pipelines > Releases > Edit



Click Artifacts



Enable: Continuous deployment trigger



It triggers release when the linked build artifact is updated



Schedule Release

Go to Stage > Pre-deployment conditions



Click the clock icon to enable scheduled release



Trigger by Branch (in Artifact settings)

In the Artifact source → Select Default version: Latest from specific branch



Choose: master, develop, etc.



B. GitHub Actions – Triggers in Build (Workflow) \& Deploy

GitHub combines build + release into workflows. Use the on: field to configure triggers.



Example: .github/workflows/ci-cd.yml

name: Build and Deploy



on:

&nbsp; push:

&nbsp;   branches:

&nbsp;     - main

&nbsp;     - develop

&nbsp;   paths:

&nbsp;     - 'src/\*\*'

&nbsp; pull\_request:

&nbsp;   branches:

&nbsp;     - main

&nbsp; schedule:

&nbsp;   - cron: '0 0 \* \* 0'  # Every Sunday at midnight

Manual Trigger (Workflow Dispatch)

on:

&nbsp; workflow\_dispatch:

&nbsp;   inputs:

&nbsp;     environment:

&nbsp;       type: choice

&nbsp;       description: Choose environment

&nbsp;       options:

&nbsp;         - staging

&nbsp;         - production.



**8.Apply gates to the pipeline.**

Applying gates to a pipeline in Azure DevOps (or equivalent systems) helps automate control checks before or after deployments. Gates are typically used in Release pipelines to ensure that certain conditions are met before a deployment can proceed.



Gates are checks or validations that run before or after a stage in a pipeline. They prevent deployment until certain external or internal conditions are fulfilled.



Common Gates in Azure DevOps

Gate Type	Purpose

Azure Monitor alerts	Wait until no alerts exist for a target resource

REST API check	Call external service to approve/reject

Query Work Items	Block unless specific work item criteria are met

Check Azure DevOps Query	Validate that specific items exist or not

Invoke Azure Functions	Use custom logic for approval



How to Apply Gates in Azure DevOps Pipeline

Step 1: Open Release Pipeline

Go to Pipelines > Releases



Select or create a release pipeline



Click the Pre-deployment conditions icon (⚙️) on the stage



Step 2: Enable Gates

Click "Pre-deployment conditions"



Scroll to Gates → Toggle Enabled



Configure one or more gates:



Add a REST API URL



Add an Azure Function



Select an Azure Monitor Alert



Add a Query Work Item



Set:



Evaluation interval (e.g., every 5 mins)



Timeout (how long to wait before failing)



Minimum successful evaluations (e.g., 3/3)



Example Use Case

Scenario: Don't allow production deployment if open bugs exist

Create a work item query in Azure DevOps:



Status = Active



Type = Bug



Area Path = Production



Add that query as a gate



Release waits until the bug count is 0



YAML Pipelines (Note):

Gates are not natively supported in YAML pipelines as they are in classic Release pipelines. To replicate:



Use check steps or call custom REST APIs:

\- script: |

&nbsp;   curl -X GET https://myapi.com/check-prod-status

&nbsp; displayName: 'Run Gate Check'

&nbsp; condition: succeeded()

You can also use environments with checks in YAML:



environment:

&nbsp; name: 'production'

&nbsp; resourceName: 'prod'

&nbsp; action: 'deploy'

Then define environment checks (manual approvals, external services) in the Azure DevOps UI.



Best Practices for Gates

Tip	Why It Helps

Use gates before production stages	Avoid breaking changes in live environments

Keep gate evaluation interval short	Faster feedback loop

Use Azure Functions for custom logic	Centralized, reusable decision-making logic

Use REST API gates with external tools	Integrate with tools like ServiceNow, JIRA



Summary

Feature	Classic Release Pipelines	YAML Pipelines

REST API Gates	Built-in	Custom Script

Azure Monitor	Built-in	Use script/extension

Manual Approval	Via Pre-deploy	Via Environments

Work Item Query	Built-in	Not supported.



**9.Apply security on branches so that contributors can create a pull request but cannot directly merge code to master.**

Applying gates to a pipeline in Azure DevOps (or equivalent systems) helps automate control checks before or after deployments. Gates are typically used in Release pipelines to ensure that certain conditions are met before a deployment can proceed.



Gates are checks or validations that run before or after a stage in a pipeline. They prevent deployment until certain external or internal conditions are fulfilled.



Common Gates in Azure DevOps

Gate Type	Purpose

Azure Monitor alerts	Wait until no alerts exist for a target resource

REST API check	Call external service to approve/reject

Query Work Items	Block unless specific work item criteria are met

Check Azure DevOps Query	Validate that specific items exist or not

Invoke Azure Functions	Use custom logic for approval



How to Apply Gates in Azure DevOps Pipeline

Step 1: Open Release Pipeline

Go to Pipelines > Releases



Select or create a release pipeline



Click the Pre-deployment conditions icon on the stage



Step 2: Enable Gates

Click "Pre-deployment conditions"



Scroll to Gates → Toggle Enabled



Configure one or more gates:



Add a REST API URL



Add an Azure Function



Select an Azure Monitor Alert



Add a Query Work Item



Set:



Evaluation interval (e.g., every 5 mins)



Timeout (how long to wait before failing)



Minimum successful evaluations (e.g., 3/3)



Example Use Case

Scenario: Don't allow production deployment if open bugs exist

Create a work item query in Azure DevOps:



Status = Active



Type = Bug



Area Path = Production



Add that query as a gate



Release waits until the bug count is 0



YAML Pipelines (Note):

Gates are not natively supported in YAML pipelines as they are in classic Release pipelines. To replicate:



Use check steps or call custom REST APIs:

\- script: |

&nbsp;   curl -X GET https://myapi.com/check-prod-status

&nbsp; displayName: 'Run Gate Check'

&nbsp; condition: succeeded()

You can also use environments with checks in YAML:



environment:

&nbsp; name: 'production'

&nbsp; resourceName: 'prod'

&nbsp; action: 'deploy'

Then define environment checks (manual approvals, external services) in the Azure DevOps UI.



Best Practices for Gates

Tip	Why It Helps

Use gates before production stages	Avoid breaking changes in live environments

Keep gate evaluation interval short	Faster feedback loop

Use Azure Functions for custom logic	Centralized, reusable decision-making logic

Use REST API gates with external tools	Integrate with tools like ServiceNow, JIRA



Summary

Feature	Classic Release Pipelines	YAML Pipelines

REST API Gates	Built-in	Custom Script

Azure Monitor	Built-in	Use script/extension

Manual Approval	Via Pre-deploy	Via Environments

Work Item Query	Built-in	Not supported.



**10.Use work items in pipelines.**

Using work items in Azure DevOps pipelines helps you track deployments, link builds to tasks or bugs, and enforce traceability between code changes and project management artifacts like user stories or bugs.

Benefit	Description

Link work items to builds	Automatic traceability of code-to-task

Enforce policy compliance	Prevent deployments unless work items are linked

Track deployments by feature	Know what user story or bug was included in a release

Better reporting \& auditing	Work item IDs get tracked in release notes and logs



How to Use Work Items in Azure DevOps Pipelines

1\. Link Work Items Automatically

Azure DevOps automatically links work items to builds/releases if:



Your commit messages mention work item IDs (e.g., #123)



You associate work items manually during PRs or check-ins



Example Commit Message:



git commit -m "Fix login issue #456"

2\. Include Work Items in Build Artifacts

Azure DevOps will:



Include linked work items in the build summary



Track which work items were deployed in each release



You can view this under:



Pipelines > Build > Summary > Associated Work Items



3\. Work Item Checks in Release Pipelines (Gates)

You can add a work item query gate to a release stage.



Steps:

Go to Pipelines > Releases > Edit



Click on the ⚙️ icon (Pre-deployment conditions)



Scroll to Gates > Enable Gates



Add Query Work Items



Query example: “Active Bugs in Production Area”



Block deployment if query returns results



4\. Work Item Validation in YAML Pipelines (Indirect)

YAML pipelines don’t support native work item checks, but you can:



Option A: Use the Azure DevOps CLI or REST API

Get linked work items:



\- task: PowerShell@2

&nbsp; inputs:

&nbsp;   targetType: 'inline'

&nbsp;   script: |

&nbsp;     $buildId = "$(Build.BuildId)"

&nbsp;     $uri = "$(System.TeamFoundationCollectionUri)$(System.TeamProject)/\_apis/build/builds/$buildId/workitems?api-version=7.0"

&nbsp;     Write-Output "Fetching work items for build ID: $buildId"

&nbsp;     Invoke-RestMethod -Uri $uri -Headers @{Authorization = "Bearer $(System.AccessToken)"} | ConvertTo-Json -Depth 10

Option B: Manual enforcement in PR policies

Require that all PRs link at least one work item



Can be enforced via branch policies



Bonus: Release Notes from Work Items

Use the Azure DevOps Generate Release Notes Extension:



Auto-generates markdown/html from linked work items \& commits



Great for changelogs



Example template usage:



\- task: GenerateReleaseNotes@5

&nbsp; inputs:

&nbsp;   templateLocation: 'InLine'

&nbsp;   inlineTemplate: |

&nbsp;     ### Work Items

&nbsp;     {{#forEach workItems}}

&nbsp;     - \[{{id}}] {{fields.'System.Title'}}

&nbsp;     {{/forEach}}

Summary

Feature	Classic Pipeline	YAML Pipeline

Link commits to work items	Auto	Auto

Work item query gate	 Yes	Not native

List work items in summary	Yes	Yes

Generate release notes	Via extension	Via task





















