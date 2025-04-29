NetApp’s Anti-Ransomware Protection, known as Autonomous Ransomware Protection (ARP), is an AI-powered security feature integrated into NetApp’s ONTAP storage operating system. It is designed to detect and respond to ransomware attacks in real-time, safeguarding data stored in NAS environments (NFS and SMB). ￼ ￼

Key Features of NetApp’s Autonomous Ransomware Protection
	•	Real-Time Detection: ARP monitors file system activities to identify abnormal behaviors indicative of ransomware attacks. Upon detecting such activities, it automatically creates snapshots of the affected volumes, enabling rapid data recovery.  ￼
	•	AI-Powered Analytics: Starting with ONTAP version 9.16.1, ARP incorporates machine learning models trained on extensive datasets of pre- and post-attack file behaviors. This enhances its ability to detect evolving ransomware threats with high accuracy.  ￼
	•	Integration with AWS: As of April 2025, ARP is available at no additional cost for all Amazon FSx for NetApp ONTAP file systems across all AWS regions. This integration allows for seamless protection of cloud-based data.  ￼
	•	Deployment Modes: For ONTAP versions 9.10.1 to 9.15.1, ARP operates initially in a "learning mode" to establish a baseline of normal activity, reducing false positives. From version 9.16.1 onward, ARP can be activated immediately without a learning period.  ￼
	•	Comprehensive Protection: ARP is part of NetApp’s broader ransomware protection portfolio, which includes features like SnapLock for immutable backups, multi-admin verification to prevent unauthorized changes, and Cloud Insights for monitoring user behavior.  ￼

NetApp’s ARP offers a robust, integrated solution for organizations seeking to enhance their data resilience against ransomware threats. 