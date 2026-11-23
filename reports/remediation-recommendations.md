
# Remediation Recommendations
Capstone Project: Enterprise Network Vulnerability Assessment & Penetration Testing
Target: Metasploitable2 (192.168.56.101)

## 1. vsftpd 2.3.4 Backdoor (Port 21)
Remove the compromised vsftpd package immediately. Reinstall from an official,
verified source, and always validate package signatures before installation.
Upgrade to the latest stable vsftpd release.

## 2. UnrealIRCd 3.2.8.1 Backdoor (Port 6667)
Remove the trojaned UnrealIRCd installation entirely and reinstall from the
official UnrealIRCd repository. Verify file checksums against official
releases before deployment.

## 3. Samba usermap_script Command Execution (Ports 139/445)
Upgrade Samba to a patched version (3.0.25rc4 or later). Disable the
"usermap script" option in smb.conf unless explicitly required, and restrict
SMB access to trusted internal networks only.

## 4. distccd Remote Code Execution (Port 3632)
Disable the distcc daemon entirely unless actively needed for distributed
compilation. If required, bind it only to trusted internal interfaces and
never expose it to untrusted networks, since it has no built-in
authentication.

## 5. Ingreslock Backdoor (Port 1524)
This is a known backdoor with no legitimate purpose -- the host should be
treated as compromised. Full system reimaging is recommended rather than
attempting to "clean" an infected system.

## 6. TWiki XSS / Command Execution (Port 80)
Upgrade TWiki to version 4.2.4 or later. Sanitize all user-supplied input in
URLPARAM and SEARCH variables, and disable eval()-based template processing
where possible.

## 7. Apache Tomcat Ghostcat (Port 8009)
Upgrade Tomcat to 7.0.100, 8.5.51, 9.0.31, or later. If the AJP connector
isn't needed, disable it entirely in server.xml; otherwise restrict it to
trusted internal hosts only.

## 8. Distributed Ruby (dRuby) RCE (Port 8787)
Set appropriate $SAFE levels (>=2) in the dRuby service configuration,
implement drb/acl.rb access control lists to restrict connections to trusted
hosts, and avoid running dRuby services with elevated privileges.

## 9. rexec Service Exposed (Port 512)
Disable the rexec service entirely. Replace remote execution needs with SSH,
which encrypts credentials and traffic.

## 10. rlogin Passwordless Login (Port 513)
Disable the rlogin service immediately and remove any .rhosts trust
configurations. Replace with SSH key-based authentication.

## 11. Operating System End-of-Life (Ubuntu 8.04)
Upgrade the underlying OS to a currently supported Ubuntu LTS release. An EOL
OS receives no security patches, meaning every vulnerability above is
compounded by the inability to patch at the OS level -- this should be the
top priority fix, since it undermines every other remediation.
