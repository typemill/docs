# Integrating MCP into Typemill

I would implement MCP as a dedicated interface on top of Typemill's existing API, ideally packaged as a Typemill plugin. This would allow AI tools such as ChatGPT, Claude, and MCP-compatible development environments to work with Typemill content without requiring each client to implement its own API integration.

The important architectural decision is to keep the existing REST-style API intact and add an MCP server that exposes selected Typemill functionality as AI-callable tools.

Your [Typemill API documentation](https://docs.typemill.net/api) already provides a starting point: it documents the API's authentication and endpoints for navigation, article content, metadata, individual items, settings, and user queries.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.typemill.net\&sz=32)

Typemill



## 1. Recommended architecture

AI clients

ChatGPT · Claude · IDEs · Custom agents

MCP over Streamable HTTP

Typemill MCP server

Tool discovery · Input validation · Authentication · Permissions

Example: `https://your-site.example/mcp`

Typemill application layer

Existing API, content services, filesystem and plugin methods

MCP is not simply a standard JSON API. It defines a protocol for discovering and invoking tools, with JSON-RPC messages and standardized transports. For a remotely hosted CMS, I would start with Streamable HTTP rather than a locally launched process using stdio.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



There are two viable implementation strategies:

- Plugin-based MCP server — recommended: integrate MCP routes into Typemill so the server can use its authentication, permissions, content handling and plugin APIs.
- Standalone MCP adapter: run a separate PHP or Node.js service that calls Typemill's HTTP API. This is useful for a quick proof of concept, but introduces another service, credential management and a network boundary.

For a CMS intended to support many installations, the plugin approach provides a more natural integration. A separate adapter can still be useful as an initial prototype.

## 2. Map Typemill functionality to MCP tools

I would not expose every REST endpoint one-to-one. Instead, create tools that describe meaningful tasks in terms an AI model can understand.

The current documented API is a good foundation for a read-only first version. For example, Typemill already supports retrieving navigation, article content in Markdown and HTML, metadata, and article items.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.typemill.net\&sz=32)

Typemill

+4



Here is the initial tool set I would expose:

| MCP tool            | Purpose                                         | Access |
| ------------------- | ----------------------------------------------- | ------ |
| `list_pages`        | Retrieve the content navigation or page tree    | Read   |
| `get_page`          | Get a page's Markdown content                   | Read   |
| `find_page`         | Resolve a page using its slug                   | Read   |
| `get_page_metadata` | Read page metadata                              | Read   |
| `search_pages`      | Find pages matching a phrase or keyword         | Read   |
| `get_site_info`     | Return a limited set of public site information | Read   |

The first four can map closely to existing endpoints. `search_pages` may need to be implemented by traversing the content index and searching locally, depending on your current API capabilities. `get_site_info` should expose only deliberately selected fields, not all system settings.

Once this foundation works, add write tools:

| MCP tool               | Purpose                   | Suggested protection  |
| ---------------------- | ------------------------- | --------------------- |
| `create_page`          | Create a page or article  | Explicit confirmation |
| `update_page`          | Replace or modify content | Explicit confirmation |
| `update_page_metadata` | Change metadata           | Explicit confirmation |
| `publish_page`         | Publish a draft           | Explicit confirmation |
| `delete_page`          | Delete content            | Strong confirmation   |

These write tools require application functionality beyond what I could verify in the currently documented endpoint list. You would need to connect them to Typemill's internal content-handling methods or implement suitable API operations.

### Why tool design matters

Consider these two interfaces:

- `GET /api/v1/article/content?url=...`
- `get_page(slug, include_metadata)`

The first exposes an HTTP implementation detail. The second gives an AI model a clear, task-oriented capability.

An MCP tool definition also includes a description and a JSON Schema for its arguments. That schema helps the model choose the right tool and provide valid input.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub



## 3. Authentication and permissions

Typemill currently ties API access to user accounts. Its documentation specifies Basic Authentication, API access enabled on the account, and role-based authorization. It also recommends dedicated API users and restricting trusted IPs or hosts.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.typemill.net\&sz=32)

Typemill



I would preserve those permission boundaries rather than create a parallel authorization system that could accidentally grant additional access.

The security model should look like this:

- Each MCP connection is associated with an authenticated Typemill user.
- The server checks that user's permissions on every tool invocation.
- Read-only access is the default.
- Write access is granted only to appropriately authorized accounts.
- Passwords and API credentials are stored securely and never exposed in tool descriptions or results.
- Tool arguments are validated, calls are rate-limited, and content returned to the model is appropriately filtered.

For a first-party deployment, an MCP route could accept a dedicated bearer token or another securely configured credential and map it to a Typemill API identity. For a public, multi-user MCP service, implement the appropriate OAuth-based authorization flow instead of relying on a shared site-wide secret. The MCP specification distinguishes its HTTP authorization framework from authentication approaches used with local stdio servers.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



One important distinction: MCP tool descriptions are not a security boundary. A tool described as read-only must also be prevented from performing writes by its implementation and permissions.

## 4. Implement the MCP server in PHP

Since Typemill is PHP-based, I would investigate using the official PHP MCP SDK or a compatible maintained implementation rather than implementing JSON-RPC and MCP protocol handling from scratch.

The server needs to support at least:

- `initialize` and capability negotiation
- `tools/list` for tool discovery
- `tools/call` for execution
- Streamable HTTP transport
- Appropriate error responses and input validation

The protocol defines these operations and their expected behavior.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



The application-specific part should be relatively small: map tool names to Typemill methods and return results in MCP-compatible content blocks.

A simplified illustration of the application layer might look like this:

```
final class TypemillMcpTools
{
    public function __construct(
        private TypemillContentService $content
    ) {}

    public function getPage(string $slug): array
    {
        // Resolve the slug using Typemill's content index.
        // Check the current user's permissions.
        // Retrieve the Markdown content.
        // Return a normalized result.
        return $this->content->getBySlug($slug);
    }
}
```

This is illustrative, not a drop-in implementation: `TypemillContentService` and `getBySlug()` represent application-specific interfaces that would need to be connected to the actual Typemill codebase.

I would keep the protocol layer independent from content retrieval. This means you can later expose the same content services through REST, MCP, or other integrations without duplicating the underlying logic.

### Suggested plugin structure

Typemill MCP plugin

Plugin entry point

Register routes and initialize the MCP server through Typemill's plugin mechanisms.

Authentication and authorization

Authenticate requests and enforce user roles and API permissions.

Tool registry

Define tool names, descriptions, argument schemas and handlers.

Content adapter

Connect tools to existing API endpoints or internal content methods.

The exact route-registration and dependency-injection code should follow the existing Typemill plugin conventions. I would also verify that the chosen SDK supports your PHP version and Slim version before selecting it.

## 5. MCP tools, resources and prompts

MCP has more than one way to expose functionality. For Typemill, I would prioritize them as follows.

Tools — first priority

Let AI clients retrieve, create, update and publish content through controlled operations. These are the core capabilities for an AI-assisted CMS.

Resources — second priority

Expose readable content such as individual Markdown pages or site documentation through stable resource URIs. This can help clients browse knowledge bases without making every operation a tool call.

Prompts — optional

Provide reusable workflows such as reviewing an article, preparing a documentation update, or checking a page for broken internal references.

I would start with tools and add resources once the basic integration is stable. Prompts are useful for making the integration easier to use, but they are not required for an MCP server to be useful.

## 6. A practical implementation roadmap

1. Phase 1 — Read-only prototype

   Register an MCP endpoint, authenticate one dedicated Typemill user, and implement `list_pages`, `get_page` and `find_page`. Test discovery and execution with an MCP inspector.
2. Phase 2 — Complete content access

   Add metadata, content search, pagination and resources. Normalize responses so models receive Markdown and structured metadata instead of unnecessary API response details.
3. Phase 3 — Content editing

   Add create and update operations, draft handling and publishing. Implement permission checks, explicit confirmation for destructive actions, and safeguards against accidental overwrites.
4. Phase 4 — Distribution

   Document how to connect clients, support appropriate authentication, and test compatibility with several MCP clients. Package and distribute the plugin through the Typemill plugin ecosystem if appropriate.

For testing and production, also validate request origins, reject unauthorized requests, and avoid exposing the MCP endpoint without authentication. Streamable HTTP has specific transport security requirements, including validation of the `Origin` header when present.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://github.com\&sz=32)

GitHub

+1



## 7. One important product decision

I would make the MCP integration useful for both existing Typemill installations and future AI-driven workflows, without requiring users to deploy an external service.

That suggests a plugin with:

- A configurable enable/disable setting.
- Read-only access as the initial default.
- Dedicated credentials or an appropriate authorization flow.
- Fine-grained control over exposed tools.
- A clear distinction between reading content, editing drafts and publishing content.
- Compatibility with the existing API rather than a breaking API redesign.

The most important technical question is whether the MCP server should call Typemill's existing HTTP API or access its internal content services directly. My preference is internal services for a first-party plugin, with the existing API retained for external integrations. That avoids unnecessary HTTP round trips and makes it easier to implement operations that are not currently exposed through the public API.

Which implementation would you like to pursue first?

Implementation approach

Native Typemill plugin — integrated into Typemill itself

Standalone PHP MCP server — calls the existing API

Initial capabilities

Read-only — search and retrieve content

Read and edit — create and update pages

Full CMS control — including publishing and deletion

Connection method

Remote MCP over HTTP — for hosted Typemill websites

Local stdio server — for locally configured AI tools

Develop this implementation