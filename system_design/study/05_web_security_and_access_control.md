# 05 — Web Security & Access Control (Part 1 of 3)
*(Sections 1–3 | Extended Bilingual Concepts Version)*

---

## 1. Web Security Fundamentals — CIA Triad & Threat Modeling

Modern web security is guided by the **CIA triad** — *Confidentiality*, *Integrity*, and *Availability*, and enhanced by structured threat modeling approaches such as **STRIDE**.

| Principle | Description | Example |
|------------|--------------|----------|
| **Confidentiality** | Prevent unauthorized access to sensitive information | HTTPS, encryption-at-rest |
| **Integrity** | Prevent unauthorized modification of data | Hashing, HMAC, digital signatures |
| **Availability** | Ensure systems remain accessible under attack | DDoS mitigation, load balancing |

> 💡 **中文解释：** Web 安全的三大核心目标：机密性（防止数据被未授权访问）、完整性（防止篡改）、可用性（防止攻击导致服务中断）。

### STRIDE Threat Model

| Threat Type | Description | Example |
|--------------|-------------|----------|
| **S**poofing | Impersonating identity | Fake login page |
| **T**ampering | Altering data in transit | Modify form data |
| **R**epudiation | Denying user actions | Missing logs |
| **I**nformation Disclosure | Leaking sensitive info | XSS stealing tokens |
| **D**enial of Service | Making system unavailable | Flooding API |
| **E**levation of Privilege | Gaining admin rights | Privilege escalation |

> 💡 **中文解释：** STRIDE 是常用威胁建模方法，用于系统性识别安全风险。

---

## 2. Same-Origin Policy & CORS — Deep Dive

### 2.1 The Same-Origin Policy (SOP)

**Origin = protocol + hostname + port**  
Example:  
`https://app.example.com:443` ≠ `https://api.example.com:443` (different subdomain)

The SOP prevents one origin from reading data from another without permission.

> 💡 **中文解释：** 同源策略限制脚本跨域访问其他源的数据，是浏览器安全的基石。

### 2.2 Cross-Origin Resource Sharing (CORS)

CORS allows controlled access between origins via specific HTTP headers.

**Preflight Request Flow:**  
```
Client:  OPTIONS /api/data
Headers:
  Origin: https://frontend.example.com
  Access-Control-Request-Method: GET

Server:  200 OK
Headers:
  Access-Control-Allow-Origin: https://frontend.example.com
  Access-Control-Allow-Methods: GET, POST
  Access-Control-Allow-Credentials: true
```

> 💡 **中文解释：** CORS 通过服务器返回的 HTTP 响应头告知浏览器哪些源允许访问，从而安全地实现跨域请求。

**Key Headers:**

| Header | Purpose | Example |
|---------|----------|----------|
| `Access-Control-Allow-Origin` | Whitelist allowed origins | `https://app.example.com` |
| `Access-Control-Allow-Credentials` | Allow cookies/auth | `true` |
| `Access-Control-Allow-Headers` | Allowed request headers | `Content-Type, Authorization` |
| `Access-Control-Max-Age` | Cache preflight result | `3600` |

> 💡 **中文解释：** “预检请求（OPTIONS）”是 CORS 的关键环节，用于确保跨域安全。

---

## 3. Cross-Site Scripting (XSS) — Attacks and Defense

### 3.1 Types of XSS

| Type | Description | Example |
|------|--------------|----------|
| **Reflected** | Script injected via request parameters | `?q=<script>alert(1)</script>` |
| **Stored** | Script stored in database (e.g., comment field) | Persistent attack |
| **DOM-based** | Client-side JS manipulates DOM unsafely | `element.innerHTML = location.hash` |

> 💡 **中文解释：** XSS 分为反射型、存储型和 DOM 型，分别利用输入注入、持久化内容或前端逻辑漏洞。

### 3.2 Mitigation Strategies

1. **Output Encoding:** Escape HTML special characters (`<`, `>`, `"`, `'`).  
2. **Input Sanitization:** Strip unwanted tags using `DOMPurify`.  
3. **Content Security Policy (CSP):** Whitelist trusted sources.  
4. **Avoid `dangerouslySetInnerHTML` in React.**  
5. **Use Trusted Types API (for modern browsers).**  

> 💡 **中文解释：** 防御 XSS 的关键是转义输出、过滤输入、限制外部脚本来源。React 等框架提供了内建防御机制。

### 3.3 React / Next.js Practical Example

```jsx
import DOMPurify from 'dompurify';

function SafeContent({ html }) {
  const clean = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

> 💡 **中文解释：** 在 React 中可通过 DOMPurify 先清理 HTML，再使用 `dangerouslySetInnerHTML` 渲染，从而防止恶意脚本执行。

### 3.4 Browser Enforcement — CSP + Trusted Types

```http
Content-Security-Policy: script-src 'self'; require-trusted-types-for 'script';
```

> 💡 **中文解释：** CSP 可结合 Trusted Types 使用，彻底禁止非受信脚本注入，是现代浏览器最强的 XSS 防御。

---

**End of Part 1 (Sections 1–3)**

# 05 — Web Security & Access Control (Part 2 of 3)
*(Sections 4–6 | Extended Bilingual Concepts Version)*

---

## 4. Cross-Site Request Forgery (CSRF)

CSRF forces an authenticated user’s browser to send unauthorized requests to a trusted site.

### 4.1 Attack Example
```html
<!-- Malicious site -->
<img src="https://bank.com/transfer?amount=1000&to=hacker" />
```
When the victim is logged in to `bank.com`, the browser automatically includes cookies, completing the forged transaction.

### 4.2 Mitigation Strategies

| Method | Description |
|---------|-------------|
| **CSRF Token** | Random token stored in session and embedded in forms |
| **SameSite Cookie** | Restricts cross-origin cookie sending |
| **Double Submit Cookie** | Token stored both in cookie and request header |
| **Origin/Referer Check** | Verify request source |

> 💡 **中文解释：** CSRF 通过伪造用户请求实现攻击。可通过 CSRF Token、SameSite Cookie 和来源检查防御。

**Token Example (Express.js):**
```js
app.use(csrf());
app.get('/form', (req, res) => {
  res.render('send', { csrfToken: req.csrfToken() });
});
```

**SameSite Cookie Options:**
| Value | Behavior |
|--------|-----------|
| `Strict` | Cookies sent only to same-site requests |
| `Lax` | Cookies sent for top-level navigation (default) |
| `None` | Cookies sent cross-site (must be Secure) |

> 💡 **中文解释：** SameSite Cookie 可防止第三方请求携带 Cookie，是现代浏览器防御 CSRF 的关键策略。

---

## 5. Content Security Policy (CSP)

CSP allows web developers to control which resources can be loaded and executed.

### 5.1 Basic Policy
```http
Content-Security-Policy: default-src 'self'; img-src https://cdn.example.com; script-src 'self' 'nonce-abc123';
```

> 💡 **中文解释：** CSP 用于定义资源加载白名单。`'self'` 表示仅允许本站资源，`nonce` 用于指定唯一授权脚本。

### 5.2 Nonce vs Hash Strategies

| Method | Description | Example |
|---------|-------------|----------|
| **Nonce** | Random per-response key attached to allowed scripts | `script-src 'nonce-r4nd0m'` |
| **Hash** | Allows inline scripts matching a specific SHA hash | `script-src 'sha256-AbCdEfGh...'` |

> 💡 **中文解释：** Nonce 和 Hash 是两种授权内联脚本方式，前者随机、后者基于内容校验。

### 5.3 Reporting Violations
```http
Content-Security-Policy: default-src 'self'; report-uri /csp-violation
```
Browsers will POST a JSON report on blocked scripts or resources.

> 💡 **中文解释：** CSP 报告机制可收集潜在攻击或配置错误信息，常用于监控。

### 5.4 React Integration Example
```jsx
<script nonce={nonce}>
  {`console.log("Safe inline script");`}
</script>
```

> 💡 **中文解释：** React 中可通过 `nonce` 属性与后端配合动态生成安全脚本标识。

---

## 6. Authentication & Authorization Mechanisms

### 6.1 Cookie / Session Authentication

| Aspect | Description |
|---------|--------------|
| **Mechanism** | Server stores session data; browser keeps session ID in cookie |
| **Pros** | Simple, mature, automatic cookie handling |
| **Cons** | Server stateful; scaling requires session replication |

```http
Set-Cookie: sessionId=abc123; HttpOnly; Secure; SameSite=Lax
```

> 💡 **中文解释：** 基于 Session 的认证需要服务器维护状态；适合传统单体架构。

---

### 6.2 JSON Web Token (JWT)

JWT provides stateless authentication. It contains claims encoded in Base64.

**Structure:**
```
header.payload.signature
```

**Payload Example:**
```json
{
  "sub": "user123",
  "role": "admin",
  "exp": 1731730800
}
```

**Flow:**
1. User logs in and receives JWT.  
2. JWT sent in `Authorization: Bearer <token>` header.  
3. Server validates signature with secret/public key.  

> 💡 **中文解释：** JWT 是无状态认证机制，客户端携带 Token 完成认证，服务器仅验证签名。

**Pros:** Stateless, scalable.  
**Cons:** Hard to revoke early; large token size.

---

### 6.3 OAuth 2.0 / OpenID Connect (OIDC)

OAuth 2.0 is used for delegated authorization; OIDC extends it for authentication.

**Authorization Code Flow (with PKCE):**
```
1. User → /authorize?client_id&code_challenge
2. Authorization Server → Redirect with code
3. Client → /token (with code_verifier)
4. Server → Issues Access + Refresh tokens
```

> 💡 **中文解释：** OAuth2 授权码流程（配合 PKCE）用于安全授权第三方应用访问资源，OpenID Connect 在此基础上添加身份认证。

**Tokens:**
| Token Type | Purpose | Lifetime |
|-------------|----------|-----------|
| **Access Token** | Authorize API access | Minutes |
| **Refresh Token** | Get new Access Token | Days/weeks |
| **ID Token (OIDC)** | Identify user | Short |

**Example Authorization Header:**
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Refresh Token Flow:**
```http
POST /token
grant_type=refresh_token&refresh_token=abc123
```

> 💡 **中文解释：** Access Token 用于访问 API，Refresh Token 可长期使用以续期，ID Token 用于识别身份。

---

**End of Part 2 (Sections 4–6)**

# 05 — Web Security & Access Control (Part 3 of 3)
*(Sections 7–9 + Interview Summary | Extended Bilingual Concepts Version)*

---

## 7. OAuth2 Threats and Security Enhancements

Even secure protocols like OAuth 2.0 can be misused. Understanding common attacks helps in designing safer systems.

### 7.1 Token Leakage

Tokens exposed through browser history, logs, or referrer headers can be stolen.

**Mitigation:**
- Use `response_mode=form_post` instead of URL fragments.  
- Avoid including tokens in query params.  
- Apply short token lifetimes.

> 💡 **中文解释：** Token 泄漏是 OAuth2 的主要风险，应避免在 URL 中传递 Token，且 Access Token 应短期有效。

### 7.2 Redirect URI Manipulation

Attackers can modify redirect URIs to capture authorization codes.

**Mitigation:**
- Pre-register redirect URIs.  
- Validate exact match on the server.  

> 💡 **中文解释：** OAuth2 中应严格校验 Redirect URI，防止攻击者伪造回调地址窃取授权码。

### 7.3 CSRF in Authorization Flow

Attackers trick users into authorizing unintended clients.

**Mitigation:**
- Include `state` parameter with random nonce.  
- Validate returned `state`.  

> 💡 **中文解释：** OAuth2 流程中通过随机 state 参数防止授权劫持。

### 7.4 PKCE (Proof Key for Code Exchange)

PKCE prevents authorization code interception in public clients (SPAs).

**Flow:**
```
1. Client → Create code_verifier + code_challenge
2. /authorize?code_challenge=HASH(verifier)
3. Exchange code with /token (verifier)
4. Server validates HASH(verifier)
```

> 💡 **中文解释：** PKCE 通过校验 code_challenge/code_verifier 确保授权码仅能被合法客户端使用。

---

## 8. Front-End Security Best Practices

### 8.1 General Principles

1. Always use **HTTPS** and enable **HSTS**.  
2. Sanitize and escape all user input/output.  
3. Limit third-party scripts; prefer subresource integrity (SRI).  
4. Use **CSP**, **X-Frame-Options**, and **X-Content-Type-Options**.  
5. Enable security linters and dependency audits.  
6. Keep all libraries updated (npm/yarn audit).  
7. Log and monitor anomalies (Sentry, Datadog, Cloudflare RUM).  

> 💡 **中文解释：** 前端安全需从传输层、输入验证、依赖管理等多方面防御；持续监控与审计尤为重要。

### 8.2 Dependency & Build-Time Security

- Use **npm audit**, **OWASP Dependency Check**, or **Snyk**.  
- Lock dependencies with `package-lock.json` or `yarn.lock`.  
- Use **webpack Subresource Integrity (SRI)** to ensure scripts aren’t tampered.  

```html
<script src="/bundle.js"
  integrity="sha384-abcd..." crossorigin="anonymous"></script>
```

> 💡 **中文解释：** SRI 校验可防止外部脚本被篡改，构建工具可自动插入 integrity 属性。

### 8.3 Continuous Security Testing

| Type | Tool | Purpose |
|------|------|----------|
| **SAST** | SonarQube, CodeQL | Analyze code for vulnerabilities |
| **DAST** | OWASP ZAP, Burp Suite | Test running app for attacks |
| **IAST** | Contrast Security | Combine runtime and static testing |
| **RASP** | Runtime protection agent | Block live attacks |

> 💡 **中文解释：** SAST/DAST/IAST/RASP 构成现代应用安全测试体系，涵盖代码、运行时与实时防御。

---

## 9. Interview-Oriented Summary

### 9.1 Core Concepts to Explain Clearly

| Topic | Key Points |
|--------|-------------|
| **SOP & CORS** | Describe preflight flow and headers |
| **XSS vs CSRF** | XSS injects code; CSRF exploits trust |
| **CSP** | Restricts external scripts; use nonce/hash |
| **JWT vs Session** | Stateless vs stateful authentication |
| **OAuth2** | Delegated access with Access/Refresh tokens |

> 💡 **中文解释：** 面试中考官更关注候选人是否理解安全机制背后的原理与设计权衡。

### 9.2 Security Design Checklist

1. Identify sensitive data and attack surfaces.  
2. Apply layered defenses (Defense in Depth).  
3. Use proper HTTP headers (CSP, HSTS, Referrer-Policy).  
4. Implement secure authentication and token rotation.  
5. Continuously monitor, log, and audit.  

### 9.3 Example Interview Answer

> “I design security from the browser outward: enforce Same-Origin Policy, apply strict CSP, secure cookies with SameSite+HttpOnly, and use token-based authentication (JWT or OAuth2). I also integrate monitoring and audit logs for observability.”

> 💡 **中文解释：** 回答时应从架构角度描述防御策略，强调安全分层与监控机制。

---

**End of Part 3 (Sections 7–9 + Interview Summary)**

