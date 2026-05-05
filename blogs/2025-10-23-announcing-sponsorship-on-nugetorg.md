---
title: "Announcing Sponsorship on NuGet.org"
url: "https://devblogs.microsoft.com/dotnet/announcing-sponsorship-on-nugetdotorg-for-maintainer-appreciation/"
date: "Thu, 23 Oct 2025 22:18:00 +0000"
author: "Sean"
feed_url: "https://devblogs.microsoft.com/dotnet/category/nuget/feed/"
---
<p>Package maintainers are the backbone of the NuGet.org ecosystem, building and maintaining the packages we all rely on. Today we are excited to announce the new <strong>NuGet.org Sponsorship feature</strong> which makes it easier than ever for consumers to recognize and support the authors behind their favorite packages. With Sponsorship on NuGet.org:</p>
<ul>
<li><img alt="🔍" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f50d.png" style="height: 1em;" /> Maintainers get visibility for their funding needs</li>
<li><img alt="🤝" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f91d.png" style="height: 1em;" /> Consumers can give back easily</li>
<li><img alt="🌱" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f331.png" style="height: 1em;" /> The community grows stronger together</li>
</ul>
<p>This feature is a step toward a more sustainable and resilient .NET ecosystem.</p>
<h2><img alt="🚀" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f680.png" style="height: 1em;" /> What’s New?</h2>
<p><img alt="👩‍💻" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f469-200d-1f4bb.png" style="height: 1em;" /> For Package Authors: NuGet.org now allows package authors to add a <strong>Sponsorship URL</strong> to their packages. This link appears as a <img alt="❤" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2764.png" style="height: 1em;" /> icon or “Sponsor this package” button on the package’s page, guiding users to secure and popular platforms like GitHub Sponsors, Patreon, and Open Collective. Adding a sponsorship link is simple following the steps outlined in the next section.</p>
<p><img alt="👨‍🔧" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f468-200d-1f527.png" style="height: 1em;" /> For Package Consumers: Supporting your favorite packages is now easier than ever. Look for the sponsor icon <img alt="❤" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2764.png" style="height: 1em;" /> on NuGet.org. See which packages need support while browsing or managing dependencies. Click to visit the maintainer’s sponsorship page and contribute. Even small contributions can make a big difference in keeping critical packages maintained and secure.</p>
<h2>For Package Publishers: Setting Up Sponsorship</h2>
<h3>Prerequisites</h3>
<ul>
<li>You must be the owner or co-owner of a package on NuGet.org</li>
<li>Your sponsorship link platform must be from the approved list:
<ul>
<li>GitHub Sponsors</li>
<li>Patreon</li>
<li>Open Collective</li>
<li>Ko-fi</li>
<li>Tidelift</li>
<li>Liberapay</li>
</ul>
</li>
</ul>
<h3>Step 1: Navigate to Your Package Management Page</h3>
<ol>
<li>Go to <a href="https://nuget.org">NuGet.org</a> and sign in to your account</li>
<li>Click on your username in the top right corner</li>
<li>Select &#8220;Manage Packages&#8221; from the dropdown menu</li>
<li>Find and click on the package you want to add sponsorship information to</li>
</ol>
<h3>Step 2: Access Sponsorship Settings</h3>
<ol>
<li>On your package management page, scroll down to find the &#8220;Sponsorship Links&#8221; section</li>
<li>Click to expand the collapsible &#8220;Sponsorship Links&#8221; section</li>
<li>You&#8217;ll see a form where you can add sponsorship URLs<img alt="NuGet.org Manage Package Page with Sponsorship Links Section" class="alignnone" height="764" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/10/manage_sponsorship_links.png" width="895" /><em>Screenshot of the manage package page with the &#8216;Sponsorship Links&#8217; section</em></li>
</ol>
<h3>Step 3: Add Your Sponsorship URLs</h3>
<ol>
<li>Enter your sponsorship URL in the text field
<ul>
<li>Example: <code>https://github.com/sponsors/yourusername</code></li>
<li>Example: <code>https://www.patreon.com/yourusername</code></li>
</ul>
<p><img alt="NuGet.org Manage Package Page with an example sponsorship link filled in" class="alignnone" height="377" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/10/add_sponsorship_link.png" width="905" /></p>
<p><em>Screenshot of the sponsorship URL form with example URL filled in</em></li>
<li>Click the &#8220;Add&#8221; button</li>
<li>The system will automatically validate that your URL is from an approved platform</li>
<li>If any URLs are invalid or from non-approved platforms, you&#8217;ll see error messages to correct them. Otherwise, you&#8217;ll see a confirmation message that your sponsorship link has been saved<img alt="NuGet.org Manage Package Page with an example sponsorship link that is not approved" class="alignnone" height="456" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/10/sponsorship_link_error.png" width="909" /><em>Screenshot of a URL from a platform that is not approved</em></li>
<li>Each added sponsorship URL will have a &#8220;Remove&#8221; button next to it if you need to delete it</li>
<li>To add multiple sponsorship platforms, click &#8220;Add&#8221; button again</li>
<li>You can add up to 10 different sponsorship URLs per package ID</li>
</ol>
<h3>Step 5: Verify Your Sponsorship Display</h3>
<ol>
<li>Navigate to your package&#8217;s public page on NuGet.org</li>
<li>Look for the &#8220;Sponsor&#8221; button in the package details &#8220;About&#8221; section</li>
<li>Click the &#8220;Sponsor&#8221; button to test that your URLs appear correctly in the popup<img alt="NuGet.org Package Details page with a Sponsor button" class="alignnone" height="470" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/10/sponsor_button.png" width="917" />
Screenshot of a package details page with a Sponsor button</li>
</ol>
<h2>For Users: Finding and Supporting Packages</h2>
<h3>Step 1: Identify Packages That Need Sponsorship</h3>
<ol>
<li>Browse to any package page on NuGet.org</li>
<li>Look for packages displaying a &#8220;Sponsor&#8221; button in the package details section</li>
<li>The &#8220;Sponsor&#8221; button indicates that the package maintainer is seeking financial support<img alt="NuGet.org Package Details page with a Sponsor button" class="alignnone" height="478" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/10/sponsor_button.png" width="932" /><em>Screenshot of a package details page with a Sponsor button</em></li>
</ol>
<h3>Step 2: View Available Sponsorship Options</h3>
<ol>
<li>Click the &#8220;Sponsor&#8221; button on the package page</li>
<li>A popup window will appear showing all available sponsorship links for that package<img alt="NuGet.org Package Details page with sponsorship links popup" class="alignnone" height="535" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/10/sponsorship_links_display.png" width="939" /><em>Screenshot of a package details page with the sponsorship links popup open</em></li>
</ol>
<h3>Step 3: Choose Your Preferred Sponsorship Platform</h3>
<ol>
<li>Review the available sponsorship options in the popup</li>
<li>Click on your preferred platform to be redirected to the external sponsorship page</li>
<li>The link will open in a new tab/window, keeping the NuGet package page open</li>
</ol>
<p><img alt="⚠" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/26a0.png" style="height: 1em;" /> <div class="alert alert-info"><p class="alert-divider"><i class="fabric-icon fabric-icon--Info"></i><strong>Important Notice</strong></p>These links will take you to third-party platforms. Microsoft is not affiliated with or responsible for the content or practices of third-party platforms.</div></p>
<h2>Frequently Asked Questions</h2>
<p><strong>Q: Can I add sponsorship information to older versions of my package?</strong></p>
<p>A: Yes! Sponsorship information is managed at the package ID level, so it automatically applies to all versions of your package, including previously published versions.</p>
<p><strong>Q: What happens if my sponsorship platform URL changes?</strong></p>
<p>A: You can update your sponsorship URLs anytime through the package management page. Changes take effect immediately across all versions.</p>
<p><strong>Q: Can I see analytics on how many people clicked my sponsorship links?</strong></p>
<p>A: No, NuGet.org doesn&#8217;t track sponsorship link clicks. You&#8217;ll need to check analytics on your sponsorship platform directly.</p>
<p><strong>Q: Can I add custom sponsorship platforms not on the approved list?</strong></p>
<p>A: Currently, only the approved list of platforms is supported. This helps ensure security and legitimacy of sponsorship links.</p>
<p><strong>Q: Does NuGet.org store my financial information?</strong></p>
<p>A: No personal or financial data is stored by NuGet.org. All transactions occur on secure external platforms that a maintainer chooses for sponsoring their packages.</p>
<h2><img alt="✅" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2705.png" style="height: 1em;" /> Get Started Today</h2>
<p><img alt="🎯" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f3af.png" style="height: 1em;" /> Whether you&#8217;re a package author or a consumer:</p>
<ul>
<li><img alt="📦" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f4e6.png" style="height: 1em;" /> Authors: Add your sponsorship link on NuGet.org now</li>
<li><img alt="👥" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f465.png" style="height: 1em;" /> Consumers: Look for the <img alt="❤" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2764.png" style="height: 1em;" /> and support the packages you love</li>
</ul>
<p>Let’s contribute to a more sustainable NuGet.org ecosystem — one sponsorship at a time!</p>
<p>The post <a href="https://devblogs.microsoft.com/dotnet/announcing-sponsorship-on-nugetdotorg-for-maintainer-appreciation/">Announcing Sponsorship on NuGet.org</a> appeared first on <a href="https://devblogs.microsoft.com/dotnet">.NET Blog</a>.</p>
