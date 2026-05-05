---
title: "New Trusted Publishing enhances security on NuGet.org"
url: "https://devblogs.microsoft.com/dotnet/enhanced-security-is-here-with-the-new-trust-publishing-on-nuget-org/"
date: "Mon, 22 Sep 2025 15:40:00 +0000"
author: "Evgeny Tvorun, Sean"
feed_url: "https://devblogs.microsoft.com/dotnet/category/nuget/feed/"
---
<p>We’re excited to announce Trusted Publishing on nuget.org — a simpler, safer way to publish NuGet packages from GitHub Actions. Rather than relying on long‑lived API keys, your workflow can use a short‑lived GitHub OIDC token to request a temporary, single‑use NuGet API key. These keys expire quickly (≈ 1 hour), eliminating long‑lived secrets that need to be stored, rotated, or protected from leaks.</p>
<p><img alt="👉" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f449.png" style="height: 1em;" /> Read the docs at <a href="https://aka.ms/nuget/trusted-publishing">aka.ms/nuget/trusted-publishing</a></p>
<p><a href="https://www.nuget.org/account/trustedpublishing" rel="noopener" target="_blank"><img alt="Trusted Publishing screenshot from NuGet.org" class="alignnone" height="1206" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/09/trusted-publishing-short.png" width="2084" /></a></p>
<h2>Why Trusted Publishing? <img alt="🔒" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f512.png" style="height: 1em;" /></h2>
<ul>
<li>No long‑lived secrets — nothing sensitive stored in your repository or CI.</li>
<li>Short‑lived credentials — temporary API keys are issued just‑in‑time and typically last about <strong>1 hour</strong>.</li>
<li>One token → one key — each job&#8217;s OIDC token maps to a single temporary API key used for that publish.</li>
</ul>
<h2>Getting started <img alt="🚀" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f680.png" style="height: 1em;" /></h2>
<ol>
<li><strong>Open the Trusted Publishing page</strong>
Sign in to <strong>nuget.org</strong> → open your user menu (top right) → <strong>Trusted Publishing</strong> (next to <strong>API Keys</strong>).</li>
<li><strong>Create a policy</strong>
<ul>
<li><strong>Package owner:</strong> you or your organization</li>
<li><strong>Repository owner / repository:</strong> your GitHub org/user and repository name (for example <code>contoso-sdk</code>)</li>
<li><strong>Workflow file:</strong> the YAML file in <code>.github/workflows/</code> (for example <code>release.yml</code>)</li>
<li><em>(Optional)</em> <strong>Environment:</strong> if your workflow uses GitHub Actions environments</li>
</ul>
</li>
<li><strong>Wire up your GitHub Actions workflow</strong> using the minimal example below.</li>
</ol>
<h2>Minimal GitHub Actions example <img alt="⚙" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2699.png" style="height: 1em;" /></h2>
<p>This example includes only the steps that interact with nuget.org: enabling OIDC, exchanging the token for a temporary API key, and pushing the package.</p>
<pre><code class="language-yaml">permissions:
  id-token: write   # required for GitHub OIDC

jobs:
  build-and-publish:
    permissions:
      id-token: write  # enable GitHub OIDC token issuance for this job

    steps:
      # Build your artifacts/my-sdk.nupkg package here

      # Get a short-lived NuGet API key
      - name: NuGet login (OIDC → temp API key)
        uses: NuGet/login@v1
        id: login
        with:
          # Recommended: use a secret like ${{ secrets.NUGET_USER }} for your nuget.org username (profile name), NOT your email address
          user: contoso-bot

      # Push the package
        run: dotnet nuget push artifacts/my-sdk.nupkg --api-key ${{ steps.login.outputs.NUGET_API_KEY }} --source https://api.nuget.org/v3/index.json</code></pre>
<hr />
<h2>How it works <img alt="🔁" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f501.png" style="height: 1em;" /></h2>
<ol>
<li>GitHub issues an OIDC token to the job.</li>
<li>The NuGet login step sends that token to nuget.org.</li>
<li>nuget.org validates the token against your Trusted Publishing policy and returns a temporary API key.</li>
<li>Your workflow uses that key to publish. Request the key immediately before running <code>dotnet nuget push</code> — it expires quickly (≈ <strong>1 hour</strong>).</li>
</ol>
<h2>Policy ownership &amp; lifecycle <img alt="📜" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f4dc.png" style="height: 1em;" /></h2>
<ul>
<li><strong>Private repo bootstrap (7 days, re-activate anytime).</strong> New policies for private repositories start out as active for 7 days by default. After the first successful NuGet login (the exchange of a job&#8217;s OIDC token for a temporary API key), the policy becomes permanently active and is bound to immutable GitHub IDs. If you miss the initial 7‑day window, you can manually re‑activate the policy for another 7 days from the Trusted Publishing page. A successful NuGet login is sufficient — you don’t need to publish a package.</li>
<li><strong>Owner matters.</strong> A policy is owned by a user or organization and applies only to packages owned by that owner.</li>
<li><strong>Org changes are respected.</strong> If the policy creator loses org membership, or the org is locked or deleted, the policy is disabled and displays a clear warning. When membership or org access is restored, the policy re‑activates automatically.</li>
</ul>
<h2>Migrating from long‑lived API keys <img alt="🔄" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f504.png" style="height: 1em;" /></h2>
<p>Already publishing from GitHub Actions? Switching is easy:</p>
<ol>
<li>Create a Trusted Publishing policy on nuget.org.</li>
<li>Remove stored NuGet API keys from your repo or CI secrets.</li>
<li>Add <code>NuGet/login@v1</code> to your workflow and use its output key with <code>dotnet nuget push</code>.</li>
<li>Done — enjoy, no more key management!</li>
</ol>
<h2>Try it today <img alt="🚀" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/1f680.png" style="height: 1em;" /></h2>
<ul>
<li>Read the docs at <a href="https://aka.ms/nuget/trusted-publishing">aka.ms/nuget/trusted-publishing</a></li>
<li>Sign in to <strong>nuget.org → Trusted Publishing</strong> (next to <strong>API Keys</strong>) and create your first policy.</li>
</ul>
<p><span>Huge thanks to <a class="fui-Link ___1q1shib f2hkw1w f3rmtva f1ewtqcl fyind8e f1k6fduh f1w7gpdv fk6fouc fjoy568 figsok6 f1s184ao f1mk8lai fnbmjn9 f1o700av f13mvf36 f1cmlufx f9n3di6 f1ids18y f1tx3yz7 f1deo86v f1eh06m1 f1iescvh fhgqx19 f1olyrje f1p93eir f1nev41a f1h8hb77 f1lqvz6u f10aw75t fsle3fq f17ae5zn" href="https://openssf.org/" id="menur55l" rel="noreferrer noopener" target="_blank" title="https://openssf.org/">OpenSSF</a> and the <a class="fui-Link ___1q1shib f2hkw1w f3rmtva f1ewtqcl fyind8e f1k6fduh f1w7gpdv fk6fouc fjoy568 figsok6 f1s184ao f1mk8lai fnbmjn9 f1o700av f13mvf36 f1cmlufx f9n3di6 f1ids18y f1tx3yz7 f1deo86v f1eh06m1 f1iescvh fhgqx19 f1olyrje f1p93eir f1nev41a f1h8hb77 f1lqvz6u f10aw75t fsle3fq f17ae5zn" href="https://openssf.org/technical-initiatives/repository-security/" id="menur55n" rel="noreferrer noopener" target="_blank" title="https://openssf.org/technical-initiatives/repository-security/">Securing Software Repos working group</a> for defining the Trusted Publishing guidelines and encouraging their adoption throughout the broader ecosystem.</span></p>
<p>Publish more securely and with less friction — thank you for contributing to the NuGet community. <img alt="✨" class="wp-smiley" src="https://s.w.org/images/core/emoji/17.0.2/72x72/2728.png" style="height: 1em;" /></p>
<p>The post <a href="https://devblogs.microsoft.com/dotnet/enhanced-security-is-here-with-the-new-trust-publishing-on-nuget-org/">New Trusted Publishing enhances security on NuGet.org</a> appeared first on <a href="https://devblogs.microsoft.com/dotnet">.NET Blog</a>.</p>
