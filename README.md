# Devsecops

# Common Threats & The Importance of Container Security
<img width="2978" height="1872" alt="image" src="https://github.com/user-attachments/assets/46f1c135-bb1f-4304-970d-61edc162433f" />
<img width="1490" height="888" alt="image" src="https://github.com/user-attachments/assets/ded5dd0e-cbb9-4746-ab1c-f1d1b3d9183b" />
<img width="987" height="580" alt="image" src="https://github.com/user-attachments/assets/f9d52971-afb1-489c-bc6a-bba5d6c865c1" />


# Single github action pipeline can run SAST + SCA + SBOM + DAST 

1 SAST (earliest, on PR)

Analyze source code before build. Fast feedback, catches code-level bugs/taint issues. CodeQL (code scanning in PRs)

2 SCA (twice: PR & post-build) SCA with Snyk (fails on High/Critical; uploads SARIF)

PR stage: scan dependency manifests/lockfiles (app-level SCA).

Post-build: scan the built artifact/image (OS packages + app deps). This catches what actually ships.

3 SBOM (at build, after artifact is final) SBOM generation with Syft (CycloneDX) and Cosign attestation to your image

Generate from the final image/binary so it exactly matches what you’ll deploy; sign/attest and store.

Syft (Anchore) – CycloneDX/SPDX for images, dirs, repos.
syft ghcr.io/acme/app:1.2.3 -o cyclonedx-json > sbom.cdx.json

Trivy (Aqua) – SBOM plus vuln/secret/IaC scanners.
trivy image --format cyclonedx -o sbom.cdx.json ghcr.io/acme/app:1.2.3

Sign images + SBOM attestation (Cosign).

4  Admission policy (Kyverno/Gatekeeper/Sigstore) to allow only signed images from your CI.

5  DAST (after deploy to staging/prod mirror) with OWASP (Open Worldwide Application Security Project) ZAP Baseline against your staging URL (fails on Medium/High)

Needs a running app/API with auth/test data; run baseline on every change, deeper active scans nightly.



# SAST Static Application Security Testing:  SonarQube

# SBOM(Software Bill of Material): 
Syft produces a CycloneDX SBOM for the built image; it’s stored as a workflow artifact and attached to the image as a Cosign attestation (tied to the digest in GHCR).
Uses open formats (CycloneDX or SPDX)
Stored centrally (e.g., registry/Artifact Hub, Dependency-Track) and kept in sync with deployments.
Signed/attested (e.g., Cosign) and queryable.

trivy image --format cyclonedx -o sbom.cdx.json your.registry/app:tag

# SCA(Software Composition Analysis): 
Snyk test scans your dependencies; it fails the job on High/Critical issues and publishes SARIF to Security → Code scanning.
Trivy: finds CVEs in OS packages & app dependencies in images, filesystems, and repos.

# DAST(Dynamtic Application Security Testing): 
it focuses on inputs and outputs of the running application.
The goal of DAST is to catch any security issues that weren’t caught by SAST, which means that they’re supposed to be complementary to each other.
ZAP Baseline crawls your staging URL and fails on Medium/High alerts; reports are saved as artifacts.


# Shift-left security 
with DevSecOps, we want to include security from the very beginning so that problems are found as early in the process as possible: closest to the decision makers and to the context.

<img width="1090" height="596" alt="image" src="https://github.com/user-attachments/assets/8180e8fa-06e2-456e-b8a3-c4992e22528e" />
<img width="1158" height="610" alt="image" src="https://github.com/user-attachments/assets/82c911e6-fe35-463f-88eb-e2c56da9f527" />

<img width="1722" alt="image" src="https://github.com/user-attachments/assets/2c2e1ff1-6068-4415-96ce-0a4d835bab77">

<img width="1310" alt="image" src="https://github.com/user-attachments/assets/fc49c82e-24a9-4141-8bd3-868dc444d0f6">

The scan result as following: Critical severity vulnerability found in zlib/zlib1g

<img width="1665" alt="image" src="https://github.com/user-attachments/assets/95085816-ccdc-4039-88fb-3d7b16e4e01c">

<img width="1080" alt="image" src="https://github.com/user-attachments/assets/789ae184-fe2f-4e08-b053-5245ee26c587">


# Container image best practices 
<img width="1176" height="832" alt="image" src="https://github.com/user-attachments/assets/da81260f-b481-4fba-84c1-8dbad1b30940" />
<img width="1501" height="883" alt="image" src="https://github.com/user-attachments/assets/e6e2f9ff-09a5-42fe-bc30-9ed14901c521" />

<img width="991" height="759" alt="image" src="https://github.com/user-attachments/assets/554984ef-dd91-4e83-9a91-be9b889f10a4" />

# Signs the image using Cosign
Signing a container image with Cosign makes your supply chain tamper-evident and trustable. It proves who built an image and that what you deploy is exactly what was built.
1 Integrity: Binds a cryptographic signature to the image digest → any post-build change breaks verification.
2 Policy enforcement: Kubernetes admission (Kyverno/Gatekeeper/Sigstore Policy Controller) can block unsigned/untrusted images.
3 Auditability: Signatures/attestations can be logged in a transparency log (Rekor) → non-repudiation and forensics.
4 Rich metadata (attestations): Attach SBOMs, build provenance (SLSA), and vulnerability scan results; verify them at deploy time.

# Adminssion Control (2025/09/27)

# How to control no privileged container in namespace through admission controller
1) Pod Security Admission (built-in; simplest)

Kubernetes ≥1.25 replaces PodSecurityPolicy with Pod Security Admission (PSA).
Label the namespace to enforce the restricted profile (which forbids privileged: true, along with other risky settings)

kubectl label ns prod-apps \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted

2) Kyverno policy (flexible; easy to read) 
Install Kyverno, then apply a deny policy for privileged containers
<img width="713" height="570" alt="image" src="https://github.com/user-attachments/assets/6b121e4f-89f6-4e60-9cd2-dd9a2b2e941d" />




Kubernetes (K8s) Admission Control is a critical security and governance mechanism that acts as a "gatekeeper" for your cluster. It intercepts requests to the Kubernetes API server after the request has been authenticated and authorized, but before the object is persisted to etcd (the cluster's database).
<img width="1718" height="622" alt="image" src="https://github.com/user-attachments/assets/8522290d-7abe-4eb5-b513-f9dac0e2aa0f" />

Key Functions and Benefits
Admission controllers are essential for various advanced Kubernetes features and cluster management:

1 Security: They enforce security contexts, prevent the use of privileged containers, and ensure that only compliant images are deployed.

2 Policy Enforcement/Governance: They ensure that all resources adhere to organizational rules, such as mandatory labels, naming conventions, and resource limits.

3 Resource Management: They enforce resource quotas and limits (CPU, memory) to prevent resource hogging and maintain cluster stability.

4 Defaulting: Mutating controllers can automatically inject necessary values if they are missing from a resource definition, such as setting a default storage class for a PersistentVolumeClaim.



# POD Security Admission 
Pod Security Admission (PSA) is used to control what Pods are allowed to run, from a security point of view.
PSA’s purpose is to prevent insecure Pod specs from ever getting admitted—it’s guardrails for Pod security
<img width="1648" height="461" alt="image" src="https://github.com/user-attachments/assets/2a128503-6864-41bf-afca-3d58f78df4c7" />
Profiles you choose per namespace:

1 privileged – allow everything (for system namespaces)

2 baseline – blocks known-bad patterns

3 restricted – strong hardening (most prod namespaces)

Modes:

1 enforce (deny non-compliant pods)

2 warn (allow but show warnings)

3 audit (allow but log)

Defaults per namespace

Prod app namespaces: enforce=restricted, warn/audit=restricted, *-version=latest.

Dev / QA: enforce=baseline, warn/audit=restricted.

System / operators (e.g., kube-system, CNI/CSI): keep privileged (no restrictive labels).




# Container Runtime Security (AWS GuardDuty EKS Runtime Monitor and Falco )
<img width="1102" height="510" alt="image" src="https://github.com/user-attachments/assets/dd43a645-42e8-40c4-ba1b-ae5f31d16776" />
Falco (by Sysdig) is the most popular and widely adopted container runtime security tool. It runs as a DaemonSet, uses eBPF/syscalls, and ships with lots of community rules to detect things like exec-ing a shell in a container, privilege-escalation, or sensitive file access.

<img width="897" height="510" alt="image" src="https://github.com/user-attachments/assets/a0ed507a-30e3-4994-8064-795f5b3d20bc" />
<img width="1307" height="690" alt="image" src="https://github.com/user-attachments/assets/855dc045-2ab4-480f-8238-7b2a7a6bf622" />

1  fastest value, minimal ops, centralized AWS findings (Security Hub), and predictable pricing then Choose GuardDuty EKS Runtime Monitoring (+ EKS Protection)
2  need custom, fine-grained policies (e.g., “alert if apt/yum runs in prod, except image X”), custom sinks, or multi-cloud/on-prem parity. Add Falco (often alongside GuardDuty)

# Practical combo (common in EKS) Many teams run both

1 GuardDuty for managed detections + account-level visibility and compliance reporting.

2 Falco for tailored, high-signal rules your team controls (with Sidekick → Slack/SIEM) and rapid iteration. 

EKS Runtime Monitoring provides runtime threat detection coverage for Amazon EKS nodes and containers. It uses the GuardDuty security agent (EKS add-on) that adds runtime visibility into individual EKS workloads, for example, file access, process execution, privilege escalation, and network connections identifying specific containers that may be potentially compromised.

Detects suspicious runtime behavior: unexpected shells, privilege-escalation attempts, crypto-mining, sensitive file access, anomalous network activity, etc.

Findings appear in GuardDuty (and flow to EventBridge/SIEM for alerting/auto-remediation).

Pairs well with EKS Audit Log Monitoring (control-plane detections without an agent).

Note: GuardDuty detects and alerts; it doesn’t block containers. For prevention/enforcement, keep using PSA restricted, admission policies (Kyverno/OPA/CEL), network policies, and runtime LSMs (SELinux/AppArmor) alongside GuardDuty.

<img width="1051" height="808" alt="image" src="https://github.com/user-attachments/assets/7dde5b4e-82e5-4064-894c-327a0488bb9d" />



# Run OWASP ZAP to our site 
OWASP ZAP is a penetration testing tool that helps developers and security professionals detect and find vulnerabilities in web applications. OWASP ZAP performs multiple security functions including: Passively scanning web requests. Using dictionary lists to search for files and folders on web servers.
<img width="1325" alt="image" src="https://github.com/user-attachments/assets/75cdae9b-05e8-4106-9abf-38d71d6b73f5">
<img width="1287" alt="image" src="https://github.com/user-attachments/assets/070a95b7-8717-4c5f-8cdf-b3eeed49c996">

# OPA Gatekeeper 
The cluster can enforce Kubernetes policies at admission time (when resources are created/updated). Gatekeeper is the OPA-powered admission controller that lets you write policy-as-code and block/allow deploys based on rules.

# Kyverno Kubernetes-native policy engine
<img width="812" height="273" alt="image" src="https://github.com/user-attachments/assets/c0ec72ad-0fd8-4fa3-af4e-4d00e7a4c716" />

Policies are just Kubernetes CRDs (YAML), not Rego:

ClusterPolicy (cluster-wide) and Policy (namespace-scoped).

Runs as admission webhooks (mutating + validating) and a background controller that scans existing resources and produces PolicyReports.

<img width="819" height="533" alt="image" src="https://github.com/user-attachments/assets/79b921d0-d5d4-43ad-a5bb-fa8aa6c68765" />
<img width="793" height="577" alt="image" src="https://github.com/user-attachments/assets/71f5fdf9-715c-4fd0-953e-3f03f0fb0db8" />


# Cloudformation event order 
<img width="1728" alt="image" src="https://github.com/user-attachments/assets/9d3b096f-b197-46ee-aafe-68a259cb4f74">


On your Mac check your IP address and add it to IP ruleset to allow traffic to Cloudfront 

<img width="1346" alt="image" src="https://github.com/user-attachments/assets/c785cf29-eb40-4220-be46-69c1faea1a49">
<img width="1459" alt="image" src="https://github.com/user-attachments/assets/250d8454-ed95-4c4f-96b9-32e15cabede9">


<img width="1084" alt="image" src="https://github.com/user-attachments/assets/c4c81a6b-b00f-4257-9da8-352303f9f13a">

<img width="1133" alt="image" src="https://github.com/user-attachments/assets/e530158e-e046-4513-bdae-38a77905ab6d">


<img width="1694" alt="image" src="https://github.com/user-attachments/assets/e0e71196-0148-4cb2-ab62-655a6fd1e877">

<img width="1347" alt="image" src="https://github.com/user-attachments/assets/58a3f736-f659-4ac4-a13e-5ab02916e7ea">

<img width="1684" alt="image" src="https://github.com/user-attachments/assets/8c9f6538-45ad-4311-8ead-939da86186c5">

<img width="1699" alt="image" src="https://github.com/user-attachments/assets/50efb837-9c83-4712-824d-1bcdcb0d6fe7">

<img width="1696" alt="image" src="https://github.com/user-attachments/assets/0f9ad865-6e12-4cc8-8016-545aa9a1d117">


Need add your Mac IP to WAF allow IP list or your request will be blocked 

<img width="1307" alt="image" src="https://github.com/user-attachments/assets/f90af684-50ed-481a-92bf-50e46f8d07fc">



<img width="1086" alt="image" src="https://github.com/user-attachments/assets/8bedbe2c-386b-41c0-91ad-44a058b4680e">

Use cloudfront to cache Web page 

<img width="1696" alt="image" src="https://github.com/user-attachments/assets/81c01a7f-7e1c-4fbe-bf57-7db37ed11b57">

Cloudfront has two origins, one is Nginx pod running on EKS , one is API GW which has lambda function and Dynamodb as backend

<img width="1378" alt="image" src="https://github.com/user-attachments/assets/9312bd7a-f075-4e4e-9088-a4fdf2058c72">

We use managed cache policy for both ALB and API GW

<img width="734" alt="image" src="https://github.com/user-attachments/assets/941872b0-48cc-461b-ac58-c0d7a4cd1af9">
<img width="805" alt="image" src="https://github.com/user-attachments/assets/70beb430-97fc-4755-8cd1-1691456c02ea">

API GW also has cache enabled so from X-RAY we can find cache query does not invoke lambda function.
<img width="1367" alt="image" src="https://github.com/user-attachments/assets/0728dc06-64dd-4760-9f80-3b7073bf09b9">



Performance Monitor using Container Insight
<img width="1676" alt="image" src="https://github.com/user-attachments/assets/1dcdc070-59c6-4d15-8182-2c412b9ac764">

Log monitor using Log Insight 
<img width="1679" alt="image" src="https://github.com/user-attachments/assets/c2c1a96b-e6cf-433d-a76f-496acf645655">

AWS WAF sit in front of Cloudfront to protect Cloudfront with managed rules and user defined rules.
<img width="1689" alt="image" src="https://github.com/user-attachments/assets/dd29b67e-c9f1-4fd7-b8bc-f09b64ec1917">




<img width="839" alt="image" src="https://github.com/user-attachments/assets/b27f6f9a-91ae-4f96-a890-3a13b0722745">


# github-actions

![image](https://github.com/xiongye77/github-actions/assets/36766101/eb89464c-d7de-4899-a1f9-331b524ff29c)

![image](https://github.com/user-attachments/assets/0fd3efb5-d74e-4726-bc1c-8592d403c036)
![image](https://github.com/user-attachments/assets/9a0d8412-969b-4630-b973-4b2a8a6bf08a)
![image](https://github.com/user-attachments/assets/d4fac4fd-26fb-44ae-8f04-19e724cd08cf)
![image](https://github.com/user-attachments/assets/472e59e4-2722-4417-a4b0-0e5a709c2447)
![image](https://github.com/user-attachments/assets/7701727e-a58f-4ec7-9d4c-1a45acb6f20d)
![image](https://github.com/user-attachments/assets/efba3400-a185-4012-953c-4188dd03d5f8)



![image](https://github.com/user-attachments/assets/ad567116-9c43-4faa-815a-79e11e0160cd)
![image](https://github.com/user-attachments/assets/0985b53d-52f6-407a-8225-a8b837dbcabc)



# Static Application Security Testing (SAST)  and  OWASP Top Ten
The OWASP Top Ten is a list of the top ten most critical web application security risks. It is compiled by the Open Web Application Security Project (OWASP), a nonprofit organization focused on improving software security.

The OWASP Top Ten is updated periodically to reflect the evolving landscape of web application security threats. The list is intended to raise awareness about common vulnerabilities and provide guidance on how to mitigate them effectively. It serves as a valuable resource for developers, security professionals, and organizations to prioritize their efforts in securing web applications.

The OWASP Top Ten typically includes vulnerabilities such as:

Injection: This refers to security flaws that occur when untrusted data is sent to an interpreter as part of a command or query, leading to execution of unintended commands or access to unauthorized data.

Broken Authentication: This involves security weaknesses in authentication and session management, which can allow attackers to compromise user accounts, steal session tokens, or impersonate users.

Sensitive Data Exposure: This refers to vulnerabilities that expose sensitive data, such as passwords, credit card numbers, or personal information, to unauthorized parties.

XML External Entities (XXE): This occurs when XML input containing a reference to an external entity is processed by a weakly configured XML parser, leading to disclosure of confidential data, denial of service, or server-side request forgery.

Broken Access Control: This involves security flaws that allow users to access unauthorized resources or perform unauthorized actions, bypassing access controls.

Security Misconfiguration: This refers to insecure configurations of web applications, web servers, frameworks, or platforms, which can lead to unauthorized access, information disclosure, or other security issues.

Cross-Site Scripting (XSS): This occurs when an attacker injects malicious scripts into web pages viewed by other users, leading to theft of session cookies, redirection to malicious websites, or other attacks.

Insecure Deserialization: This involves security weaknesses in deserialization processes, which can lead to remote code execution, denial of service, or other attacks.

Using Components with Known Vulnerabilities: This refers to security risks associated with the use of outdated or vulnerable components, libraries, or frameworks in web applications.

Insufficient Logging and Monitoring: This involves inadequate logging and monitoring of security events, which can hinder detection and response to security incidents.

By addressing the vulnerabilities outlined in the OWASP Top Ten, organizations can significantly improve the security posture of their web applications and better protect sensitive data and resources from cyber threats.



# Relationship between Static Application Security Testing (SAST) and OWASP Top Ten


Static Application Security Testing (SAST) is a method of analyzing source code, bytecode, or binary code to find security vulnerabilities that may exist within an application's codebase. It is typically performed during the development phase of the software development lifecycle.

![image](https://github.com/xiongye77/github-actions-devsecops/assets/36766101/1b8f7c2a-3324-4950-8bfd-af53a7f9c081)
![image](https://github.com/xiongye77/github-actions-devsecops/assets/36766101/26ed837b-b139-4b0c-a6aa-4be0ca2eae0d)
![image](https://github.com/xiongye77/github-actions-devsecops/assets/36766101/b266021f-31bf-484d-a80d-d2befd852cfe)


The relationship between SAST and the OWASP Top Ten is primarily about how SAST tools can help identify and mitigate the security risks outlined in the OWASP Top Ten. Here's how SAST and the OWASP Top Ten are related:

Identification of Vulnerabilities: SAST tools analyze source code to identify potential vulnerabilities such as injection flaws, broken authentication, sensitive data exposure, and others, which are part of the OWASP Top Ten. By scanning the codebase, SAST tools can detect security weaknesses that could lead to the exploitation of these OWASP Top Ten vulnerabilities.

Prioritization of Remediation Efforts: Since the OWASP Top Ten represents the most critical web application security risks, organizations often prioritize remediating vulnerabilities related to these risks. SAST tools can help prioritize remediation efforts by identifying which vulnerabilities in the codebase correspond to the OWASP Top Ten.

Compliance with Best Practices: Following the OWASP Top Ten recommendations is considered a best practice in web application security. By using SAST tools to identify and address vulnerabilities related to the OWASP Top Ten, organizations can demonstrate compliance with industry standards and best practices for secure software development.

Integration with Development Workflow: SAST tools can be integrated into the development workflow to automate security testing as part of the continuous integration/continuous deployment (CI/CD) pipeline. By incorporating SAST scans into the development process, organizations can detect and fix security issues early in the software development lifecycle, reducing the likelihood of OWASP Top Ten vulnerabilities making their way into production.

Overall, SAST tools play a crucial role in helping organizations address the security risks outlined in the OWASP Top Ten by identifying vulnerabilities in the codebase and facilitating their remediation during the development process. is a method of analyzing source code, bytecode, or binary code to find security vulnerabilities that may exist within an application's codebase. It is typically performed during the development phase of the software development lifecycle.

The relationship between SAST and the OWASP Top Ten is primarily about how SAST tools can help identify and mitigate the security risks outlined in the OWASP Top Ten. Here's how SAST and the OWASP Top Ten are related:

Identification of Vulnerabilities: SAST tools analyze source code to identify potential vulnerabilities such as injection flaws, broken authentication, sensitive data exposure, and others, which are part of the OWASP Top Ten. By scanning the codebase, SAST tools can detect security weaknesses that could lead to the exploitation of these OWASP Top Ten vulnerabilities.

Prioritization of Remediation Efforts: Since the OWASP Top Ten represents the most critical web application security risks, organizations often prioritize remediating vulnerabilities related to these risks. SAST tools can help prioritize remediation efforts by identifying which vulnerabilities in the codebase correspond to the OWASP Top Ten.

Compliance with Best Practices: Following the OWASP Top Ten recommendations is considered a best practice in web application security. By using SAST tools to identify and address vulnerabilities related to the OWASP Top Ten, organizations can demonstrate compliance with industry standards and best practices for secure software development.

Integration with Development Workflow: SAST tools can be integrated into the development workflow to automate security testing as part of the continuous integration/continuous deployment (CI/CD) pipeline. By incorporating SAST scans into the development process, organizations can detect and fix security issues early in the software development lifecycle, reducing the likelihood of OWASP Top Ten vulnerabilities making their way into production.

Overall, SAST tools play a crucial role in helping organizations address the security risks outlined in the OWASP Top Ten by identifying vulnerabilities in the codebase and facilitating their remediation during the development process.





https://cybr.com/courses/beginners-guide-to-aws-cloudtrail-for-security/lessons/about-the-course-7/
