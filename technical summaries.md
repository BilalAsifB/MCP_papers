# Securing the Model Context Protocol (MCP): Risks, Controls, and Governance

This paper goes way deeper into the threat models for MCP, and it made me realize how dangerous AI agents can be. The main
problem is that we are moving from static APIs where developers hard-code everything, to dynamic agent systems where the AI
just decides what to do based on the data it reads. They categorize the threats into three main types of "adversaries".

### *1. Content Injection:*
   Where a hacker hides malicious instructions inside normal data like a customer support ticket. The AI reads the ticket,
   gets confused, and executes the bad instructions. 
   
### *2. Supply Chain attackers:* 
   Which is when users download unofficial community MCP servers that secretly have malicious code in them. 

### *3. The Inadvertent Agent:*
   This is when the AI isn't malicious, but because it's trying so hard to complete its goal, it accidentally pulls sensitive
   data like API keys and leaks them. 

To fix all this, the authors say we need to stop giving AI "god mode" access. We needper-user authentication, strict sandboxing
so unvetted code doesn't wreck your computer, and inline policy enforcement tocatch data exfiltration.

---

# Model Context Protocol (MCP) at First Glance: Studying the Security and Maintainability of MCP Servers

Teh paper is a huge empirical study where the researchers looked at 1899 open source MCP servers. Foundation Models like GPT-4
are super smart but they need tools to interact with the real world, and Anthropic made the Model Context Protocol (MCP) to
standardize how this works. It got popular really fast. The researchers wanted to know if this ecosystem is actually healthy and
safe. Surprisingly, the health of these projects is pretty great! They found that MCP servers average about 5.5 commits per week,
which is way higher than the 2.5 commits you see in normal open-source software. But the security side is where it gets sketchy.
They ran static analysis tools and found that 7.2% of the servers have general vulnerabilities (mostly exposing credentials)
and 5.5% have MCP-specific issues called "tool poisoning". Tool poisoning is bad because it means the AI can be tricked. Also,
developers are kind of rushing these out because 66% of the servers have "code smells" (which just means the code is really
complex and hard to maintain) and 14.4% actually have bugs. Basically, MCP is growing fast but developers need to be way more
careful about security.

---

# A Survey on Model Context Protocol: Architecture, State-of-the-art, Challenges and Future Directions

This paper gave a detailed architectural overview of how MCP actually works under the hood. Basically, older protocols like
REST APIs were too rigid and stateless to handle AI conversations that need to constantly fetch context. MCP uses JSON-RPC 2.0
to fix this, making the communication stateful and structured.The system is split into three main parts. There is the Host
(which orchestrates everything, manages the UI, and handles security), the Client (which keeps an isolated 1-to-1 session with
a server), and the Server (which exposes the actual features). When the client and server connect, they do a "capability
negotiation" phase where they figure out what features each other supports before they start talking. Servers can provide
Prompts (templates), Resources (context data), Tools (executable functions), and Utilities like logging. For the actual transport
layer, they can talk locally using stdio (which is super fast and runs the server as a background process) or remotely using
Streamable HTTP with SSE (which is better for cloud servers and asynchronous tasks). It's a really solid framework for the
future, but they noted it still has challenges with latency and keeping the context windows from getting overloaded.

---

# MCP Safety Audit: LLMs with the Model Context Protocol Allow Major Security Exploits

This paper demonstrated real hacks on Claude and Llama using MCP. The researchers proved that you can easily coerce these
top-tier LLMs to perform Malicious Code Execution (MCE), Remote Access Control (RAC), and Credential Theft (CT) just by using
tools from default MCP servers. Sometimes the LLM's built-in safety guardrails would kick in and it would refuse the request,
but if the attacker just tweaked the prompt a little bit, the AI would go ahead and do the hack anyway. They also showed off
a crazy new attack called RADE (Retrieval-Agent Deception). Basically, an attacker poisons a public file with hidden commands,
the user adds it to their Chroma vector database, and when the user asks the AI to search the database, the AI reads the
hidden commands and automatically leaks the user's API keys over Slack. Since relying on the AI to refuse bad prompts clearly
doesn't work 100% of the time, the researchers built a tool called McpSafetyScanner. It uses multiple AI agents (like a hacker
and an auditor) to automatically probe an MCP server, find its vulnerabilities, and write a security report so developers can
patch it before deployment.

---
