# Advanced Browser Fingerprinting & Bot Detector

A self-contained, client-side JavaScript tool designed to gather detailed browser telemetry and apply a configurable scoring system to distinguish between human users and automated bots.

![Screenshot of the Bot Detector UI]

## Features

-   **Detailed Client-Side Telemetry:** Gathers a wide range of browser information, including User-Agent, screen resolution, platform, and CPU cores.
-   **Advanced Fingerprinting:** Utilizes advanced techniques like Canvas, WebGL, and AudioContext fingerprinting to create a highly unique client profile.
-   **Server-Side Header Analysis:** Includes a mechanism to fetch and display HTTP headers as seen by the server (requires Nginx configuration), providing another layer of validation.
-   **Configurable Bot Scoring System:** Implements a flexible scoring system where different suspicious behaviors increment a "bot score."
-   **JavaScript Proof-of-Work (PoW):** Issues a small computational challenge to the browser to test for automation and environment legitimacy.
-   **Conditional CAPTCHA:** A simple CAPTCHA challenge is presented only if the client's bot score exceeds a defined threshold, ensuring minimal friction for legitimate users.
-   **Modern, Single-File Deployment:** All HTML, CSS, and JavaScript are contained within a single `index.html` file for easy deployment.

---

## How It Works

The script operates in a few key stages:

1.  **Telemetry Collection:** On page load, the JavaScript collects dozens of data points from the browser's `navigator`, `screen`, and other APIs. It also performs the advanced fingerprinting checks.
2.  **Header Fetching:** It makes an asynchronous `fetch` call to a server-side endpoint (`/api/headers`) to retrieve the HTTP headers from the initial request.
3.  **Bot Detection Analysis:** A series of checks are performed. Each check that indicates bot-like behavior (e.g., the presence of a `webdriver` flag, missing WebGL support, failing the PoW) adds points to a `botScore`.
4.  **Decision & Action:** The final `botScore` is compared against the `BOT_SCORE_THRESHOLD`. If the score is met or exceeded, the client is flagged as a potential bot, and the CAPTCHA challenge is displayed for further verification.

---

## Setup and Installation

This project requires a web server like Nginx to handle the server-side header API.

### Step 1: Deploy the Code

Place the `index.html` file in the root directory of your web server (e.g., `/var/www/html`).

### Step 2: Configure Nginx for Header API

For the "Server-Observed Headers" feature to work, you **must** configure your Nginx server to provide the headers at an API endpoint.

Open your site's Nginx configuration file and add the following `location` block inside your `server` block:

```nginx
# Add this location block inside your existing server { ... } block

location /api/headers {
    # Set the content type to JSON
    add_header 'Content-Type' 'application/json';
    
    # Return a 200 OK response with a JSON body containing the headers.
    return 200 '{
        "host": "$host",
        "user-agent": "$http_user_agent",
        "accept": "$http_accept",
        "accept-language": "$http_accept_language",
        "accept-encoding": "$http_accept_encoding",
        "dnt": "$http_dnt",
        "sec-ch-ua": "$http_sec_ch_ua",
        "sec-ch-ua-mobile": "$http_sec_ch_ua_mobile",
        "sec-ch-ua-platform": "$http_sec_ch_ua_platform",
        "remote-address": "$remote_addr"
    }';
}
```

### Step 3: Reload Nginx

After saving the configuration file, test and reload Nginx:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

### Step 4: Enable HTTPS (Recommended)

The Proof-of-Work challenge uses the `crypto.subtle` API, which requires a **secure context** to run over a network. It is highly recommended to secure your site with an SSL certificate (e.g., from Let's Encrypt using Certbot). The script includes a fallback but will be most effective over `https://`.

---

## Configuration

You can easily customize the detection sensitivity by editing the constants at the top of the `<script>` tag in `index.html`.

```javascript
// --- CONFIGURATION ---

// The score at which a client is considered a bot and needs a CAPTCHA.
// Lower this number to make detection MORE sensitive.
// Raise this number to make detection MORE lenient.
const BOT_SCORE_THRESHOLD = 5; 

// The computational difficulty for the Proof-of-Work challenge. Higher is harder.
const POW_DIFFICULTY = 4;
```

For more advanced control, you can modify the points awarded for each individual check within the `runBotDetection()` function.

## Disclaimer

Bot detection is an ever-evolving field. While this tool provides a strong baseline for identifying simple to moderately sophisticated bots, determined adversaries can spoof many of these signals. This tool should be part of a larger security strategy.

## License

Distributed under the MIT License. See `LICENSE` for more information.