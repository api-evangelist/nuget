---
title: "Announcing the NuGet MCP Server Preview"
url: "https://devblogs.microsoft.com/dotnet/nuget-mcp-server-preview/"
date: "Thu, 14 Aug 2025 17:00:00 +0000"
author: "Jeff Kluge"
feed_url: "https://devblogs.microsoft.com/dotnet/category/nuget/feed/"
---
<p>Last month, we announced support for <a href="https://devblogs.microsoft.com/dotnet/mcp-server-dotnet-nuget-quickstart/">building custom MCP servers with .NET and publishing them to NuGet</a>.
Building on that foundation, today we&#8217;re announcing the official NuGet MCP Server, which enables you to integrate real-time NuGet package information and management tools directly into your AI-powered development workflow.</p>
<h2>What is an MCP Server?</h2>
<p>The <a href="https://modelcontextprotocol.io/docs/getting-started/intro">Model Context Protocol</a> (MCP) is an open standard that enables AI assistants to securely connect to external data sources and tools.
MCP servers act as bridges between AI assistants and various services, allowing for seamless integration and enhanced functionality.</p>
<p>Since the NuGet package ecosystem is always evolving, large language models (LLMs) get out-of-date over time and there is a need for something that assists them in getting information in realtime.
The NuGet MCP server provides LLMs with information about new and updated packages that have been published after the models as well as tools to complete package management tasks.
It also can connect to the feeds you have configured including private feeds, allowing for a more complete view of your packages.  The NuGet MCP server utilizes an <a href="https://devblogs.microsoft.com/dotnet/introducing-nugetsolver-a-powerful-tool-for-resolving-nuget-dependency-conflicts-in-visual-studio/">algorithm called NuGetSolver</a> which was developed in collaboration with Microsoft Research which simplifies the process by automatically resolving NuGet dependency conflicts in your projects.</p>
<h2>Getting Started</h2>
<p>The NuGet MCP server is available as a <a href="https://www.nuget.org/packages/NuGet.Mcp.Server">NuGet package</a> with the <a href="https://devblogs.microsoft.com/dotnet/mcp-server-dotnet-nuget-quickstart/">new functionality</a> in the .NET ecosystem for implementing and releasing MCP servers.</p>
<h3>Prerequisites</h3>
<p>Before you can use the NuGet MCP server, you will need to have <strong>.NET 10 Preview 6</strong> installed on your system.
You can download it from the <a href="https://dotnet.microsoft.com/download/dotnet/10.0">official .NET download page</a>.</p>
<h3>Configuration</h3>
<p>To configure the NuGet MCP server, add the following configuration to your MCP client:</p>
<pre><code class="language-json">{
  "servers": {
    "nuget": {
      "type": "stdio",
      "command": "dnx",
      "args": [
        "NuGet.Mcp.Server",
        "--prerelease",
        "--yes"
      ]
    }
  }
}</code></pre>
<p><strong>Note</strong>: The MCP server on NuGet.org is in preview, so you’ll need to use the &#8211;prerelease flag to install it. This ensures you automatically get the latest preview versions as they’re released.</p>
<h4>Using a Specific Version</h4>
<p>If you prefer to use a specific version of the MCP server instead of automatically updating to the latest preview, you can specify the version exact version:</p>
<pre><code class="language-json">{
  "servers": {
    "nuget": {
      "type": "stdio",
      "command": "dnx",
      "args": [
        "NuGet.Mcp.Server@0.2.0-preview",
        "--yes"
      ]
    }
  }
}</code></pre>
<h2>Current Capabilities</h2>
<p>The NuGet MCP server currently supports the following functionality:</p>
<ul>
<li><strong>Package Version Discovery</strong>: Determine the latest version of a package available on your configured feeds</li>
<li><strong>Security Updates</strong>: Update package vulnerabilities to the lowest compatible version that resolves security issues. This functionality will only update you to the lowest version that resolves the security issues, increasing the likelihood that the update will not break your project.</li>
<li><strong>Version Updates</strong>: Update package versions to the highest compatible version with your project&#8217;s target framework. Most tooling today that updates NuGet packages does not take into account a project&#8217;s target framework, which can causes updates to fail.</li>
</ul>
<h2>Integration with Development Tools</h2>
<h3>Visual Studio</h3>
<p>To add the NuGet MCP server to Visual Studio, add the following to your <code>.mcp.json</code> file next to a solution or at <code>%UserProfile%\.mcp.json</code>:</p>
<pre><code class="language-json">{
  "servers": {
    "nuget": {
      "type": "stdio",
      "command": "dnx",
      "args": [
        "NuGet.Mcp.Server",
        "--prerelease",
        "--yes"
      ]
    }
  }
}</code></pre>
<p>There are several places that Visual Studio looks for MCP server configurations, see <a href="https://learn.microsoft.com/visualstudio/ide/mcp-servers?view=vs-2022#file-locations-for-automatic-discovery-of-mcp-configuration">File locations for automatic discovery of MCP configuration</a> for more information.</p>
<p>When configured properly, the NuGet MCP server will automatically start and should look like this:</p>
<p><img alt="Visual Studio MCP server configuration" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/08/vs-mcp-configuration.png" /></p>
<h3>VS Code</h3>
<p>The easiest way to add MCP servers to VS Code is to click one of the links below for your particular installation:</p>
<p><a href="https://insiders.vscode.dev/redirect/mcp/install?name=nuget&amp;config=%7B%22type%22%3A%20%22stdio%22%2C%22command%22%3A%20%22dnx%22%2C%22args%22%3A%20%5B%22NuGet.Mcp.Server%22%2C%22--prerelease%22%2C%22--yes%22%5D%7D"><img alt="Install in VS Code" src="https://img.shields.io/badge/VS_Code-Install_NuGet_MCP_Server-0098FF?style=flat-square&amp;logo=visualstudiocode&amp;logoColor=white" /></a> <a href="https://insiders.vscode.dev/redirect/mcp/install?name=nuget&amp;config=%7B%22type%22%3A%20%22stdio%22%2C%22command%22%3A%20%22dnx%22%2C%22args%22%3A%20%5B%22NuGet.Mcp.Server%22%2C%22--prerelease%22%2C%22--yes%22%5D%7D&amp;quality=insiders"><img alt="Install in VS Code Insiders" src="https://img.shields.io/badge/VS_Code_Insiders-Install_NuGet_MCP_Server-24bfa5?style=flat-square&amp;logo=visualstudiocode&amp;logoColor=white" /></a></p>
<p>Altneratively, you can manually add the NuGet MCP server to your VS Code by adding the the following snippet to your <code>.vscode/mcp.json</code> file:</p>
<pre><code class="language-json">{
  "servers": {
    "nuget": {
      "type": "stdio",
      "command": "dnx",
      "args": [
        "NuGet.Mcp.Server",
        "--prerelease",
        "--yes"
      ]
    }
  }
}</code></pre>
<p>When configured properly, the NuGet MCP server will automatically start and should look like this:</p>
<p><img alt="VS Code MCP server configuration" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/08/vscode-mcp-configuration.png" /></p>
<h3>GitHub Coding Agent</h3>
<p>The <a href="https://docs.github.com/copilot/concepts/coding-agent/coding-agent">GitHub Coding Agent</a> can utilize the NuGet MCP server to assist with dependency management in your repositories, making it easier to maintain and update your projects&#8217; package references.
Since the NuGet MCP server requires .NET 10 SDK Preview 6 or greater, you&#8217;ll need to add the following file to your repository:</p>
<p><code>.github/workflows/copilot-setup-steps.yml</code></p>
<pre><code class="language-yaml">name: "Copilot Setup Steps"

on:
  workflow_dispatch:
  push:
    paths:
      - .github/workflows/copilot-setup-steps.yml
  pull_request:
    paths:
      - .github/workflows/copilot-setup-steps.yml

jobs:
  # The job MUST be called `copilot-setup-steps` or it will not be picked up by Copilot.
  copilot-setup-steps:
    runs-on: ubuntu-latest

    permissions:
      contents: read

    steps:
      - name: Install .NET 10.x
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: |
            10.x
          dotnet-quality: preview
</code></pre>
<p>See <a href="https://docs.github.com/copilot/how-tos/use-copilot-agents/coding-agent/customize-the-agent-environment">Customizing the development environment for Copilot coding agent</a> for more information.</p>
<p>To configure the NuGet MCP server, visit <code>Settings -&gt; Copilot -&gt; Coding agent</code> in your repository and add this to your &#8220;MCP Configuration&#8221;</p>
<pre><code class="language-json">{
  "mcpServers": {
    "nuget": {
      "command": "dnx",
      "args": [
        "NuGet.Mcp.Server",
        "--prerelease",
        "--yes"
      ],
      "tools": ["*"],
      "type": "local"
    }
  }
}</code></pre>
<p>The configuration page should look like this:</p>
<p><img alt="GitHub Copilot coding agent configuration" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/08/github-mcp-configuration.png" /></p>
<h2>Preview Status and Feedback</h2>
<p>Note: This is a preview release of the NuGet MCP server. We&#8217;re actively building new features and improvements, and we’d love your feedback on how to make it more valuable to you.
Your input will ensure the tool evolves to better meet the needs of the .NET development community.</p>
<p>Please file an issue on <a href="https://github.com/NuGet/Home/issues/new?template=MCPSERVER.yml">NuGet/Home</a> with your thoughts, suggestions, or any issues you encounter.</p>
<p>The post <a href="https://devblogs.microsoft.com/dotnet/nuget-mcp-server-preview/">Announcing the NuGet MCP Server Preview</a> appeared first on <a href="https://devblogs.microsoft.com/dotnet">.NET Blog</a>.</p>
