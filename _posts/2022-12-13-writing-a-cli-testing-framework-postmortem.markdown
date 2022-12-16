---
layout: post
title:  "Writing a CLI Testing Framework - A Postmortem"
date:   2022-12-13 8:23:10 -0800
categories: generaldev
---
As it turns out, writing a [testing framework](https://github.com/zeplar-exe/ClUnit/tree/1.1.0) isn't all that simple. The actual library was a breeze, no doubt. However, one detail I overlooked was the `dotnet test` command. `dotnet test` calls `testhost.exe`, a file that gets built based the [Microsoft.NET.Test.Sdk](https://www.nuget.org/packages/Microsoft.NET.Test.SDK) package (I think). The test host, to my knowledge, is hard-coded to work with nunit, xunit, and maybe a few others.

Thus, my [original idea](https://zeplar-exe.github.io/templar-blog/jekyll/update/2022/12/10/writing-a-cli-testing-framework.html) made use of a test runner console app alongside the library itself. The runner would load an assembly and attempt to find any `ClITestAttribute` attached to methods. Of course, this immediately ran into reflection, versioning, and other issues:

1. What if the target assembly isn't using the same package version of the runner? (the runner referenced the NuGet package)

I intended for a simple check to determine whether the target assembly contained my attribute in the first place. Just a small problem though; my attribute is defined in the ClUnit package, not the target assembly. So the natural solution would be to load the referenced assemblies, right? Well I guess not, since the referenced assemblies don't actually exist on the file system without a very specific setup.

2. How does exception handling work when we're calling a loaded assembly?

This one was quite dirty, as I had to hook up a delegate to [AppDomain.FirstChanceException](https://learn.microsoft.com/en-us/dotnet/api/system.appdomain.firstchanceexception?view=net-6.0). Of course, this means the exception handling is separate from the actual method call, which could lead to race conditions. 

But that's not all! Since I needed to check for `AssertSuccessException` and `AssertFailedException`, and actually imitate catching the exception. Thus, I'm also forced to clear the system out `TextWriter` while handling the exception. Who knows what problems this could cause in asynchronous tests and applications that use threading.

3. Does a single version of the runner work with multiple versions of the library?

No. No it does not.

StackOverflow was very unhelpful in this regard. The only solutions given were variations of "Create an interface per version" and "As long as there's no breaking changes, it shouldn't matter". While the latter is true, it doesn't help in the case of a dynamically loaded assembly. Especially considering that `CliTestAttribute` would probably have more added to it in the future. Thus leading to lots of stringly-typed, poor boilerplate.

---------------

<br/>

So what was the solution? Just make it a haphazard NUnit extension. There was really nothing special about my "framework". You could run Cmd commands, and I had a few assertions in place, but there was no real reason to make it separate from everything else.