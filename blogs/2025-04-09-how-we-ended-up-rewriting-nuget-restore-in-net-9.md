---
title: "How we ended up rewriting NuGet Restore in .NET 9"
url: "https://devblogs.microsoft.com/dotnet/rewriting-nuget-restore-in-dotnet-9/"
date: "Wed, 09 Apr 2025 17:05:00 +0000"
author: "The NuGet Team"
feed_url: "https://devblogs.microsoft.com/dotnet/category/nuget/feed/"
---
<p>This is the story of how team members across NuGet, Visual Studio, and .NET embarked on a journey to fully rewrite the NuGet Restore algorithm to achieve break-through scale and performance. Written from the perspective of several team members, this entry provides a deep dive into the internals of NuGet, as well as strategies to identify and address performance issues. We hope that you enjoy it!</p>
<h2>Foreword</h2>
<blockquote>
<p>By Arturo Ortiz, Principal Engineering Manager for the NuGet Client team</p>
</blockquote>
<p>When an internal team at Microsoft—let&#8217;s call them TeamX—reached out to us because their NuGet Restore times had drastically increased to over 30 minutes, I knew we were about to begin a difficult journey. The core algorithm of NuGet Restore hadn’t been touched in over ten years, and few engineers understood all the rules and special cases for generating a complete and accurate set of dependencies. A single regression in functionality could break builds for millions of .NET developers around the world, or worse, result in obscure runtime failures in critical applications.</p>
<p>Thankfully, at Microsoft, we have some of the brightest software engineers in the industry. Perhaps even more importantly, we foster an engineering culture that equally values both helping others and accepting help from others. This culture allowed us to quickly engage and collaborate with performance experts across the organization to optimize the existing implementation, without needing to work through layers of management or planning processes. This first push resulted in NuGet Restore times being cut in half, which was a reasonable stopping point for our work. However, along the way, we realized that a more extensive rewrite could improve performance by a factor of 5x or more.</p>
<p>We were extremely fortunate that we were then able to get .NET architects like Scott Wadsworth and Brian Robbins involved. Together, we started what would become a six-month project to rewrite the core NuGet Restore algorithm from the ground up, something that we wouldn’t have even considered a year earlier due to the aforementioned risks.</p>
<p>The end results were impressive. For TeamX, the performance of NuGet Restore was improved by a factor of 16x (~32 mins to ~2 mins), saving thousands of compute hours per day for their Continuous Integration/Continuous Deployment pipeline and significantly improving their developer productivity. In addition, they were able to continue their journey to migrate all their services to a modern .NET Core architecture without worrying about exponentially increasing NuGet Restore (and, consequently, Build and Deployment) times. Furthermore, scalability testing with synthetic solutions shows that NuGet Restore can now easily scale to tens of thousands of projects with an almost linear pattern.</p>
<p>Below, you will read from the engineers themselves not only about the final solution, but also about their individual journeys—from applying well-known performance and memory allocation patterns to implementing early prototypes and breaking down an intricate algorithm into smaller, tractable problems.</p>
<p>I’d like to thank everyone who contributed with expertise, fixes, prototyping, and other efforts; also, the peers and managers who supported and encouraged us from start to end. Thank you!</p>
<h2>NuGet in .NET</h2>
<blockquote>
<p>By Jeff Kluge and Nikolche Kolev, Principal Software Engineers in NuGet Client</p>
</blockquote>
<p>Before we step into the rewrite, let&#8217;s take a step back in time and look at how we got here.</p>
<h3>History</h3>
<p>When NuGet was first developed in 2011, .NET Framework projects targeted a single framework, and NuGet only needed to resolve the dependency graph when a package was installed. For example, when installing a package using the NuGet Package Manager in Visual Studio, NuGet would resolve the dependency graph upon clicking Install and write a flattened representation of it to a file named <code>packages.config</code>. Later on, as part of a restore operation, NuGet would simply read the <code>packages.config</code> file and download all the packages in it, never recalculating the graph again unless a package was added or removed. This also meant that users could manually edit the <code>packages.config</code> file, bypassing NuGet&#8217;s dependency resolver logic which could lead to issues later on. <code>Packages.config</code> was not transitive, so each project would need its own list of packages, and users would need to install packages to every project as needed. As the number of projects in a Visual Studio solution grew, maintaining the dependencies at scale became very difficult.</p>
<p>Here is a sample <code>packages.config</code> for a typical ASP.NET web application project:</p>
<pre><code class="language-xml">&lt;?xml version="1.0" encoding="utf-8"?&gt;
&lt;packages&gt;
  &lt;package id="Antlr" version="3.5.0.2" /&gt;
  &lt;package id="bootstrap" version="5.2.3" /&gt;
  &lt;package id="jQuery" version="3.7.0" /&gt;
  &lt;package id="log4net" version="3.0.0" /&gt;
  &lt;package id="Microsoft.AspNet.FriendlyUrls" version="1.0.2" /&gt;
  &lt;package id="Microsoft.AspNet.FriendlyUrls.Core" version="1.0.2" /&gt;
  &lt;package id="Microsoft.AspNet.ScriptManager.MSAjax" version="5.0.0" /&gt;
  &lt;package id="Microsoft.AspNet.ScriptManager.WebForms" version="5.0.0" /&gt;
  &lt;package id="Microsoft.AspNet.Web.Optimization" version="1.1.3" /&gt;
  &lt;package id="Microsoft.AspNet.Web.Optimization.WebForms" version="1.1.3" /&gt;
  &lt;package id="Microsoft.CodeDom.Providers.DotNetCompilerPlatform" version="2.0.1" /&gt;
  &lt;package id="Microsoft.Web.Infrastructure" version="2.0.0" /&gt;
  &lt;package id="Modernizr" version="2.8.3" /&gt;
  &lt;package id="Newtonsoft.Json" version="13.0.3" /&gt;
  &lt;package id="WebGrease" version="1.6.0" /&gt;
&lt;/packages&gt;</code></pre>
<p>In the early days of NuGet, there were less than 500 packages available, and the levels of dependencies were not very deep. As the complexity of packages grew over time, so did the complexity of the .NET SDK. As we developed a newer version of NuGet for .NET Core, the ability to target multiple .NET frameworks was added. This was done so that library authors could build a .NET Core assembly while also maintaining their .NET Framework implementation, which helped with the transition to .NET Core. To reduce toil stemming from the increasing complexity of the ecosystem, it was also decided to not require users to specify the full transitive dependency graph for each project. Instead, only direct package references would be needed. Also, as part of a restore operation, NuGet would evaluate and resolve the dependency graph every time. This would fix the problem of users adding or removing packages outside of the Package Manager and eliminate the need for a <code>packages.config</code> file next to every single project. At that time, the NuGet dependency graph resolution algorithm was re-written in order to support all of the required functionality. Projects were able to target multiple frameworks, and NuGet Restore would resolve the transitive package graph on-the-fly. The transitivity was extended to projects, so any project referenced by another would automatically flow its dependencies to its parent projects.</p>
<p>In this example, Project 1 references Project 2, so Project 2’s dependencies, Package B, Package C, and Package D are part of Project 1’s resolved graph.</p>
<p><img alt="Mermaid diagram 1" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid1.png" /></p>
<p>During restore, NuGet now needed to not only walk the transitive dependency graph based on declared references, but also merge in the subgraphs of each child project and its dependencies.</p>
<p>Here is a sample of a project that uses PackageReference items where the transitive dependencies are not declared because NuGet determines them at restore time:</p>
<pre><code class="language-xml">&lt;Project Sdk="Microsoft.NET.Web.Sdk"&gt;
  &lt;ProperyGroup&gt;
    &lt;TargetFramework&gt;net472&lt;/TargetFramework&gt;
  &lt;/ProperyGroup&gt;
  &lt;ItemGroup&gt;
    &lt;PackageReference Include="bootstrap" Version="5.2.3" /&gt;
    &lt;PackageReference Include="jQuery" Version="3.7.0" /&gt;
    &lt;PackageReference Include="Microsoft.AspNet.FriendlyUrls" Version="1.0.2" /&gt;
    &lt;PackageReference Include="Microsoft.AspNet.ScriptManager.MSAjax" Version="5.0.0" /&gt;
    &lt;PackageReference Include="Microsoft.AspNet.ScriptManager.WebForms" Version="5.0.0" /&gt;
    &lt;PackageReference Include="Microsoft.AspNet.Web.Optimization.WebForms" Version="1.1.3" /&gt;
    &lt;PackageReference Include="Modernizr" Version="2.8.3" /&gt;
  &lt;/ItemGroup&gt;
&lt;/Project&gt;</code></pre>
<h3>Why is resolving a graph difficult?</h3>
<p>The concept of a dependency graph is fundamental to computers and is widely used in software of all kinds. Consider an automation system which runs steps in a defined order. A user declares the steps to execute along with the dependency order in which they must run. In this example, Task 2 depends on Task 3, Task 1 depends on Task 2 and Job 1 is considered complete when all tasks are done.</p>
<p><img alt="Mermaid Diagram 2" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid2.png" /></p>
<p>In this example, the software needs to build that graph and execute the tasks in the correct order:</p>
<p><img alt="Mermaid Diagram 3" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid3.png" /></p>
<p>This sort of graph is fairly simple where each node declares what it depends on and no node can appear more than once in the graph.</p>
<p>The primary complication with NuGet package graphs are that the nodes are versioned. This means that each node declares a dependency and a range of versions that are compatible with it, like &gt;= 1.0.0.</p>
<p>Consider these two packages and their transitive graphs:</p>
<p><img alt="Mermaid Diagram 4" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid4.png" /></p>
<p>If a project references Package A 1.0.0 and Package B 2.0.0, the graph would look something like this:</p>
<p><img alt="Mermaid Diagram 5" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid5.png" /></p>
<p>However, now there are two versions of Package B and Package C in the graph which is not allowed. In this case, NuGet must unify the graph by eclipsing lower versions of packages in favor of higher ones since this graph has declared that it needs Package B &gt;=2.0.0. This would be the resolved graph where Package B 1.0.0 and Package C 1.0.0 do not end up in the list:</p>
<p><img alt="Mermaid Diagram 6" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid6.png" /></p>
<h3>Limitations of the NuGet implementation</h3>
<p>The .NET Core implementation of NuGet’s dependency graph resolution algorithm was recursive, where each node would call the same method on each child. The algorithm would resolve dependencies with a depth first approach, meaning that it would start at the bottom and resolve dependencies as it walked up the parent tree. All this resulted in a lot of queuing of tasks which needed to wait on other tasks, as well as lots of memory allocations and lookups since all of the parents are waiting on children. Using recursion in software is a valid pattern, but, in this case, it was causing large, complex graphs to take longer to resolve. Additionally, the recursive calls made it very difficult to debug the algorithm.</p>
<pre><code class="language-c#">public static Node&lt;T&gt; CreateNode&lt;T&gt;(T item)
    where T: class
{
    Node&lt;T&gt; node = new Node&lt;T&gt;(item);

    foreach (T child in node.GetDependencies())
    {
        Node&lt;T&gt; childNode = CreateNode(child);

        node.Children.Add(childNode);
    }

    return node;
}</code></pre>
<p>Another major problem with this algorithm is that it created a full representation of the graph, including all nodes and edges. If a particular dependency was found in the graph multiple times, NuGet added a node for it every time, resulting in duplicated nodes. In the repository from TeamX, one project was creating a graph with 1.6 million nodes with thousands of duplicates.</p>
<p>Once the graph was created in full, NuGet iterated over each dependency at each level, resolving the graph for that subset of dependencies. In large graphs, there can be hundreds of levels of dependencies in the tree. NuGet walked each level until it got to the bottom and kept track of whether or not it came across any work to do. NuGet would repeat this process until it got through the graph with no work left to do. This was necessary because, as individual sections of the dependency graph were resolved, the outcome of the entire graph could be impacted. So, as these individual resolutions happened, the algorithm needed to re-walk the graph to determine if those changes affected other parts of it.</p>
<p>We also noticed that, as the graph was being constructed, each node would have its validity calculated by walking up its parent tree, looking for cycles and downgrades. This essentially would pause graph construction for each node while its validity was checked. Duplicated nodes and graphs with millions of items would make this process very long.</p>
<p>For the project from TeamX mentioned, NuGet iterated a total of nine times through all 1.6 million nodes! To make things worse, this project targeted two frameworks, .NET Framework and .NET Core, each requiring a completely separate calculation of the dependency graph. Keep in mind that this was only one project out of approximately 2,500 projects taking part in a NuGet Restore.</p>
<p>This graph shows how the algorithm would represent duplicates in the graph over and over, leading to memory pressure issues for large NuGet Restores. In this case, we have a total of 72 nodes rather than 16 tracking each unique dependency in the graph. This is because &#8220;O&#8221; is a common package with its own subgraph, and it is referenced by a lot of other dependencies.</p>
<p><img alt="Mermaid Diagram 7" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid7.png" /></p>
<p>It would be much better to not create duplicate nodes in the graph and instead create the appropriate links so that the graph is still resolved correctly. This dependency graph is a better representation with only 16 nodes:</p>
<p><img alt="Mermaid Diagram 8" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid8.png" /></p>
<p>The design of this algorithm made sense for the scale at the time (and arguably for the scale today, where 99.99% of repositories have less than 1,000 projects). It was written with the idea of creating a full representation of the graph, then walking through the levels of the graph, resolving nodes on-the-go. For most projects, the performance of this algorithm was very good. Most NuGet Restores completed in under a few seconds so there wasn&#8217;t a real need to improve the algorithm. However, as more and more large repositories adopted NuGet, the pressure was building to come up with a better implementation.</p>
<h2>First Step: Performance Optimizations</h2>
<blockquote>
<p>By Eric Arndt, Senior Software Engineer in Visual Studio</p>
</blockquote>
<p>The increasing utilization of NuGet began noticeably impacting the solution load experience in Visual Studio. There is a lot of work that has to happen to enable the multitude of features that developers crave and nobody likes to wait. As a member of the team that owns the solution experience, I started looking at options we had to make this experience better, and one of those options was reducing the time and resources used to perform a NuGet Restore. This code is core to the modern .NET ecosystem, so the desire was to address these performance issues while minimizing risk.</p>
<p>Addressing performance issues is hard, and one of the hardest parts of addressing a performance issue is figuring out where to get started. Casually perusing the code often doesn’t lead to any actionable changes, and even if you find something that looks promising, it may not even matter for the scenarios you care about. On top of that, there can be multiple separate issues that need to be addressed such as CPU time, disk access, network access, thread contention, or garbage collection. Understanding the nature of the problem is crucial to addressing it.</p>
<p>So how do we make meaningful progress? The best thing to do is to collect data during our scenario and use that data to decide where to focus. Luckily, there are several helpful tools that are available for this purpose. I typically use <a href="https://github.com/microsoft/perfview">PerfView</a>, and while it’s not the most approachable tool, it offers an incredible amount of power and flexibility once you get used to it. Alternatively, the performance profiler in Visual Studio is much more approachable and shouldn’t be overlooked if you want an easy way to start understanding how your app performs. Several other apps are available with different tradeoffs, so it’s about finding one that works best for you. The most important part is collecting data, and once you have this data, you can begin narrowing in on the problem.</p>
<p>So, what was causing NuGet Restores to take so long? Allocations! Real world traces showed that in some larger restores, hundreds of gigabytes were being allocated, and as a result, nearly half of the restore time was spent performing hundreds of separate garbage collections. During a garbage collection, all managed threads are paused until the collection completes, and the code you actually care about sits idle during this time. The most expensive path during a restore operation was the construction of the package dependency graph, and as solutions continue to grow, this problem was getting exponentially worse.</p>
<p>Once you’ve narrowed in on the problematic area, keep in mind that there are important tradeoffs to consider. Making something faster can lead to less maintainable code, so it’s important to ensure that the tradeoff is worth it. Also, it&#8217;s not always obvious where allocations crop up, so it’s essential to use data from traces to guide you. Consider this common pattern:</p>
<pre><code class="language-c#">static void Main(string[] args)
{
    List&lt;int&gt; numbers = new() { 1, 2, 3 };
    WriteNumbers(numbers);
}

static void WriteNumbers(IEnumerable&lt;int&gt; numbers)
{
    foreach (int number in numbers)
    {
        // do work
    }
}</code></pre>
<p>Each call to <code>WriteNumbers()</code> causes an allocation by boxing the underlying struct enumerator implemented by <code>List&lt;T&gt;</code>. Is this a problem? It depends! If this is only called a handful of times, changing it will have essentially no impact. However, in very hot paths, this can result in substantial amounts of allocations that harm performance. This PR <a href="https://github.com/NuGet/NuGet.Client/pull/5588">Avoid boxing HashSet Enumerator in IsBestVerion by Erarndt · Pull Request #5588 · NuGet/NuGet.Client</a> addresses this exact issue and can save multiple gigabytes of allocations.</p>
<p>When talking about performance in .NET, LINQ inevitably ends up being discussed. Proper use of LINQ can lead to well performing and maintainable code since it allows you to stream data through an arbitrary number of transformations while lazily evaluating the results. The issues often arise when these statements are frequently called or when leaky abstractions lead to problems.  A handful of these issues are addressed in the change <a href="https://github.com/NuGet/NuGet.Client/pull/5592">Avoid multiple enumeration when writing Json objects by Erarndt · Pull Request #5592 · NuGet/NuGet.Client</a>. Let’s take a deeper look at a specific change in this PR:</p>
<pre><code class="language-c#">private static void SetArrayValue(IObjectWriter writer, string name, IEnumerable&lt;string&gt; values)
{
    if (values != null &amp;&amp; values.Any())
    {
        writer.WriteNameArray(name, values);
    }
}</code></pre>
<p>This function tries to avoid doing unnecessary work if by skipping the call to <code>WriteNameArray()</code> if there’s nothing to write.  This looks rather innocent, and in most cases, this approach is just fine. However, think about what happens if callers look like this:</p>
<pre><code class="language-c#">SetArrayValue(writer, "noWarn", dependency
    .NoWarn
    .OrderBy(c =&gt; c)
    .Distinct()
    .Select(code =&gt; code.GetName())
    .Where(s =&gt; !string.IsNullOrEmpty(s)));</code></pre>
<p>Just evaluating the <code>Any()</code> call now requires multiple steps:</p>
<ol>
<li>Tracking and returning only distinct items.</li>
<li>Processing all distinct items to put them in order.</li>
<li>Evaluating the <code>Select()</code> and <code>Where()</code> statements until the first matching element is found.</li>
</ol>
<p>All of this is done just to figure out if we have any work to do, and if we do, we must do this all over again in the body of <code>WriteNameArray()</code> where the actual work happens. Each time we evaluate this chain of calls there are allocations and potentially non-trivial CPU work happening here, and in frequently called paths, this can really add up. We could try to avoid LINQ altogether here, but since <code>SetArrayValue()</code> has many callers, that’s likely to lead to code that’s harder to maintain. Instead, I fixed this by only evaluating the LINQ chain once. To do that, I had to remove the call to <code>Any()</code> and process the enumerator directly inside of <code>WriteNameArray()</code>.</p>
<pre><code class="language-c#">var enumerator = values.GetEnumerator();
if (!enumerator.MoveNext())
{
    return;
}

_writer.WritePropertyName(name);
_writer.WriteStartArray();
_writer.WriteValue(enumerator.Current);
while (enumerator.MoveNext())
{
    _writer.WriteValue(enumerator.Current);
}

_writer.WriteEndArray();</code></pre>
<p>So far, the examples have been surgical fixes that are rather narrow in scope, but sometimes more extensive changes can be required. One such situation was with the change <a href="https://github.com/NuGet/NuGet.Client/pull/5624">Switch CreateNodeAsync to an iterative approach by Erarndt · Pull Request #5624 · NuGet/NuGet.Client</a>. Since <code>CreateGraphNodeAsync</code> uses async/await, a compiler generated state machine object is transparently created to track the execution state of the method as it suspends and resumes. Each call to <code>CreateGraphNodeAsync</code> results in the creation of a new state machine object, and since it is recursive, this can happen quite often during larger restores.</p>
<p>To avoid this, we decided to switch to an iterative approach so that only one state machine object is allocated. For tail recursive methods, this is trivial to do. However, <code>CreateGraphNodeAsync</code> makes multiple recursive calls in a loop at each level and does additional work once all the recursive calls at that level are finished.</p>
<p>This method is at the heart of all restore operations, so we wanted to minimize risk by emulating what happens with the call stack during the recursive calls. I approached this by having a <code>Stack&lt;GraphNodeStackState&gt;</code> that would keep track of all the required state. There were multiple helper methods that modified state and made recursive calls, so tracking the flow of execution was the first hurdle in switching. The first thing I did was inline some of these helpers and restructure the code to make it easier to follow. This let me separate state that stays the same from state that is modified and has to be tracked in the emulated stack. With that accomplished, all that was left was pushing state onto the stack for each “recursive” call, popping from the stack to evaluate the next “frame”, and looping until the stack is empty.</p>
<p>It’s important to note that no single change resulted in significant improvements, and we only started seeing a measurable difference when several improvements were batched together. In addition to reducing allocations and CPU costs, another good optimization strategy that was employed was to refine the existing parallelism to better utilize the CPU and reduce wall clock execution time. With all the improvements implemented, we brought NuGet Restore for TeamX down from 32 to approximately 16 minutes.</p>
<p>Unfortunately, the exponential nature of the algorithm meant that adding a handful of new projects and relationships would push this number back up to near 32 minutes again. Small tweaks and optimizations are perfect when the core of the system has the right runtime complexity, but if we wanted to make larger improvements, we would need something drastically different. More fundamental changes come with greater risk, but one of the side benefits of the incremental approach was that we had a better understanding of how the current algorithm worked and growing confidence that we could make more radical changes.</p>
<h2>Prototyping a new algorithm</h2>
<blockquote>
<p>By Scott Wadsworth, Partner Software Engineer in .NET, and Brian Robbins, Principal Software Engineer in .NET</p>
</blockquote>
<p>When we joined the NuGet Restore performance optimization effort, a lot of great work had already been done. We didn’t know as much about the core system as most of the people already involved, so we spent a little time looking through the code and analyzing traces of execution for common scenarios using PerfView. We also felt it was important to understand how the restore algorithm behaved at scale, to determine its algorithmic complexity as a function of the input size and input complexity.</p>
<p>Before we get into that, let’s talk about what happens when you run restore in your repository. Restore itself can be divided in a few parts. It starts with the reading of projects and configurations, such as evaluating all the project files, reading the <code>NuGet.config</code>, in other words, processing all the inputs. When restoring a solution, all projects are processed before any package downloading begins. For example, <a href="https://learn.microsoft.com/nuget/reference/msbuild-targets#restoring-with-msbuild-static-graph-evaluation">Static Graph Restore</a> speeds up the <em>inputs</em> stage of restore.</p>
<p>We then have the core restore part, which runs on a per project level and it consists of three parts:</p>
<ul>
<li>No-op restore: Takes the information gathered during the <em>inputs</em> step and compares it with the last successful restore if any. If all the <em>inputs</em> and <em>outputs</em> are the same, we&#8217;re done.</li>
<li>Restore resolver: If anything has changed, we then run the restore algorithm on the given inputs. This is where the package graph resolution and package download happen.</li>
<li>Commit step: The step where we write the <em>outputs</em> such as <code>project.assets.json</code> and other NuGet files you&#8217;d find in the <code>obj</code> folder.</li>
</ul>
<p>With this context in mind, our journey began with a couple of experiments.</p>
<p>The first experiment helped us model how NuGet behaved as project size and complexity increased, revealing a near exponential scaling. For this experiment, we created a tool that generated projects that referenced packages to build an arbitrarily complicated restore graph, then timed restore operations on it. We made sure that all packages had been previously downloaded to ensure that we weren’t measuring the quality of our network connections, but were instead measuring only the restore algorithm itself.</p>
<p><img alt="Chart comparing the performance of the NuGet Restore prototype vs the original NuGet Restore algorithm" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/prototyping-0.png" /></p>
<p>We observed that there was a clear degradation in performance that grew much worse the larger a project graph became. We saw a very reasonable restore time of under 5 seconds up until around 200 projects of modest complexity. Then, the cost began to grow more visibly.</p>
<p>The second experiment was a game-changer: it showed us that performing the core restore computation in isolation could be blazingly fast, running in milliseconds. This gave us the confidence to rethink the entire process. We wrote a very simple system that manually loaded the input csproj and NuGet nuspec files and did a rough calculation to arrive at resolved package versions. This prototype didn’t respect important parts of the restore system like overrides and other complex rules of dependencies. But, it got much of the restore algorithm right and was able to run in under a second.  This boosted our confidence that we could address the problem with an alternative implementation and achieve massive performance improvements. At the same time, it was clear that implementing all the existing policy and features would be a challenge.</p>
<p>The general approach we employed was to separate the restore code into chunks that we could reason about separately, and to profile them to understand where we should focus our efforts. These chunks are based on the architecture of the code and are formalized in a <a href="https://github.com/NuGet/NuGet.Client/pull/5650">PR</a> adding instrumentation to the restore algorithm:</p>
<ul>
<li>CalcNoOpRestore: Evaluation of the cache file.</li>
<li>BuildRestoreGraph: Execute the restore algorithm.</li>
<li>BuildAssetsFile: Translate results from the restore algorithm into the assets file format.</li>
<li>WriteAssetsFile: Write the assets file to disk.</li>
<li>WriteCacheFile: Write the cache file to disk.</li>
<li>WritePackagesLockFile: Write the <code>packages.lock</code> file to disk.</li>
<li>WriteDgSpecFile: Write the <code>dgspec.json</code> file to disk.</li>
</ul>
<p>We ultimately determined that BuildRestoreGraph represented about 92% of the wall clock time of the restore across a combination of test and real projects, and that’s where we chose to focus our efforts. We set out to replace this chunk of code, while maintaining the existing interface between this chunk and the others. This strategy provided us with high confidence that we could achieve success. Lots of traces and wandering through the code lead us to a method called <code>RestoreCommand::ExecuteRestoreAsync</code>.  This became the target of our work.</p>
<p>To ensure the correctness of the new restore algorithm, we knew that having a fast and repeatable testing loop would be critical to success. To that end, we settled on using project files (*.csproj) as inputs and NuGet assets files (assets.json) as outputs. Each project was given a durable integer ID to ensure that we compared the right outputs between those generated by the old and new algorithms. After generating a baseline using the old algorithm, we employed diffing tools to compare the outputs from the baseline to those of the new algorithm as we developed it. We knew that getting to a clean diff would ensure that there would not be downstream impacts at compile-time or runtime.</p>
<p><img alt="Diff of assets files" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/prototyping-1.png" /></p>
<p>Initially, we disabled some parallelization to make debuggability easier. We then proceeded by adding a new method after <code>RestoreCommand::ExecuteRestoreAsync</code> that housed the new algorithm, running it after the original baseline parts. This allowed us to start, step-by-step, filling out the output data structure expected by the next step of the restore. The <code>RestoreTargetGraph</code> class contained about 8 primary chunks of data; the first of these being <code>Flattened</code>. With our new computation occurring after the existing algorithm, we could replace the result of the initial computation with our own, and let the rest of the restore operation  (including things like downloads) occur. This enabled us to quickly reason about what input projects we got right and which ones we didn’t. We’d focus on each one we got wrong and reason out the policy that needed to be implemented. Often this required consulting with the NuGet Client team to understand something we didn’t feel was super obvious in the rules that govern restore behavior. We’d often create a standalone test that we could use to understand and debug behavior, <a href="https://github.com/NuGet/Entropy/tree/main/PackageSamples/Projects">many of which are checked in on GitHub</a>. After a few dozen of these discoveries and fixes, we landed on a system that could perform the restore operation without the original algorithm running.</p>
<p>The new algorithm has a fundamentally different design than the previous one. The old algorithm loaded all dependencies into memory, produced a full graph, and then applied policy by bubbling the impact of it across the tree. The new algorithm makes decisions about each package as it is encountered during the resolution process and only generates a flattened representation of the dependency graph. For example, if it encounters a new version of a package, it will update its choice on the spot and avoid triggering additional graph walks as a result of the change. It massively reduces the amount of memory to allocate as well as the number of updates to that memory. The initial prototype of the new algorithm correctly restored about 60% of the large projects we tested.</p>
<p>For more complex projects, the algorithm employs a reset-and-reimport strategy. If a conflict is encountered, the new algorithm clears its list of resolved packages, remembers which package it should take to resolve the conflict, and starts over, this time ignoring previously encountered versions of the package that ultimately led to the conflict. This approach ensured compatibility without introducing additional unwind/undo overhead to the system.</p>
<p><img alt="Mermaid Diagram 9" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid9.png" /></p>
<p>Once all outputs were identical between the two algorithms, the new algorithm could restore what previously took 16 minutes in about 4 minutes. Our mental goal was to be under 2 minutes. This was a great opportunity to apply traditional performance profiling and approaches, which quickly brought the end-to-end time down to just that. An example of a crucial optimization was string encoding. All profiling of both the existing and new algorithm showed that string comparison of package identities was incredibly costly. By converting package names and versions to unique integer IDs, we significantly sped up comparison operations, particularly during dependency resolution. This seemingly small change had an outsized impact, reducing processing time for scenarios with extensive dependency trees.</p>
<p>Collaboration with the NuGet Client team was instrumental in the final stages of implementation. Together, we integrated additional features like improved error handling and diagnostics, all while ensuring backwards compatibility. This cohesive teamwork was the key that allowed us to refine and ultimately ship the revamped restore process.</p>
<p>The results were transformative. Restore times for TeamX were eight times faster with packages pre-downloaded—a shift that enhanced the productivity of developers globally. Beyond the performance gains, this experience underscored the value of approaching legacy systems with fresh perspectives and a willingness to challenge assumptions.</p>
<p>Looking ahead, we’re excited to see how these improvements evolve. User feedback and real-world usage will undoubtedly highlight new opportunities for refinement, ensuring NuGet continues to meet the growing demands of modern development.</p>
<p>At the time of writing, we asked Copilot to summarize this section as a poem for fun:</p>
<p><em>In the quest for a NuGet delight,</em><br />
<em>A prototype emerged in the night.</em><br />
<em>With algorithms anew,</em><br />
<em>And performance that flew,</em><br />
<em>It turned long waits into sheer flight.</em></p>
<h2>Final ten miles</h2>
<blockquote>
<p>By Nikolche Kolev and Jeff Kluge, Principal Software Engineers in NuGet Client</p>
</blockquote>
<p>The news of the success of our prototype spread quickly. It was not too long until teams across Microsoft started asking questions such as “When will you ship this?” and “Can you ship this in .NET 9?”.
.NET 9 was in late previews and would be shipping soon, so we needed to make a decision quickly.</p>
<p>We asked ourselves the “Why, What &amp; How” questions and started crunching numbers. For TeamX, the new restore algorithm resulted in significant developer productivity and compute savings—this was just one product inside Microsoft. On one hand, we wanted other organizations around the globe to achieve similar savings. On the other, we wanted to validate the new algorithm thoroughly to avoid breaking development teams everywhere.</p>
<p>After multiplying the daily savings from TeamX by 365 days, we decided to at least try to ship the new algorithm in .NET 9 instead of waiting an entire year for .NET 10.</p>
<p>During the prototyping phase, the focus was on real-world scenarios. This meant we had validated positive cases and not cases that would result in warnings or errors, so we knew we had some gaps. In particular, we needed to implement the detection and accurate reporting of version conflicts, dependency cycles, and package downgrades as described in <a href="https://learn.microsoft.com/nuget/reference/errors-and-warnings">NuGet Errors and Warnings Reference | Microsoft Learn</a>. These detections are extremely important for package installation scenarios either through the NuGet tooling or by editing the csproj and props files. The other missing piece was <a href="https://learn.microsoft.com/nuget/consume-packages/Central-Package-Management#transitive-pinning">transitive pinning</a>, which was always going to be the next step in the rewrite. Having been a part of the team that wrote the original transitive pinning implementation and knowing the amount of time and effort that went into that, we felt comfortable setting a goal to enable transitive pinning after .NET 9 GA.</p>
<p>The final piece was deciding “how” we were going to achieve our main goal. For a process like restore that runs as part of every build, high quality is critical. Let us say we ship a regression that changes the final set of packages in the dependency graph:</p>
<ul>
<li>Best case: restore fails—it will be clear to the customer that restore is the problem, and, if we document things well, they’ll switch back to the old resolver to unblock themselves.</li>
<li>Worse case: build or tests fail—customer is not necessarily sure where the problem is; it may take a while to diagnose restore as the root cause to fix.</li>
<li>Worst case: runtime failures—issues may be found a long time after release/deployment, and it would be incredibly difficult to root-cause.</li>
</ul>
<p>For the new dependency resolver to reach the quality needed and expected, we were going to need a multi-prong validation: automated tests, real-life repository tests, and, most importantly, dogfooding from our partners in .NET and Visual Studio.</p>
<p>Even before we set the .NET 9 goal, we began productizing the prototype. Initially optimistic, we faced setbacks after running the existing NuGet automation tests and hitting hundreds of failures. When we decided to attempt shipping in .NET 9, we started triaging and addressing those test issues. We had failures that varied anywhere from the number of packages downloaded (<a href="https://github.com/NuGet/Home/issues/13943">usually</a> not breaking) or the number of errors raised (the new algorithm was reporting errors that occured with the old algorithm as well, but that the old algorithm was not reporting), to more serious issues such as version discrepancies. We prioritized <a href="https://github.com/NuGet/NuGet.Client/commit/e68fb0f2d96800ad4b78461fd491886e9e5084b2">getting the code to internal dogfooding builds</a>, which required us to temporarily disable some tests as we continued the polish.</p>
<p>We kept both algorithms in our code and added an MSBuild property, <code>RestoreUseLegacyDependencyResolver</code>, to allow reverting to the old algorithm, alongside a playbook for self-diagnosing dependency problems.</p>
<p>We set-up a test bed of <a href="https://github.com/NuGet/NuGet.Client/blob/dev/test/NuGet.Core.FuncTests/NuGet.Commands.FuncTest/RestoreCommand_AlgorithmEquivalencyTests.cs">algorithm equivalency tests</a>. Given a project, it ran restore twice and compared all the restore outputs, such as the <code>project.assets.json</code> file, NuGet-generated files, and the packages on disk. This test bed drove the majority of bug fixing later in the process. Every single bug we encountered ended up as a test case in this test bed, leading to more than 100 new test cases. Beyond the verification, this test bed became an extremely valuable resource for understanding how NuGet resolution works. By the time we were done, our understanding of both the previous and the new resolver implementations had improved.</p>
<p>In a product like Visual Studio or the .NET SDK, there are many functional and performance tests at different points in the product life cycle. Restore is challenging because a lot of complexity is driven by the user configuration, so it is hard to assess out every scenario possible.</p>
<p>One of the tools we used for validating performance is our <a href="https://github.com/NuGet/NuGet.Client/tree/dev/scripts/perftests">commandline-based performance test suite</a>, which performs a series of runs that mirror common restore scenarios, such as <em>clean</em> restore (used commonly by Continuous Integration/Continuous Deployment pipelines) and <em>warm</em> restore (prevalent in developer machines). Then, it generates a .csv file with the timings. It is a wonderful tool for performance smoke testing. The most valuable <a href="https://github.com/NuGet/NuGet.Client/commit/15991039d603b8da5d2603315fc9c5c2cfb91a07">addition to our perf suite</a> was the ability to run a locally-built NuGet tool against test repositories. This massively sped up our progress because it meant we were always literally one minute away from running performance tests. In the quest for simplification of the algorithm, we had removed a lot of the parallelization and async work, and, while that improved the performance of the <em>warm</em> NuGet Restore scenarios, it did regress the <em>clean</em> NuGet Restore scenarios significantly for some repositories.</p>
<table>
<thead>
<tr>
<th></th>
<th>NuGet.Client repo</th>
</tr>
</thead>
<tbody>
<tr>
<td>Clean NuGet Restore</td>
<td>+150.0% (+36.0s)</td>
</tr>
<tr>
<td>Warm NuGet Restore</td>
<td>-15% (-0.8s)</td>
</tr>
</tbody>
</table>
<p>Digging into the numbers from our runs showed us that we needed to <a href="https://github.com/NuGet/NuGet.Client/pull/6026">add back the parallelization of package downloads</a>. These scripts continue to be essential in our work, and the numbers discussed in the next section were obtained from running these scripts.</p>
<p>In parallel to improving our automation to validate functionality and performance, we enlisted help from our friends that keep the .NET and Visual Studio engineering systems running. With their assistance, we setup workflows that allowed us to quickly validate the latest changes in Visual Studio and .NET repositories.</p>
<p>One strategy was to verify equivalency. We would run a restore with and without the changes and then compare the results as <a href="https://learn.microsoft.com/nuget/consume-packages/package-references-in-project-files#nuget-dependency-resolver">per the playbook</a> we mentioned earlier. This was a foolproof approach, but it was laborious since most builds are not authored in a way to allow for multiple restores. Another strategy was full CI verification. As the name suggests, we would run the full CI with only our bits replaced and ensure there were no regressions in the build or test steps.</p>
<p>Given that we had multiple team members 100% committed to this work, countless of bugs never made it further than our local dev machines, bugs ranging anywhere from downgrade logic detection issues to issues with handling missing versions. If you’re interested in some of the gory details, the changes in the <code>DependencyGraphResolver</code> class (<a href="https://github.com/NuGet/NuGet.Client/commits/dev/src/NuGet.Core/NuGet.Commands/RestoreCommand/DependencyGraphResolver.cs">link</a>) do capture nearly all issues that ended up on a CI pipeline at some point in this process.</p>
<p>By September, the algorithm was available for internal use, and almost every single person working in the Developer Division had adopted it during their daily work.</p>
<p>With all the work we had done around the margins, we still had some runway to tackle the biggest remaining challenge of this rewrite, and that was <a href="https://learn.microsoft.com/nuget/consume-packages/central-package-management#transitive-pinning">transitive pinning</a>. Transitive pinning is the feature that allows you to overwrite a transitive package version without adding a direct reference. It works by promoting a transitive dependency to a direct one on your behalf when necessary.</p>
<p>Let us take an example:</p>
<p><img alt="Mermaid Diagram 10" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid10.png" /></p>
<p>In the above scenario, a project is being <a href="https://learn.microsoft.com/nuget/consume-packages/central-package-management">centrally managed</a>. Given that Package B is not a direct dependency, version 2.0.0 is not considered, and we get 1.0.0 as the lowest version that satisfies all the requirements <a href="https://learn.microsoft.com/nuget/concepts/dependency-resolution#dependency-resolution-rules">per the dependency resolution rules</a>.</p>
<p><img alt="Mermaid Diagram 11" src="https://devblogs.microsoft.com/dotnet/wp-content/uploads/sites/10/2025/04/mermaid11.png" /></p>
<p>Now if we enable pinning, whenever the resolver sees Package B 1.0.0, it sees that 2.0.0 is wanted by the user, so it will upgrade that Package B to 2.0.0. If at any point Package A removes the dependency to Package B, the pinned reference will disappear as well.</p>
<p>Direct dependencies in NuGet have a very important meaning in the resolver, as documented by the <a href="https://learn.microsoft.com/nuget/concepts/dependency-resolution#direct-dependency-wins">Direct Dependency Wins rule</a>, and the introduction of the pinning concept means that we need to consider all resolution rules and rule combinations differently than we do in the non-pinning case. The sheer volume of decisions that the resolver needs to make is greater, and that means more involved tracking of previously seen packages and more involved conflict resolution.</p>
<p>One of the big benefits of the rewrite was the simplification of the code, which, coupled with our improved understanding of all the rules and edge cases, allowed us to make progress faster than we originally expected. It took a month of iterations (while working on other bugs as they got discovered) including ten hours of pair programming per week to get to a state where all of our pinning tests and repositories were working consistently.</p>
<p>By the time .NET 9 RC1 shipped, transitive pinning was enabled by default, things had fallen into place, and we felt confident and proud of the work we had done so far.</p>
<p>We did have a few known issues, some in uncommon scenarios (that we found through exploratory testing), some in more critical paths. Notably, we had some bugs in how the new resolver integrated with lock files. To work around these issues, we <a href="https://github.com/NuGet/NuGet.Client/blob/7a84f1ecdb1df83034aa639e496f3b25a16d94ec/src/NuGet.Core/NuGet.Commands/RestoreCommand/RestoreCommand.cs#L149">disabled the new resolver for lock files</a> in code, which means there is no way to opt into the new resolver if you’re using lock files. The fixes for lock files <a href="https://github.com/NuGet/NuGet.Client/pull/6253">are coming in the .NET 10 SDK</a>.</p>
<p>As .NET 9.0 and Visual Studio 17.12 rolled out, we paid close attention to feedback, and, while there were things to fix, we were happy with the overall outcomes. .NET 9.0.200 and Visual Studio 17.13 do have 6 new fixes that were not in 17.12, and we’ll continue to address newly reported NuGet Restore issues.</p>
<p>We are not done yet! We know we have further opportunities for performance improvements. Our performance journey never stops.</p>
<h2>Conclusion and next steps</h2>
<blockquote>
<p>By Arturo Ortiz, Principal Engineering Manager for the NuGet Client team</p>
</blockquote>
<h3>Final performance and memory gains</h3>
<p>For summarizing performance and memory improvements stemming from our effort, we will consider three versions of the dependency resolver algorithm used by NuGet Restore:</p>
<ol>
<li>The unoptimized legacy algorithm that shipped until .NET 8.0.200.</li>
<li>The optimized legacy algorithm—that is, the legacy algorithm with all the performance and memory optimizations that we mentioned in the <em>First Step: Performance Optimizations</em> section of this document. This version of the algorithm shipped in .NET 8.0.300.</li>
<li>The new algorithm which was shipped in .NET 9.0.100.</li>
</ol>
<p>As mentioned before, for the internal team at Microsoft that started this whole effort, TeamX, the optimized legacy algorithm reduced the average NuGet Restore time in their Azure DevOps Continuous Integration/Continuous Deployment pipeline from 32 to 16 minutes. Later, the new algorithm brought that time down to only 2 minutes.</p>
<p>In our own benchmarks, the new algorithm performed 15-32% faster for one of our reference repositories, <a href="https://github.com/OrchardCMS/OrchardCore">OrchardCore</a>, compared to the optimized legacy algorithm:</p>
<table>
<thead>
<tr>
<th>OrchardCore repo</th>
<th>20-core (x64)</th>
<th>8-core (x64)</th>
</tr>
</thead>
<tbody>
<tr>
<td>Clean NuGet Restore</td>
<td>-31.7% (-18.5s)</td>
<td>-15.0% (-35.9s)</td>
</tr>
<tr>
<td>Warm NuGet Restore</td>
<td>-23.8% (-8.0s)</td>
<td>-22.6% (-17.1s)</td>
</tr>
</tbody>
</table>
<p>In the above table, <em>warm</em> NuGet Restores are those for which all packages have already been downloaded to internal storage and knowledge of all NuGet packages is available in the <a href="https://learn.microsoft.com/nuget/consume-packages/managing-the-global-packages-and-cache-folders">http cache</a>. In contrast, <em>clean</em> NuGet Restores don’t have anything downloaded or cached.</p>
<p>For repositories targeting multiple frameworks, the optimized legacy algorithm still does better than the new one when running a <em>clean</em> NuGet Restore on a machine with many cores, as exhibited in <a href="https://github.com/NuGet/NuGet.Client">our own code repository on GitHub</a>. This is because the legacy dependency resolver parallelized tasks extremely well. Of course, this is something that we will be addressing in future versions of .NET.</p>
<table>
<thead>
<tr>
<th>NuGet.Client repo</th>
<th>20-core (x64)</th>
<th>8-core (x64)</th>
</tr>
</thead>
<tbody>
<tr>
<td>Clean NuGet Restore</td>
<td>+19.2% (+25.8s)</td>
<td>-26.3% (-26.6s)</td>
</tr>
<tr>
<td>Warm NuGet Restore</td>
<td>-1.3% (-5.3s)</td>
<td>-25.9% (-8.0s)</td>
</tr>
</tbody>
</table>
<p>Because the new algorithm doesn’t need to build a graph to represent the full dependency closure of an application, we knew that heap memory allocations would be lower, particularly for large repositories. As expected, compared to the optimized legacy algorithm, we saw 5% less heap memory allocations for OrchardCore and up to 25% less heap memory allocations for TeamX. However, we see 34% less total (heap + stack) allocations for OrchardCore when comparing against the unoptimized legacy algorithm which, as discussed earlier, used recursion and instantiated a state machine object for tracking async work at each level of the stack.</p>
<h3>Future work</h3>
<p>It is hard to avoid regressions when conducting a massive code rewrite like the one we did. After release, the .NET community has reported several issues which we’ve been fixing in subsequent releases, and this is likely to continue for the next several months.</p>
<p>Once we’ve addressed all issues reported, we’d like to address the following known limitations:</p>
<ol>
<li>Add more parallelism when targeting multiple frameworks to address the single known performance regression mentioned earlier.</li>
<li>Enable the new dependency resolver for projects using lock files.</li>
</ol>
<p>We also identified these opportunities for future performance improvements:</p>
<ol>
<li>Add more parallelism to MSBuild when using <a href="https://learn.microsoft.com/nuget/reference/msbuild-targets#restoring-with-msbuild-static-graph-evaluation">Static Graph-based Restore</a>.</li>
<li>Eliminate the need of an intermediary <code>project.assets.json</code> file and instead have NuGet create the necessary MSBuild tasks directly.</li>
</ol>
<h3>Learnings</h3>
<p>While it is evident today that our efforts to improve NuGet Restore performance were worthwhile, the final outcome was quite unclear at the beginning of our journey.</p>
<p>To start off, we didn’t know how much faster we could make NuGet Restore by optimizing the existing algorithm. To complicate things further, as mentioned in the <em>First Step: Performance Optimizations</em> section, none of the initial changes resulted in measurable improvements in our performance benchmarks. It was only after implementing several fixes that we began to see positive results.</p>
<p>Rewriting the dependency resolver algorithm was almost unthinkable at the onset due to the risk of  breaking a fundamental component relied upon by millions of .NET developers around the world to build, test, and deploy their code. In fact, even as we had gained confidence in our understanding of the original algorithm and our ability to come up with a new implementation, some people expressed concern about our plan and rightfully worried about the amount of time that we were dedicating to this effort.</p>
<p>This was a humbling journey which gave us valuable learnings that we would like to share with you:</p>
<ol>
<li>Performance improvements can unlock tremendous customer and business value. At Microsoft, better NuGet Restore performance translated into significant infrastructure cost savings and improved developer productivity and morale for thousands of engineers.</li>
<li>Be brave–sometimes, it’s necessary to ignore skepticism (and even our own fears) to achieve great results.</li>
<li>Related to the above point–define a reasonable time investment upfront and do not give up early!</li>
<li>Bring people with “fresh eyes” into the project. In our case, Scott, Brian, and Eric were not part of the NuGet Client team, and this gave them the freedom to ask seemingly naïve questions and challenge longstanding assumptions on how something worked or why it worked that way.</li>
<li>Beware of adding too many people during early experimentation. In our case, only two people worked on the prototype for the new dependency resolver until a clear direction was established. This allowed them to experiment, iterate, and pivot into new directions quickly.</li>
<li>Have daily check-ins. These created a lot of clarity and energy for us.</li>
<li>Invest in a really good set of tests that can help you iterate quickly and confidently. Looking back, we all feel strongly that rewriting the dependency resolver algorithm would not have been possible without excellent functional and performance tests.</li>
<li>Boost your confidence by testing complex scenarios early. Once we had a working prototype, we tested with a few of the largest and most complex repositories that we could find. Then, we dedicated time to fix all the issues found. Knowing that the algorithm worked for these repositories increased our confidence (as well as our management’s) in delivering this project successfully.</li>
</ol>
<p>That&#8217;s all! Feel free to share your own learnings in the comments section below.</p>
<p>If you encounter any problems with NuGet Restore, visit our <a href="https://learn.microsoft.com/nuget/consume-packages/Package-References-in-Project-Files#nuget-dependency-resolver">guide</a> which includes instructions to create issues in our GitHub repository.</p>
<p>Thank you for reading and happy coding!</p>
<p>The post <a href="https://devblogs.microsoft.com/dotnet/rewriting-nuget-restore-in-dotnet-9/">How we ended up rewriting NuGet Restore in .NET 9</a> appeared first on <a href="https://devblogs.microsoft.com/dotnet">.NET Blog</a>.</p>
