# NIRMAYA Property Inspection — Vercel & Hostinger Deployment Guide

This guide explains how to fix the `404: NOT_FOUND` deployment error on Vercel and configure your custom Hostinger domain to point to your deployed website.

---

## 1. Local Codebase Fix (Already Done)
Because this project contains a `package.json` file, Vercel automatically detects it as a Node.js project. It then attempts to run a build script and expects build output in a subfolder like `dist` or `public`. Since this website is a static HTML/JS site, Vercel was deploying an empty directory, resulting in 404 errors.

To resolve this, we added a build script to the `scripts` section of your `package.json` file:
```json
  "scripts": {
    "build": "echo 'No build step required'",
    "dev": "npx -y http-server -p 3000 -c-1",
    "start": "npx -y http-server -p 3000 -c-1"
  }
```
*If you are deploying via a Git repository (like GitHub), make sure to commit and push this update to your remote repository.*

---

## 2. Configuring Vercel Project Settings

To ensure Vercel serves the root folder containing your static HTML files, configure your project settings in the Vercel Dashboard:

1. Open the [Vercel Dashboard](https://vercel.com/dashboard) and click on your project.
2. Click on the **Settings** tab at the top.
3. Select **General** from the left sidebar.
4. Scroll down to the **Build & Development Settings** section:
   * **Framework Preset**: Select **Other**.
   * **Build Command**: Toggle the **Override** switch to **ON** and write `npm run build`.
   * **Output Directory**: Toggle the **Override** switch to **ON** and type `.` (a single period, which tells Vercel to serve the root directory containing your HTML files).
5. Scroll up and check the **Root Directory** setting. If your Git repository contains the project files inside a subfolder (e.g. `New folder` or `Nirmayapropertyinspection`), set this to that folder name. Otherwise, keep it blank.
6. Click **Save**.
7. Go to the **Deployments** tab at the top, select your latest deployment, click the three dots (`...`) on the right, and click **Redeploy**. This will rebuild and deploy the files correctly.

---

## 3. Connecting Your Hostinger Domain to Vercel

To point your custom domain (purchased on Hostinger) to your Vercel deployment:

### Part A: Add the Domain in Vercel
1. Go to your project in the **Vercel Dashboard**.
2. Click **Settings** > **Domains**.
3. Type your custom domain name (e.g., `nirmayapropertyinspection.com` or `nirmaya.in`) and click **Add**.
4. Vercel will recommend adding both the apex domain (`yourdomain.com`) and the subdomain (`www.yourdomain.com`) and setting a redirect between them. Select the recommended setting.
5. Vercel will display an **Invalid Configuration** warning with the exact DNS records you need to add to your registrar. Keep this page open.

### Part B: Update DNS Records in Hostinger hPanel
1. Log in to your **Hostinger hPanel**.
2. Go to **Domains and select your custom domain**.
3. Select **DNS / Nameservers** from the left menu.
4. In the **DNS Zone Editor**, add/update the following two records:

#### 1. Apex Domain A Record:
* **Type**: `A`
* **Name (Host)**: `@`
* **Points to (Value)**: `76.76.21.21`
* **TTL**: Default (usually 14400 or 3600)
* *Note: If an A record with Name `@` already exists, edit it or delete the old one first.*

#### 2. Subdomain CNAME Record:
* **Type**: `CNAME`
* **Name (Host)**: `www`
* **Points to (Target)**: `cname.vercel-dns.com`
* **TTL**: Default
* *Note: If a CNAME record with Name `www` already exists, edit it or delete the old one first.*

### Part C: Verify the Domain Setup
Go back to the **Domains** tab in your **Vercel Project Settings**. Vercel will poll your domain's DNS. Once the changes propagate (which usually takes 5-15 minutes, but can take up to 24 hours), the warning status will change to a green **Active** status. Vercel will automatically issue and renew a free SSL security certificate for your domain.
