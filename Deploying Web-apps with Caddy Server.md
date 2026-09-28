# Deploying Web Applications with Caddy: A Beginner's Guide to Web Servers

## Introduction

This documentation is designed to remove the fear around servers for beginners. It covers web servers — installing and configuring them — using examples like Nginx and Caddy, comparing the differences between them, and walking through how SSL certificates (HTTPS) get configured. By the end, you'll understand how to deploy a basic Node.js, React, or Vue application to a server, along with a few interesting related concepts.

## Setting Up the Server

### Choosing a VPS Provider

This walkthrough uses **Hostinger** as the server provider, primarily because it supports direct UPI payment, making it quick and convenient to spin up a small server for experimentation.

> **Key concept:** Regardless of which company or provider you use, what you ultimately get is a machine — commonly called a **VPS** (Virtual Private Server). Different providers use different names (e.g., EC2 on AWS, VPS elsewhere), but fundamentally it's a virtual machine that stays always on, with configurable RAM, CPU, and GPU based on what you're willing to pay for.

### Exploring What's Possible: Anaconda and Jitsi

Before diving into the main topic, a couple of side experiments are shown:

- **Anaconda**: Testing various tools and setups that could be built and demonstrated in future videos.
- **Jitsi**: An open-source video conferencing project, similar to Google Meet or Zoom, that can be self-hosted. On Hostinger, this is available as a one-click deployment under the "Media" category in the application marketplace. You simply select it, choose your machine size, and your own Google Meet-like platform is ready to go.

> **Note:** Jitsi is a more complex project since it's a full media server, not just a web server, but Hostinger simplifies deployment via one-click installation. Jitsi supports features like screen sharing, presentations, document editing, and has mobile apps on Google Play and the App Store, with source code available on GitHub (built in TypeScript).

### Spinning Up a Plain Ubuntu Machine

For this tutorial, a plain OS (Ubuntu) is needed rather than a pre-configured application template. Steps include:

1. Choose a plain Ubuntu OS template (best available version).
2. Add a password (or preferably SSH keys, which are more secure).
3. Skip unnecessary add-ons like malware scanners.
4. If reusing an existing machine, go to the OS & Panel section and select "Change OS" to reinstall a plain Ubuntu OS — this will wipe existing data.

> **Note:** This process typically takes about 10 minutes.

## Understanding Web Servers

There are several popular web servers available: **Apache** (the most commonly known), **Nginx** (another very common and widely used server), and **Caddy**.

> **Key point:** While Nginx is excellent and widely used, Caddy has become a favorite for many because it provides **HTTPS out of the box**, making sites more secure, reliable, and scalable — with remarkably small and simple configuration files.

## Installing Caddy

Caddy's official documentation provides ready-to-use install commands for Debian/Ubuntu systems — no additional configuration needed. The commands can be copy-pasted directly from the [Caddy website](https://caddyserver.com/) and run in the terminal.

Once installed, verify the installation by checking the version:

```bash
caddy version
```

## Preparing a Web Application to Serve

### Understanding the Build Process

Before serving any frontend application (React, Vue, etc.), it's important to understand a key concept:

- When you run `npm run dev`, it starts a **development server**.
- In production, you instead run `npm run build`, which compiles all your source code (TypeScript/JavaScript) into static **HTML, CSS, and JS** files.

> **Key point:** These generated static files (HTML, CSS, JS) are what ultimately get served to the browser — whether the original application was built with React, Next.js, or anything else, because the browser only understands HTML, CSS, and JS.

### Getting the Code onto the Server

To move code onto the server:

1. Push your project code to GitHub.
2. Install Git on the Linux server (`sudo apt-get install git`).
3. Clone the public repository using `git clone <repo-url>`.
4. Install Node.js on the server (installation commands are readily available by searching "Node.js Ubuntu install").
5. Run `npm install` followed by `npm run build` to generate the static HTML/CSS/JS output.

### Creating a Directory to Serve Files

A simple example is shown using two static files instead of a full framework build:

1. Create a directory under `/var/www/` (the conventional location for web server content):

```bash
mkdir -p /var/www/chai-code
cd /var/www/chai-code
```

2. Create two files: `index.html` and `style.css`, using the `nano` editor to paste in the content.

```bash
nano index.html
# Paste content, then Ctrl+O to save, Enter, Ctrl+X to exit

nano style.css
# Paste content, then Ctrl+O to save, Enter, Ctrl+X to exit
```

## Configuring Caddy

### Locating the Caddyfile

Caddy's configuration file is located at:

```bash
cd /etc/caddy
ls
sudo nano Caddyfile
```

### Setting Up the Domain and Serving Files

The default Caddyfile configuration typically references a placeholder domain (like `:80`) and a default file path (like `/usr/share`). To customize it:

1. **Get your server's public IP address** from the Hostinger dashboard overview page.
2. **Add an A record** at your domain registrar (GoDaddy, Cloudflare, Hostinger, etc.) pointing your domain or subdomain (e.g., `meet.chaicode.com`) to this public IP address.
3. **Update the Caddyfile** to point to your domain and your custom file path instead of the default:

```caddyfile
meet.chaicode.com {
    root * /var/www/chai-code
    file_server
}
```

4. Save the file (`Ctrl+O`, Enter, `Ctrl+X`).

> **Note:** DNS propagation for an A record can take anywhere from a few minutes to 10–15 minutes, though it's usually quite fast nowadays.

### Reloading Caddy

After changing the configuration, Caddy needs to be reloaded:

```bash
sudo systemctl reload caddy
```

Verify the service status:

```bash
sudo systemctl status caddy
```

Press `Q` to exit the status view. An "active" status confirms Caddy is running correctly.

> **Key benefit:** Caddy automatically provisions SSL/TLS certificates out of the box — HTTPS is enabled without any extra configuration.

## Serving Applications with a Process Manager (PM2)

For applications that need to run continuously (like a Node.js backend), **PM2** is commonly used instead of plain `npm`, since it provides more control and options for process management.

### Reverse Proxy Configuration for PM2-Managed Apps

Caddy makes it simple to reverse-proxy requests to an application running on a specific local port:

```caddyfile
example.com {
    reverse_proxy localhost:3000
}
```

Simply replace `example.com` with your actual domain and the port number with whatever port your application is running on. Multiple applications on different domains can be configured similarly, each pointing to its own internal port (e.g., 3000, 3001, 4000, 4005), all within the same Caddyfile.

> **Key point:** This is remarkably simple — no complex configuration is required to run and expose multiple applications on a single server.

## Why Caddy and Hostinger Work Well Together

Hostinger makes it easy to repeatedly experiment with a single machine — restarting it, reinstalling the OS, and trying new setups — which is ideal for learning and testing. Combined with Caddy's simple configuration and built-in HTTPS support, this creates a low-friction environment for deploying and experimenting with web applications.

## Closing Thoughts

This walkthrough demonstrated installing Caddy, preparing static files, configuring a domain with an A record, and reloading Caddy to serve a basic site with HTTPS enabled by default. The same approach extends easily to serving React, Vue, or Node.js applications — either by building static files directly or by using a reverse proxy with a process manager like PM2 for applications that need to run as persistent processes.
