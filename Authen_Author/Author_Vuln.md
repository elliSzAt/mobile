# Authorization Vuln

Authorization vuln allows attackers to access data that they normally cannot access (files, functionally,...). A successful authorization attack allow attackers to view sensitive data, morever they can control the application.

# How does it arise?

Authorization vulnerabilities typically arise due to weaknesses or errors in how an application manages and enforces permissions and access controls.

# Types of Authorization

Authorization vuln include many vuln like Directory traversal, IDOR, Misconfigured access policies...

# Detect and Exploit

 - Finding endpoints with no guards.
 - Finding IDOR vulnerabilities where additional checks need to be performed apart from using the standard guards.