
Jenkins Installation Steps:
```
sudo yum install -y fontconfig java-17-openjdk
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum install -y jenkins
sudo systemctl daemon-reload
```

Docker Installation Steps:
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin --allowerasing
```

Maven Installation Step:
```
sudo yum install maven -y
```
---


---
1. Checked Jenkins service status
```shell
sudo systemctl status jenkins -l
```


Found:

Active: failed (Result: exit-code)

2. Verified Java installation
```
java -version
```

Output:
```
OpenJDK 17
```

At first this looked fine because older Jenkins versions support Java 17.

3. Checked if port 8080 was already in use

```
sudo ss -tulpn | grep 8080
```

Output:
```
tcp LISTEN ... 0.0.0.0:8080 ... ttyd
```

Discovered that ttyd (KodeKloud web terminal) was already using port 8080.

4. Updated Jenkins to use port 8090

Edited:
```
sudo vi /lib/systemd/system/jenkins.service

```
Changed:
```
Environment="JENKINS_PORT=8080"
```

to
```
Environment="JENKINS_PORT=8090"

```
Verified:
```
sudo grep JENKINS_PORT /lib/systemd/system/jenkins.service
```

Output:
```
Environment="JENKINS_PORT=8090"
```
5. Reloaded systemd configuration
```
sudo systemctl daemon-reload
```
7. Tried starting Jenkins again

```
sudo systemctl restart jenkins
```

Still failed.

7. Checked detailed logs
```
sudo journalctl -xeu jenkins.service --no-pager
```
Found the real error:
```
Running with Java 17 ...
which is older than the minimum required version (Java 21)
```
Supported Java versions are: [21, 25]
8. Installed Java 21
```
sudo dnf install -y java-21-openjdk java-21-openjdk-devel
```
9. Switched default Java (if required)
```
sudo alternatives --config java
```
```
Selected Java 21.
```
Verification:
```
java -version
```

Expected:
```
openjdk version "21.x.x"
```
10. Restarted Jenkins
```
sudo systemctl restart jenkins
```
11. Verified Jenkins is running
```
sudo systemctl status jenkins
```

Expected:

Active: active (running)

12. Verified Jenkins is listening on port 8090
```
sudo ss -tulpn | grep 8090
```

Expected:
```
LISTEN ... :8090 ...
```
Root Cause Summary

There were two issues:
```
Port conflict: ttyd was already using port 8080.
Java version mismatch: Installed Jenkins required Java 21, but the system had Java 17.
```
After changing Jenkins to port 8090 and upgrading to Java 21, Jenkins started successfully. ✅

This is a great real-world troubleshooting example for DevOps interviews because you systematically checked:

Service status
Network ports
Configuration files
System logs
Runtime dependencies (Java version)

<img width="1198" height="383" alt="image" src="https://github.com/user-attachments/assets/407e058f-94e9-47bf-967c-073aaa243443" />

---
<img width="601" height="48" alt="image" src="https://github.com/user-attachments/assets/bf41d650-442b-4a4c-8dc9-7487043c5452" />

---
