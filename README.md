# OWASP-10-A07-identification-and-authentication-failures

A thorough walkthrough around usage of OWASP ZAP tool applying the Docker approach 

This repository accompanies a lab walkthrough demonstrating the initial steps of web application security testing using the **OWASP Zed Attack Proxy (ZAP)** tool, specifically in the context of **OWASP Top 10 A07: Identification and Authentication Failures**.

## 🚀 Getting Started with OWASP ZAP (Docker)

To ensure an isolated and secure testing environment, we use ZAP via Docker.

### Installation & Launch

1.  **Pull the Stable ZAP Image:**

    ```bash
    docker pull ghcr.io/zaproxy/zaproxy:stable
    ```

2.  **Run ZAP with Webswing UI:**
    This command maps the internal container ports to the host and launches the ZAP desktop UI accessible via a web browser.

    ```bash
    docker run -u zap -p 8080:8080 -p 8090:8090 -i ghcr.io/zaproxy/zaproxy:stable zap-webswing.sh
    ```

3.  **Access ZAP:**
    Navigate to the Webswing UI in your browser:
    [http://localhost:8080/zap](https://www.google.com/search?q=http://localhost:8080/zap)

-----

## 🛡️ Pentesting Execution: Automated Scan

The quickest way to start testing is using the **"Quick Start"** $\rightarrow$ **"Automated Scan"** feature in ZAP. This performs **Passive Scanning** (safe enumeration) followed by **Active Scanning** (real attacks).

### Key Scanning Phases

| Phase | Description | Safety |
| :--- | :--- | :--- |
| **Passive Scan** | Crawls the application, identifies URLs and pages, and analyzes requests/responses without modifying them. | **Safe** |
| **Active Scan** | Attacks discovered targets with known attack vectors. | **Potentially Disruptive** |

-----

## 🛑 Critical Problems Encountered (Troubleshooting)

During the lab, two main issues arose when targeting a vulnerable application like **WebGoat** running simultaneously in a Docker environment.

### 1\. Port Collision 💥

If both ZAP and the target application (e.g., WebGoat) are running on the same host and internal container port (e.g., `8080`), you will face a collision.

**Solution Options:**

  * Change the target application's exposed port.
  * Use custom hostnames in the `/etc/hosts` file (though this doesn't fully solve the next issue).

### 2\. The Loopback Trap (Targeting ZAP Itself) 🤯

This is the most critical issue when using Docker's default **bridge network mode**:

  * When ZAP runs in a container and attempts to attack a URL like `http://www.webgoat.local:8080/WebGoat/` (where `www.webgoat.local` resolves to `127.0.0.1` on the **host**), the address resolution happens, but the *connection* then occurs **inside the ZAP container's context**.
  * Inside the ZAP container, `127.0.0.1` (`localhost`) refers to the **ZAP container itself**.
  * **Result:** ZAP ends up attacking its own running instance\!

#### ✅ The Correct Fix: Inter-Container IP Addressing

Since Docker containers in a bridge network have their own private IP addresses and can communicate, the solution is to target the **WebGoat container's internal IP address** directly from ZAP.

1.  **Inspect the Docker Network** to find the IP of the target container (WebGoat).

    ```bash
    # Example command to find container IPs
    docker network inspect bridge 
    ```

2.  **Target the IP:** Use the target container's IP address instead of `localhost` or a custom hostname.

    ```url
    # Replace the example IP with your WebGoat container's IP
    http://172.17.0.4:8080/WebGoat
    ```

This approach successfully directs the scan to the intended target.

-----

## 📊 Interpreting Results

After the scan, the **Alerts** tab at the bottom of the ZAP UI provides the vulnerability reports, highlighting the specific part of the request or response that triggered the alert. The **Sites** tree on the left lists all queried URLs.

## 💡 Recommendations for Further Exploration

The automated scan has limitations, particularly with authenticated areas (like WebGoat's login pages). Consider the following for advanced testing:

  * **Manual Scanning:** Manually explore the application while ZAP is proxying traffic to discover more paths.
  * **API Scan:** Utilize ZAP's specialized API scanning features.
  * **Automation Framework:** Explore the ZAP Automation Framework for easier, fully customized, and continuous integration of security testing.

## 🔗 Useful Links

  * **OWASP ZAP Docker Docs:** [https://www.zaproxy.org/docs/docker/](https://www.zaproxy.org/docs/docker/)
  * **ZAP Getting Started:** [https://www.zaproxy.org/getting-started/](https://www.zaproxy.org/getting-started/)
  * **ZAP Automation Framework:** [https://www.zaproxy.org/docs/desktop/addons/automation-framework/](https://www.zaproxy.org/docs/desktop/addons/automation-framework/)
