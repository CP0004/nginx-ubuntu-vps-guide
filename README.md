# Installing and Configuring Nginx to Link an Application Port with a Domain on an Ubuntu VPS

## Introduction

If you want to run a web application (such as a Node.js or FastAPI application) on an Ubuntu VPS and link it to your own domain, using **Nginx** as a reverse proxy server is one of the best solutions. In this guide, we will go through the necessary steps to install and configure Nginx to link your application port to your domain, providing notes on potential issues and how to resolve them, as well as best practices to ensure smooth and secure operation.

---

## Step 1: Update Packages and Install Nginx

Before starting, it is advisable to update the list of available packages on the server:

```bash
sudo apt update
```

Then, install Nginx:

```bash
sudo apt install nginx
```

*Note:* You may need root privileges or use `sudo` to execute these commands.

---

## Step 2: Create a New Configuration File for the Site

Instead of modifying the default file for Nginx, it is better to create a separate configuration file for your domain. This makes managing multiple sites easier and avoids conflicts between configurations.

Create a new configuration file:

```bash
sudo nano /etc/nginx/sites-available/your_domain.com
```

*Replace `your_domain.com` with your actual domain name.*

---

## Step 3: Add Nginx Configuration for the Site

Add the following configuration to the file you opened:

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name your_domain.com www.your_domain.com;

    location / {
        proxy_pass http://127.0.0.1:3000; # IP address and port where your application is running
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Additional settings if your application uses WebSockets
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Explanation of the Configuration:

- `listen 80;` and `listen [::]:80;`: Makes Nginx listen for requests on port 80 over both IPv4 and IPv6.
- `server_name`: Specifies the domain names that this configuration will respond to.
- `location /`: Defines how Nginx should handle requests directed to the `/` path.
- `proxy_pass`: Forwards requests to the application running on `127.0.0.1:3000`. Make sure to adjust the port to match your application's port.
- WebSocket settings are important if your application uses WebSocket connections (like applications using Socket.io).

*Ensure you replace `your_domain.com` and `www.your_domain.com` with your domain name, and adjust the port `3000` to the port your application is using.*

---

## Step 4: Enable the New Configuration

To enable the new configuration, we need to create a symbolic link in the `sites-enabled` directory:

```bash
sudo ln -s /etc/nginx/sites-available/your_domain.com /etc/nginx/sites-enabled/
```

---

## Step 5: Disable the Default Configuration (Optional but Recommended)

To ensure there are no conflicts with the default configuration, you can disable it:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

*Note:* Make sure you do not need the default configuration before removing it.

---

## Step 6: Check Nginx Configuration for Errors

Before restarting Nginx, you should check the configuration for errors to ensure there are no issues:

```bash
sudo nginx -t
```

- If the configuration is correct, you will see a message similar to:
  ```
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
  nginx: configuration file /etc/nginx/nginx.conf test is successful
  ```
- If there are errors, Nginx will provide information about the issue and its location in the files for easy correction.

---

## Step 7: Restart Nginx to Apply Changes

```bash
sudo systemctl restart nginx
```

or

```bash
sudo service nginx restart
```

---

## Step 8: Verify the Site is Working

Open your web browser and navigate to `http://your_domain.com` to see if your application is displayed correctly.

---

## Potential Issues and How to Resolve Them

### Issue 1: 502 Bad Gateway Error

**Possible Cause:**

- Nginx cannot reach your application on the specified port.

**Solutions:**

- Ensure your application is running correctly on the server and on the specified port.
- Check that your application is listening on `127.0.0.1` or `0.0.0.0`.
- Check the firewall to ensure that connections on the port used by the application are not blocked.

### Issue 2: Missing or Invalid SSL/TLS Certificate

**Possible Cause:**

- You are trying to access via `https` without setting up an SSL/TLS certificate.

**Solutions:**

- Install an SSL certificate using Let's Encrypt and Certbot.
- Set up redirection from `http` to `https` in the Nginx configuration after installing the certificate.

### Issue 3: DNS Changes Not Propagated Yet

**Possible Cause:**

- You changed your domain's DNS settings, and the changes have not propagated yet.

**Solutions:**

- Wait some time (it may take up to 48 hours) for the DNS changes to propagate.
- Use tools like `dig` or `nslookup` to check the DNS status.

### Issue 4: Incorrect File Permissions or Ownership

**Solutions:**

- Ensure Nginx has the appropriate permissions to access the configuration files.
- Check the owner and group of the files (usually `root:root` for configuration files in `/etc/nginx/`).

---

## Best Practices

### 1. Use SSL/TLS to Secure the Site

- **Install Certbot:**

  ```bash
  sudo apt install certbot python3-certbot-nginx
  ```

- **Obtain an SSL Certificate:**

  ```bash
  sudo certbot --nginx -d your_domain.com -d www.your_domain.com
  ```

- Follow the instructions to complete the process and update the Nginx configuration to support `https`.

### 2. Set Up a Firewall

- Use `ufw` to manage the firewall and allow connections on necessary ports:

  ```bash
  sudo ufw allow 'Nginx Full'
  sudo ufw enable
  ```

### 3. Regularly Update the System and Packages

- Keep your system updated to ensure security and stability:

  ```bash
  sudo apt update
  sudo apt upgrade
  ```

### 4. Monitor Nginx Logs

- Check the error and access logs to identify any issues early:

  - Error logs: `/var/log/nginx/error.log`
  - Access logs: `/var/log/nginx/access.log`

---

## Conclusion

By following this guide, you can easily install and configure Nginx to link your application port with your domain on an Ubuntu VPS. This configuration allows you to present your application to users online in a secure and reliable manner.

Always remember to check configurations and system updates, and be prepared to handle any issues that may arise. By implementing the best practices mentioned, you will achieve better performance and provide an enhanced user experience.
