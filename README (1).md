# Areex Cloud — Deployment Guide

## Files
- `index.html` — Main website (homepage + payment)
- `admin.html` — Admin panel (plans, Razorpay config, settings)

---

## 🚀 Option 1: GitHub Pages (Free, Easiest)

1. Create a GitHub account at github.com
2. New repo → name it `areexcloud-site` (set to **Public**)
3. Upload both `index.html` and `admin.html`
4. Go to **Settings → Pages → Source: main branch / root**
5. Your site is live at `https://yourusername.github.io/areexcloud-site/`

### Connect your domain (AreexCloud.site):
1. In GitHub Pages settings, enter `AreexCloud.site` as custom domain
2. At your domain registrar (GoDaddy/Namecheap etc.), add these DNS records:
   ```
   Type: A      Name: @    Value: 185.199.108.153
   Type: A      Name: @    Value: 185.199.109.153
   Type: A      Name: @    Value: 185.199.110.153
   Type: A      Name: @    Value: 185.199.111.153
   Type: CNAME  Name: www  Value: yourusername.github.io
   ```
3. Wait 24–48 hours for DNS propagation
4. Enable **Enforce HTTPS** in GitHub Pages settings

---

## 🖥️ Option 2: VPS Deployment (Nginx)

```bash
# Install Nginx on Ubuntu/Debian
sudo apt update && sudo apt install nginx -y

# Create website folder
sudo mkdir -p /var/www/areexcloud

# Upload files (from your local machine)
scp index.html admin.html user@YOUR_VPS_IP:/var/www/areexcloud/

# Nginx config
sudo nano /etc/nginx/sites-available/areexcloud
```

Paste this Nginx config:
```nginx
server {
    listen 80;
    server_name areexcloud.site www.areexcloud.site;
    root /var/www/areexcloud;
    index index.html;
    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
# Enable site
sudo ln -s /etc/nginx/sites-available/areexcloud /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

# Free SSL with Certbot
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d areexcloud.site -d www.areexcloud.site
```

### VPS DNS Setup:
At your domain registrar, add:
```
Type: A    Name: @    Value: YOUR_VPS_IP
Type: A    Name: www  Value: YOUR_VPS_IP
```

---

## 💳 Razorpay Setup

1. Sign up at **dashboard.razorpay.com**
2. Go to **Settings → API Keys → Generate Live Key**
3. Copy your **Key ID** (starts with `rzp_live_`)
4. Open `index.html`, find this line and replace:
   ```js
   const RAZORPAY_KEY = 'rzp_test_REPLACE_WITH_YOUR_KEY';
   ```
   With:
   ```js
   const RAZORPAY_KEY = 'rzp_live_YOUR_ACTUAL_KEY_ID';
   ```
5. Save and re-upload the file

---

## 🔐 Admin Panel

- URL: `https://areexcloud.site/admin.html`
- Default login: `admin` / `areex2025`
- **Change your password immediately** in Settings tab!
- Use the Plans tab to add, edit, or delete hosting plans
- Changes sync to the main website automatically (uses browser localStorage)

---

## 📝 Notes

- The website uses `localStorage` to sync plan data between admin and the main site.
  This works perfectly when both pages are on the **same domain**.
- For production, consider adding a simple backend (Node.js/PHP) to persist data
  in a database and handle Razorpay webhook verification server-side.
- Razorpay webhook URL for order verification: `https://areexcloud.site/webhook` (requires backend)
