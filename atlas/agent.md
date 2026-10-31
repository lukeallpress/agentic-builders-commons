# District AI Infrastructure Atlas: brief for your AI assistant

> **Human:** paste this page's link (or this whole text) into your work AI assistant (Gemini, Copilot, ChatGPT, Claude, etc.) and say *"Follow this brief for our district."* It works best in an assistant connected to your email, files and chat.

## What this is

The Agentic Builders Committee, a nationwide group of K-12 AI practitioners, is mapping how districts actually run AI: identity platforms, which AI tools are on or off, governance and app review, and the technical guardrails under staff-built software. Answers are published at https://lukeallpress.github.io/agentic-builders-commons/atlas/ so districts can learn from each other, **by region only**: the district's name, city, state, size and any written notes are never shown publicly, and only the multiple-choice answers appear. The organizer keeps the named copy privately to handle corrections. Everything below is district-level; nothing personal is collected.

## Instructions for the AI assistant

1. **Confirm the district.** Ask the person which district or organization they represent and their role, unless it is obvious from their account. Answer for that one organization only.
2. **Look for evidence first.** Search the sources you can reach: email, shared drives (Google Drive, OneDrive, SharePoint), chat (Slack, Teams), project tools (Asana, Jira, Trello), board policy pages, the district website, tech-plan documents, admin consoles you can read. Good search terms: AI policy, acceptable use, data privacy agreement, DPA, NDPA, app vetting, app request, Gemini, Copilot, ChatGPT, MagicSchool, SSO, ClassLink, Clever, GitHub, sandbox, MCP, intellectual property, vibe coding.
3. **Ask, don't guess.** Collect the questions you could not answer from evidence and ask the person in ONE short batch (at most 8 questions, multiple choice where possible). Anything still uncertain is `unknown`. Never invent an answer.
4. **Use the exact option text** for choice, list-of-options, and tool-status questions. Keep free-text answers to 1-3 plain sentences.
5. **Keep it public-safe.** Do NOT include: names of students or staff, email addresses or phone numbers, passwords, API keys or tokens, server names, IP addresses or internal URLs, details of security weaknesses or incidents, contract prices, or anything marked confidential. Describe approaches, not secrets ("keys live in a central vault" is fine).
6. **Show the draft.** Present the completed YAML block below to the person and ask them to check it. Make any corrections they give you.
7. **Submit, with permission.** Once the person approves, and only then, draft an email to **lallpress@aguafria.org** with the subject `District Atlas: <district name>, <state>` and the YAML block as the body. Let the person press send. If you cannot draft email, tell them to copy the YAML and email it to lallpress@aguafria.org, or use the "Email my answers" button at https://lukeallpress.github.io/agentic-builders-commons/atlas/.

Tool status values: `staff` (on for staff only), `staff+students` (on for both), `pilot` (limited group), `off` (available but not enabled), `blocked` (actively blocked), `unknown`.

## Answer template

Fill in every field. Leave `unknown` (or an empty list `[]`) where you truly cannot tell.

```yaml
atlas_version: 1
submitted: 2026-10-31  # today's date

context:  # District context
  # Official district or organization name. (required)
  district_name: unknown  # 1-3 sentences
  # Two-letter state code. (required)
  state: unknown  # 1-3 sentences
  # City where the district office is.
  city: unknown  # 1-3 sentences
  # Organization type. (required)
  org_type: unknown  # one of: public district | charter network | private/independent | regional service agency | state virtual school | other
  # Student enrollment. (required)
  enrollment_band: unknown  # one of: under 1k | 1k-5k | 5k-15k | 15k-50k | 50k+
  # Grades served.
  grade_span: unknown  # one of: K-12 | K-8 | 9-12 | PK-12 | other
  # Locale.
  locale: unknown  # one of: urban | suburban | town | rural | virtual
  # Main student device program.
  devices: unknown  # one of: 1:1 Chromebook | 1:1 Windows | 1:1 iPad | 1:1 Mac | mixed 1:1 | shared carts | BYOD
  # Role of the person submitting (no names needed).
  respondent_role: unknown  # 1-3 sentences

identity:  # Identity & platforms
  # Primary productivity suite. (required)
  productivity_suite: unknown  # one of: Google Workspace | Microsoft 365 | both, Google-led | both, Microsoft-led
  # Primary identity provider (where accounts live and SSO starts). (required)
  primary_idp: unknown  # one of: Google | Microsoft Entra ID | on-prem Active Directory | ClassLink | Clever | Okta | other
  # Rostering / SSO launchpad.
  sso_portal: unknown  # one of: ClassLink | Clever | both | none
  # Is MFA required for all staff accounts?
  mfa_staff: unknown  # yes | partial | no | unknown
  # Primary LMS.
  lms: unknown  # one of: Canvas | Schoology | Google Classroom | Microsoft Teams | Brightspace | Buzz | other
  # Student information system.
  sis: unknown  # one of: PowerSchool | Synergy | Infinite Campus | Skyward | Aeries | Focus | other

tools:  # AI tools: on or off
  # Status of each AI tool in your district. staff = on for staff only; staff+students = on for both; pilot = limited group; off = available but not enabled; blocked = actively blocked; unknown.
  ai_tools:
    Gemini: unknown  # staff | staff+students | pilot | off | blocked | unknown
    NotebookLM: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Microsoft Copilot Chat: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Microsoft 365 Copilot (licensed): unknown  # staff | staff+students | pilot | off | blocked | unknown
    ChatGPT (Edu/Enterprise): unknown  # staff | staff+students | pilot | off | blocked | unknown
    ChatGPT (consumer): unknown  # staff | staff+students | pilot | off | blocked | unknown
    Claude: unknown  # staff | staff+students | pilot | off | blocked | unknown
    MagicSchool: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Brisk: unknown  # staff | staff+students | pilot | off | blocked | unknown
    SchoolAI: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Khanmigo: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Perplexity: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Flint: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Playlab: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Google AI Studio: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Gemini Gems / custom GPTs: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Copilot Studio agents: unknown  # staff | staff+students | pilot | off | blocked | unknown
    Self-hosted / local models: unknown  # staff | staff+students | pilot | off | blocked | unknown
  # Default posture toward new AI tools.
  default_posture: unknown  # one of: allowlist: blocked until approved | case-by-case | open unless flagged
  # Any age or grade gate for student AI access? Describe.
  student_ai_age_gate: unknown  # 1-3 sentences
  # Other notable AI tools in use (district-approved).
  ai_tools_other: []  # list of short phrases

governance:  # Governance & review
  # AI policy status. (required)
  ai_policy: unknown  # one of: board-adopted policy | administrative guidance / AUP | in progress | none yet
  # Who reviews and approves new AI implementations? (committee, roles, cadence)
  ai_review_structure: unknown  # 1-3 sentences
  # What does app/vendor vetting require?
  app_vetting: []  # list, any of: signed DPA | SDPC / NDPA | state privacy agreement | security questionnaire | accessibility (VPAT/WCAG) check | instructional review | curriculum alignment | AI-specific rubric | cost/procurement review | none formal
  # Key items on your vetting checklist (short phrases).
  vetting_checklist: []  # list of short phrases
  # Policy on who owns tools/content staff build with AI?
  teacher_ip_policy: unknown  # one of: yes, written policy | in progress | informal understanding | none | unknown
  # Describe the ownership rule, if any.
  teacher_ip_notes: unknown  # 1-3 sentences
  # What data may go into AI tools? (e.g. no PII in consumer tools, PII only in approved enterprise tools)
  data_tiers: unknown  # 1-3 sentences
  # Student AI-use / academic integrity guidance.
  academic_integrity: unknown  # one of: district-wide framework | school or teacher discretion | in progress | none
  # Have you communicated your AI approach to families?
  parent_comms: unknown  # yes | partial | no | unknown

building:  # Democratized building
  # Who builds AI tools or apps in your district? (required)
  who_builds: []  # list, any of: IT / data team | innovation or ed-tech team | instructional coaches | teachers | school admins | students | outside contractors | nobody yet
  # How do you support staff 'vibe coding' (building software with AI)?
  vibe_coding_support: unknown  # one of: formal program with permits/training | supported informally | allowed but unsupported | discouraged | not addressed
  # Builder tools in use.
  builder_tools: []  # list, any of: Claude Code | OpenAI Codex | GitHub Copilot | Cursor | VS Code | Replit | Lovable | Bolt | v0 | Gemini Canvas | Google AI Studio | Apps Script | Power Apps / Power Automate | Copilot Studio | n8n | Zapier / Make | Antigravity
  # Training, cohort, or permit pathway for new builders?
  builder_pathway: unknown  # 1-3 sentences
  # What review must a staff-built app pass before real users touch it?
  app_launch_review: unknown  # 1-3 sentences
  # Where do staff-built apps run?
  hosting: []  # list, any of: GitHub Pages | Apps Script web apps | Firebase / Google Cloud | Vercel / Netlify | Azure | AWS | on-prem (IIS/Linux) | Replit / vendor hosting | nowhere official yet
  # Model Context Protocol (MCP) servers?
  mcp_servers: unknown  # one of: built our own, in production | built our own, piloting | using vendor MCPs only | exploring | not yet
  # What do your MCP servers expose? (no hostnames or secrets)
  mcp_details: unknown  # 1-3 sentences
  # AI agents or automations in real use (one line each).
  agents_in_production: []  # list of short phrases
  # Model APIs the district provisions.
  model_api_access: []  # list, any of: Gemini API / Vertex AI | Azure OpenAI | OpenAI API | Anthropic API | AWS Bedrock | self-hosted models | none

sandbox:  # Builder sandbox & protocols
  # Is there a separate development sandbox for builders?
  sandbox_env: unknown  # one of: yes, dedicated cloud project/tenant | yes, separate on-prem dev host/environment | yes, per-app dev copies | yes, separate OU/test accounts | builders use personal/dev accounts | no sandbox
  # Where can builders run code before production?
  dev_environments: []  # list, any of: local machine, empty or synthetic database | shared dev server or host | per-app dev copy | cloud dev project/tenant | low-code page sandbox with a review gate | none
  # Is version control required for staff-built code?
  version_control: unknown  # one of: required, district GitHub/GitLab org | encouraged | not required | not addressed
  # Is code reviewed before deployment?
  code_review: unknown  # one of: required review by IT | peer review | automated checks only | no review
  # Is AI-written code reviewed by a second, independent AI agent before a person approves it?
  ai_code_review: unknown  # one of: required for every live change | required for security/platform changes | optional | no
  # What stands between a merged change and production?
  deploy_gate: unknown  # one of: named human approves the deploy run | merge to main deploys automatically | IT deploys by hand | builder deploys directly | no rule yet
  # How do new staff-built tools reach users?
  staged_rollout: unknown  # one of: staged pilot, then one site, then all | all at once | builder decides | no rule yet
  # How are API keys and credentials handled? (approach only, never actual secrets)
  secrets_handling: unknown  # 1-3 sentences
  # How do staff-built apps authenticate to databases and services?
  app_identity: unknown  # one of: passwordless per-app identity (managed/integrated) | per-app service account with stored password | shared service account | builder's personal account | API keys in config
  # Which automated guards keep secrets and student data out of repositories?
  secret_scanning: []  # list, any of: pre-commit hook | CI secret scan | full-history scan before a repo goes public | forbidden-string scan (hostnames, names) | none
  # Do builders' coding agents load a shared, district-maintained set of instructions or skills before building?
  agent_context_standard: unknown  # one of: shared repo every builder's agent pulls before each build | per-project instruction files only | informal / per builder | none
  # Accessibility bar for staff-built apps.
  accessibility_standard: unknown  # one of: WCAG 2.1 AA with automated + manual checks | automated scan only | case by case | not addressed
  # Databases builders may use.
  database: []  # list, any of: Google Sheets | Firestore / Firebase | Supabase / Postgres | SQL Server | MySQL | BigQuery | SharePoint / Dataverse | SQLite / local files
  # What data is allowed in the sandbox?
  sandbox_data: unknown  # one of: synthetic only | de-identified | real data, least privilege | no rule yet
  # How must staff-built apps authenticate users?
  app_auth: unknown  # one of: district SSO required | SSO encouraged | app-specific logins allowed | no rule yet
  # Are prompts, tool calls, or app actions logged for audit?
  logging_audit: unknown  # yes | partial | no | unknown
  # How does a tool move from sandbox to production, and who approves?
  promotion_path: unknown  # 1-3 sentences
  # What happens to a tool when its builder leaves?
  ownership_handoff: unknown  # 1-3 sentences
  # Coding requirements or standards you enforce (short phrases).
  coding_standards: []  # list of short phrases
  # How do you cap or track AI/API spend?
  cost_controls: unknown  # 1-3 sentences

agents:  # Agents & MCP governance
  # When an AI model queries district data, where is the user's permission enforced?
  mcp_scope_enforcement: unknown  # one of: in the tools, scope resolved server-side per user | service account; the model is told what to respect | admin-only tools, no per-user scope | no AI access to district data yet
  # May a model write its own queries (SQL, API calls) against district data?
  model_generated_queries: unknown  # one of: never; predefined tools only | read-only, on a sandbox copy | read-only, on production data | no rule yet
  # What may agents change in district systems?
  agent_write_authority: unknown  # one of: read-only | propose; a person accepts and is the author of record | write within set limits | not addressed
  # Who approves a person's access to an MCP tool or agent, and is the grant logged?
  mcp_access_grants: unknown  # 1-3 sentences
  # How do you decide a model (especially a smaller or self-hosted one) is reliable enough for a production task?
  model_task_validation: unknown  # 1-3 sentences

data:  # Data foundation
  # Central data store for analytics.
  data_warehouse: unknown  # one of: BigQuery | Snowflake | SQL Server / on-prem warehouse | Azure / Fabric | vendor dashboard only | spreadsheets | none
  # How do you pull SIS data?
  sis_integration: unknown  # one of: direct API | OneRoster | scheduled exports | vendor connector | manual
  # Do apps, reports and agents read the live SIS or a district-owned copy?
  reporting_copy: unknown  # one of: district-owned reporting copy/warehouse only | live SIS read replica | live SIS directly | vendor API only
  # How are staff-built app databases structured?
  db_per_app: unknown  # one of: own database and own identity per app | shared database, separate schema per app | shared database and account | no rule yet
  # How do apps and agents read district databases?
  data_access_pattern: unknown  # one of: stored procedures or views only, EXECUTE grants | read-only role on tables | direct table access | through an API layer only | no rule yet
  # How do schema changes ship?
  schema_changes: unknown  # one of: numbered migration scripts in the repo, run by a DBA | ORM migrations applied on deploy | manual changes by a DBA | builders change production directly | no rule yet
  # Which shared services must every staff-built app use instead of building its own?
  central_ops_plumbing: []  # list, any of: shared sign-in | central event/audit log | district job scheduler | central reminder/notification service | shared hosting template | none
  # Do you maintain a data dictionary or ontology agents can use?
  semantic_layer: unknown  # yes | partial | no | unknown

learning:  # People & learning
  # How staff learn AI.
  pd_model: []  # list, any of: cohorts / communities of practice | PD days | coaching | self-paced courses | vendor training | student-led | none formal
  # One AI win you would show another district.
  biggest_win: unknown  # 1-3 sentences
  # One question you wish another district would answer for you.
  open_question: unknown  # 1-3 sentences
  # Artifacts you would share (policies, checklists, code).
  can_share: []  # list of short phrases

notes: ""  # anything else other districts should know (optional)
```

Thank you. Your district's answers will appear on the atlas after a quick review, and you can ask for any correction or removal at any time by emailing lallpress@aguafria.org.
