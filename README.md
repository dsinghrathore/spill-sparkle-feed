# Spill Sparkle Update Feed

This directory contains the static files that Spill uses for automatic updates via Sparkle.

## Structure

```
sparkle-feed/
├── appcast.xml          # Update feed XML (app checks this for new versions)
├── updates/             # Directory containing release ZIPs
│   ├── Spill-1.0.0.zip
│   ├── Spill-1.0.1.zip
│   └── ...
└── README.md           # This file
```

## Hosting on Dokploy

This folder should be deployed as a static site on your VPS at:
- **URL:** `https://ghq.rathoreactual.com/`
- **Path:** `/appcast.xml` should serve the appcast.xml file
- **Path:** `/updates/Spill-X.X.X.zip` should serve the update ZIPs

### Deployment Steps

1. **Create a new static site in Dokploy**
2. **Point it to this directory** (or a git repo containing this folder)
3. **Configure nginx/web server** to serve files at the correct URLs
4. **Ensure HTTPS is enabled** (Sparkle requires secure connections)

## appcast.xml

The `appcast.xml` file tells Spill about available updates. It contains:
- Version numbers
- Download URLs for each version
- EdDSA signatures (for security)
- Release notes
- Minimum system requirements

## Updates Directory

Place your signed release ZIPs here:
- `Spill-1.0.0.zip` - Initial release
- `Spill-1.0.1.zip` - Bug fixes
- `Spill-1.1.0.zip` - New features
- etc.

**Important:** Each ZIP must be:
1. Created with `ditto -c -k --sequesterRsrc --keepParent Spill.app Spill-X.X.X.zip`
2. Signed with `sign_update Spill-X.X.X.zip -f private-key.txt`
3. The app inside must be notarized by Apple

## Release Process

When releasing a new version:

1. **Build and notarize** the app in Xcode
2. **Create ZIP:**
   ```bash
   ditto -c -k --sequesterRsrc --keepParent "Spill.app" "Spill-1.0.1.zip"
   ```
3. **Sign ZIP:**
   ```bash
   ~/Downloads/bin/sign_update "Spill-1.0.1.zip" -f ~/path/to/sparkle-private-key.txt
   ```
   This outputs: `sparkle:edSignature="..." length="12345"`
4. **Upload ZIP** to `updates/` directory
5. **Update appcast.xml** with new version info and signature
6. **Deploy** updated appcast.xml to server
7. **Test** by running an older version and checking for updates

## Example appcast.xml Entry

```xml
<item>
    <title>Version 1.0.1</title>
    <link>https://ghq.rathoreactual.com/updates/Spill-1.0.1.zip</link>
    <sparkle:version>1.0.1</sparkle:version>
    <sparkle:shortVersionString>1.0.1</sparkle:shortVersionString>
    <sparkle:edSignature>abc123def456...</sparkle:edSignature>
    <sparkle:minimumSystemVersion>14.0</sparkle:minimumSystemVersion>
    <pubDate>Thu, 24 Oct 2025 12:00:00 +0000</pubDate>
    <enclosure
        url="https://ghq.rathoreactual.com/updates/Spill-1.0.1.zip"
        length="12345678"
        type="application/octet-stream"
        sparkle:edSignature="abc123def456..." />
    <description><![CDATA[
        <h2>What's New</h2>
        <ul>
            <li>Fixed voice recording bug</li>
            <li>Improved performance</li>
        </ul>
    ]]></description>
</item>
```

## Security

- ✅ All updates must be served over HTTPS
- ✅ All ZIPs must be signed with your EdDSA private key
- ✅ All apps must be notarized by Apple
- ✅ Sparkle will reject unsigned or tampered updates

## Public Key

Your app's public key (already in Info.plist):
```
5eg7ivqQ7IMWPyV9VtaBvDnZGu4Ks/C7IS+ZdniAJ7c=
```

This is used to verify that updates are signed with your private key.

## Private Key Location

The private key is stored at:
- `~/Desktop/sparkle-private-key.txt` (on your machine)
- Should also be in your password manager
- Should be shared securely with whoever handles releases

## Testing Updates Locally

You can test the update feed locally before deploying:

1. **Start a local server:**
   ```bash
   cd sparkle-feed
   python3 -m http.server 8000
   ```
2. **Update Info.plist** to point to local feed:
   ```xml
   <key>SUFeedURL</key>
   <string>http://localhost:8000/appcast.xml</string>
   ```
3. **Build and run** the app
4. **Check for updates** manually

## Nginx Configuration (Example)

If deploying to nginx on your VPS:

```nginx
server {
    listen 443 ssl http2;
    server_name ghq.rathoreactual.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    root /var/www/sparkle-feed;
    index appcast.xml;

    location / {
        try_files $uri $uri/ =404;
        add_header Cache-Control "no-cache, must-revalidate";
    }

    location /updates/ {
        add_header Cache-Control "public, max-age=31536000";
    }
}
```

## Resources

- [Sparkle Documentation](https://sparkle-project.org/documentation/)
- [Publishing Updates Guide](https://sparkle-project.org/documentation/publishing/)
- [Signing Updates](https://sparkle-project.org/documentation/signing/)
