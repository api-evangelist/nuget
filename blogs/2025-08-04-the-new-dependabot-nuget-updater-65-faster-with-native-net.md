---
title: "The new Dependabot NuGet updater: 65% faster with native .NET"
url: "https://devblogs.microsoft.com/dotnet/the-new-dependabot-nuget-updater/"
date: "Mon, 04 Aug 2025 15:00:00 +0000"
author: "Jamie Magee, Brett Forsgren"
feed_url: "https://devblogs.microsoft.com/dotnet/category/nuget/feed/"
---
<p>If you&#8217;ve ever waited impatiently for Dependabot to update your .NET dependencies, or worse, watched it fail with cryptic errors, we have some great news. Over the past year, the Dependabot team has worked on a refactor of the NuGet updater, and the results are impressive.</p>
<h2>From hybrid to native</h2>
<p>The previous NuGet updater used a hybrid solution that relied heavily on manual XML parsing and string replacement operations written in Ruby. While this approach worked for basic scenarios, it struggled with the complexity and nuances of modern .NET projects. The new updater takes a completely different approach by using .NET&#8217;s native tooling directly.</p>
<p>Instead of trying to reverse-engineer what NuGet and MSBuild do, the new updater leverages actual .NET tooling:</p>
<ul>
<li><a href="https://learn.microsoft.com/nuget/reference/nuget-client-sdk">NuGet client libraries</a> for package operations</li>
<li><a href="https://learn.microsoft.com/visualstudio/msbuild/msbuild-api">MSBuild APIs</a> for project evaluation and dependency resolution  </li>
<li><a href="https://learn.microsoft.com/dotnet/core/tools/">.NET CLI</a> for restore operations</li>
</ul>
<p>This shift from manual XML manipulation to using the actual .NET toolchain means the updater now behaves exactly like the tools developers use every day.</p>
<h2>Performance and reliability improvements</h2>
<p>The improvements in the new updater are dramatic. The test suite that previously took 26 minutes now completes in just 9 minutes—a 65% reduction in runtime. But speed is only part of the story. The success rate for updates has jumped from 82% to 94%, meaning significantly fewer failed updates that require manual intervention.</p>
<p>These improvements work together to deliver a faster, more reliable experience. When Dependabot runs on your repository, it spends less time processing updates and succeeds more often—reducing both the wait time and the manual intervention needed to keep your dependencies current.</p>
<h2>Real dependency detection with MSBuild</h2>
<p>One of the most significant improvements is how the updater discovers and analyzes dependencies. Previously, the Ruby-based parser would attempt to parse project files as XML and guess what the final dependency graph would look like. This approach was fragile and missed complex scenarios.</p>
<p>The new updater uses MSBuild&#8217;s project evaluation engine to properly understand your project&#8217;s true dependency structure. This means it can now handle complex scenarios that previously caused problems.</p>
<p>For example, the old parser missed conditional package references like this:</p>
<pre><code class="language-xml">&lt;ItemGroup Condition="'$(TargetFramework)' == 'net8.0'"&gt;
  &lt;PackageReference Include="Microsoft.Extensions.Hosting" Version="8.0.0" /&gt;
&lt;/ItemGroup&gt;</code></pre>
<p>With the new MSBuild-based approach, the updater can handle</p>
<ul>
<li>Conditional package references based on target framework or build configuration</li>
<li><a href="https://learn.microsoft.com/visualstudio/msbuild/customize-by-directory"><code>Directory.Build.props</code> and <code>Directory.Build.targets</code></a> that modify dependencies</li>
<li>MSBuild variables and property evaluation throughout the project hierarchy</li>
<li>Complex package reference patterns that weren&#8217;t reliably detected before</li>
</ul>
<h2>Dependency resolution solving</h2>
<p>One of the most impressive features of the new updater is its sophisticated dependency resolution engine. Instead of updating packages in isolation, it now performs comprehensive conflict resolution. This includes two key capabilities:</p>
<h3>Transitive dependency updates</h3>
<p>When you have a vulnerable transitive dependency that can&#8217;t be directly updated, the updater will now automatically find the best way to resolve the vulnerability. Let&#8217;s look at a real scenario where your app depends on a package that has a vulnerable transitive dependency:</p>
<pre><code class="language-plaintext">YourApp
└── PackageA v1.0.0
    └── TransitivePackage v2.0.0 (CVE-2024-12345)</code></pre>
<p>The new updater follows a smart resolution strategy:</p>
<ol>
<li>
<p>First, it checks if <code>PackageA</code> has a newer version available that depends on a non-vulnerable version of <code>TransitivePackage</code>. If <code>PackageA</code> v2.0.0 depends on <code>TransitivePackage</code> v3.0.0 (which fixes the vulnerability), Dependabot will update <code>PackageA</code> to v2.0.0.</p>
</li>
<li>
<p>If no updated version of <code>PackageA</code> is available, Dependabot will add a direct dependency on a non-vulnerable version of <code>TransitivePackage</code> to your project. This leverages NuGet&#8217;s <a href="https://learn.microsoft.com/nuget/concepts/dependency-resolution#direct-dependency-wins">&#8216;direct dependency wins&#8217; rule</a>, where direct dependencies take precedence over transitive ones:</p>
</li>
</ol>
<pre><code class="language-xml">&lt;PackageReference Include="PackageA" Version="1.0.0" /&gt;
&lt;PackageReference Include="TransitivePackage" Version="3.0.0" /&gt;</code></pre>
<p>With this approach, even though <code>PackageA</code> v1.0.0 still references <code>TransitivePackage</code> v2.0.0, NuGet will use v3.0.0 because it&#8217;s a direct dependency of your project. This ensures your application uses the secure version without waiting for <code>PackageA</code> to be updated.</p>
<h3>Related package updates</h3>
<p>The updater also identifies and updates related packages to avoid version conflicts. If updating one package in a family (like <code>Microsoft.Extensions.*</code> packages) would create version mismatches with related packages, the updater automatically updates the entire family to compatible versions.</p>
<p>This intelligent conflict resolution dramatically reduces the number of failed updates and eliminates the manual work of resolving package conflicts.</p>
<h2>Honoring global.json</h2>
<p>The new updater now properly respects <a href="https://learn.microsoft.com/dotnet/core/tools/global-json"><code>global.json</code></a> files, a feature that was inconsistently supported in the previous version. If your project specifies a particular .NET SDK version, the updater will install the exact SDK version specified in your <code>global.json</code>. This ensures that the updater evaluates dependency updates using the same .NET SDK version that your development team and CI/CD pipelines use, eliminating a common source of inconsistencies.</p>
<p>This improvement complements Dependabot&#8217;s recently added capability to <a href="https://devblogs.microsoft.com/dotnet/using-dependabot-to-manage-dotnet-sdk-updates/">update .NET SDK versions in global.json files</a>. While the SDK updater keeps your .NET SDK version current with security patches and improvements, the NuGet updater respects whatever SDK version you&#8217;ve chosen—whether manually specified or automatically updated by Dependabot. This seamless integration means you get the best of both worlds: automated SDK updates when you want them, and consistent package dependency resolution that honors your SDK choices.</p>
<h2>Full Central Package Management support</h2>
<p><a href="https://learn.microsoft.com/nuget/consume-packages/central-package-management">Central Package Management (CPM)</a> has become increasingly popular in .NET projects for managing package versions across multiple projects. The previous updater had limited support for CPM scenarios, often requiring manual intervention.</p>
<p>The new updater provides comprehensive CPM support. It automatically detects <code>Directory.Packages.props</code> files, properly updates versions in centralized version files, supports package overrides in individual projects, and handles transitive dependencies managed through CPM. Whether you&#8217;re using CPM for version management, security vulnerability management, or both, the new updater handles these scenarios seamlessly.</p>
<h2>Support for all compliant NuGet feeds</h2>
<p>The previous updater struggled with private NuGet feeds, especially those with non-standard authentication or API implementations. The new updater uses NuGet&#8217;s official client libraries. This means it automatically supports all <a href="https://learn.microsoft.com/nuget/api/overview">NuGet v2 and v3 feeds</a>, including nuget.org, Azure Artifacts, and GitHub Packages. It also:</p>
<ul>
<li>Works with standard authentication mechanisms like API keys or personal access tokens</li>
<li>Handles feed-specific behaviors and quirks that the NuGet client manages</li>
<li>Supports <a href="https://learn.microsoft.com/nuget/consume-packages/package-source-mapping">package source mapping</a> configurations for enterprise scenarios</li>
</ul>
<p>If your .NET tools can access a feed, Dependabot can too.</p>
<h2>What this means for you</h2>
<p>If you&#8217;re using Dependabot for .NET projects, you should notice these improvements immediately. Faster updates mean dependency scans and update generation happen more quickly. More successful updates result in fewer failed updates that require manual intervention. Better accuracy ensures updates that properly respect your project&#8217;s configuration and constraints. And when updates do fail, you&#8217;ll get clearer errors with actionable error messages.</p>
<p>You don&#8217;t need to change anything in your <a href="https://docs.github.com/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file"><code>dependabot.yml</code></a> configuration—you automatically get these improvements for all .NET projects.</p>
<h2>Looking forward</h2>
<p>This rewrite represents more than just performance improvements—it&#8217;s a foundation for future enhancements. By building on .NET&#8217;s native tooling, the Dependabot team will be able to add support for new .NET features as they&#8217;re released, improve integration with .NET developer workflows, extend capabilities to handle more complex enterprise scenarios, and provide better diagnostics and debugging information.</p>
<p>The new architecture also makes it easier for the community to contribute improvements and fixes, as we rewrote the codebase in C# and leverage the same tools and libraries that .NET developers use every day. This means that developers can make contributions using familiar .NET development practices, making it easier for the community to help shape the future of Dependabot&#8217;s NuGet support.</p>
<h2>Try it out</h2>
<p>The new NuGet updater is already live and processing updates for .NET repositories across GitHub. If you haven&#8217;t enabled Dependabot for your .NET projects yet, now is a great time to start. Here&#8217;s a minimal configuration to get you started:</p>
<pre><code class="language-yaml">version: 2
updates:
  - package-ecosystem: "nuget"
    directory: "/"
    schedule:
      interval: "weekly"</code></pre>
<p>And if you&#8217;re already using Dependabot, you should already be seeing the improvements. Faster updates, fewer failures, and clearer error messages—all without changing a single line of configuration.</p>
<p>The rewrite demonstrates how modern dependency management should work: fast, accurate, and transparent. By leveraging the same tools that developers use every day, Dependabot can now provide an experience that feels native to the .NET ecosystem while delivering the automation and security benefits that make dependency management less of a chore.</p>
<p>The post <a href="https://devblogs.microsoft.com/dotnet/the-new-dependabot-nuget-updater/">The new Dependabot NuGet updater: 65% faster with native .NET</a> appeared first on <a href="https://devblogs.microsoft.com/dotnet">.NET Blog</a>.</p>
