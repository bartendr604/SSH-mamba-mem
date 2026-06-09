# SSH-mamba-memThat fits well with a Node-first architecture. A practical setup is: Node MCP client/agent on top, Node MCP server in the middle, and a Blender add-on or local socket bridge at the bottom that executes `bpy` work. [github](https://github.com/sinsunsan/blender-bonsai-bim-mcp)

## Recommended stack
- Client/agent runtime: Node.js with the MCP SDK or an AI SDK that can consume MCP tools. The Node ecosystem has working patterns for both stdio and HTTP-style MCP clients. [ai-sdk](https://ai-sdk.dev/cookbook/node/mcp-tools)
- MCP server: TypeScript/Node using `@modelcontextprotocol/sdk` plus validation like `zod`. A Blender-related Node MCP server example uses exactly that combination and communicates with Blender over `net` sockets. [github](https://github.com/sinsunsan/blender-bonsai-bim-mcp)
- Blender bridge: a Python add-on inside Blender that listens for commands and executes them on Blender’s main thread. This is the part that actually creates objects, assigns materials, renders, or exports. [mcpservers](https://mcpservers.org/servers/patrykiti/blender-ai-mcp)
- Local model runtime: your Mamba-based model running separately, then wrapped by the Node agent so it can decide when to call tools. MCP does not replace the agent layer; it sits under it as the tool protocol. [linkedin](https://www.linkedin.com/posts/akshay-pachaar_i-just-built-a-100-local-mcp-client-that-activity-7327327869188079616-qGth)

## Node-first architecture
1. User talks to the Node client.
2. The client sends prompts to the local Mamba model.
3. The model returns a tool intent.
4. The Node agent calls MCP tools on the server.
5. The Node MCP server forwards the action to Blender through a local socket or transport bridge.
6. Blender runs the command and returns structured results. [mcpservers](https://mcpservers.org/servers/patrykiti/blender-ai-mcp)

## What to build first
- A minimal MCP server with 5–10 tools like `create_cube`, `move_object`, `apply_material`, `render_scene`, and `export_file`. Keeping the tool surface small helps the model choose correctly. [reddit](https://www.reddit.com/r/mcp/comments/1ro7ifh/blender_mcp_pro_100_tools_mcp_server_for_blender/)
- A Blender add-on that exposes a single local port and queues requests safely.
- A Node agent that can read tool schemas and loop until the task is complete. Node MCP client tooling already supports retrieving and using tool sets from a server. [ai-sdk](https://ai-sdk.dev/cookbook/node/mcp-tools)

## Why Node works well here
Node is a good fit for the protocol and orchestration layers because MCP SDK support is mature there, and it’s easy to glue together local transports, validation, and agent logic. The Blender-side execution still stays in Python, which is usually the right split: Node for control, Python for Blender internals. [blender](https://www.blender.org/lab/mcp-server/)

## Good transport choice
For a local-only setup, stdio MCP on the Node side and a TCP or localhost socket bridge to Blender is the cleanest approach. If you later want remote control, you can add an HTTP or streamable transport layer in front of the same server. [youtube](https://www.youtube.com/watch?v=YHDS4Fo8Vls)

Would you like a concrete folder structure and package list for a Node + Blender MCP project?
