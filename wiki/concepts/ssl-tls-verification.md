---
sources: [summaries/top_level.md]
brief: SSL/TLS verification ensures secure HTTPS connections by validating server certificates against trusted CAs.
---

# SSL/TLS Certificate Verification

SSL/TLS certificate verification is the process by which a client confirms the authenticity of a remote server during an HTTPS connection. This is accomplished by checking the server's certificate against a set of trusted Root Certificates maintained by recognized Certificate Authorities (CAs).

## How It Works

1. **Handshake**: When a client connects to an HTTPS server, the server presents its SSL/TLS certificate.
2. **Chain of Trust**: The client traces the certificate back through a chain of intermediate CAs to a trusted Root CA.
3. **Validation**: If the root CA is in the client's trusted CA bundle, the connection is considered secure.
4. **Failure Handling**: If no trusted root is found, the connection is rejected or a warning is raised.

## Root Certificate Bundles

Root certificate bundles are curated collections of trusted CA certificates. The most widely used is the **Mozilla CA Certificate Store**, which is bundled and distributed by the Python package **certifi** (see [[summaries/top_level]]).

## certifi and Python

The `certifi` package solves a common cross-platform problem: different operating systems ship different CA bundles, which can lead to inconsistent SSL/TLS behavior in Python applications. By depending on `certifi`, Python libraries such as `requests`, `urllib3`, and `httpx` gain access to a reliable, up-to-date CA bundle regardless of the host OS.

```python
import certifi
import ssl

ssl_context = ssl.create_default_context(cafile=certifi.where())
```

## Relevance in Networked Applications

SSL/TLS verification is critical whenever an application communicates over HTTPS, including:

- REST API clients
- Web scraping tools
- AI/LLM API integrations (e.g., calls to OpenAI, Anthropic, etc.)
- Package managers and dependency fetchers

In the context of this knowledge base, SSL/TLS verification underpins secure communication in [[concepts/python-networking]] stacks and any service using an [[concepts/openai-compatible-api]].

## Related Concepts

- [[concepts/python-networking]] — Python libraries that rely on certifi for secure connections
- [[concepts/ssl-tls-verification]] — Canonical concept page for this topic
- [[concepts/security-policy]] — Broader security practices
- [[concepts/vulnerability-disclosure]] — Responsible disclosure of security issues
- [[concepts/local-llm-inference]] — Local inference may bypass external TLS but still requires verification for API calls
- [[summaries/top_level]] — Source document referencing the certifi package
