---
layout: post
title: Upgrading .net core 1.1 to 2.0 for docker users
excerpt_separator: <!--more-->
---

For my latest feature we have been working on a microservice running across 4 docker containers.

All the containers are running on a runtime base image we build for .net core 1.1 - which has 
been working really well.

We are now starting the process of upgrading to .net core 2.0 (2.0.3)...

<!--more-->

Our current runtime image Dockerfile is using this mechanism for installing the .net core 1.1 runtime:

```
# Install .NET Core
ENV DOTNET_VERSION 1.1.4
ENV DOTNET_DOWNLOAD_URL https://dotnetcli.blob.core.windows.net/dotnet/release/1.1.0/Binaries/$DOTNET_VERSION/dotnet-debian-x64.$DOTNET_VERSION.tar.gz

RUN curl -SL $DOTNET_DOWNLOAD_URL --output dotnet.tar.gz \
    && mkdir -p /usr/share/dotnet \
    && tar -zxf dotnet.tar.gz -C /usr/share/dotnet \
    && rm dotnet.tar.gz \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet
```

This is basically the same as what happens in this runtime image published to dockerhub by microsoft:

https://github.com/dotnet/dotnet-docker/blob/master/1.1/runtime/jessie/Dockerfile

This has been working well.  We used this runtime base image for all our .net core containers, and if the container
was running a web workload it would run ASPNET core through referencing one or more nuget packages.

# Upgrade attempt #1

The first upgrade attempt was a simple one - let's bump the runtime installation method in the container, using the same approach as the official dotnet runtime docker hub image:

https://github.com/dotnet/dotnet-docker/blob/master/2.0/runtime/jessie/amd64/Dockerfile


```
# Install .NET Core

ENV DOTNET_VERSION 2.0.3
ENV DOTNET_DOWNLOAD_URL https://dotnetcli.blob.core.windows.net/dotnet/Runtime/$DOTNET_VERSION/dotnet-runtime-$DOTNET_VERSION-linux-x64.tar.gz
ENV DOTNET_DOWNLOAD_SHA 4FB483CAE0C6147FBF13C278FE7FC23923B99CD84CF6E5F96F5C8E1971A733AB968B46B00D152F4C14521561387DD28E6E64D07CB7365D43A17430905DA6C1C0

RUN curl -SL $DOTNET_DOWNLOAD_URL --output dotnet.tar.gz \
    && echo "$DOTNET_DOWNLOAD_SHA dotnet.tar.gz" | sha512sum -c - \
    && mkdir -p /usr/share/dotnet \
    && tar -zxf dotnet.tar.gz -C /usr/share/dotnet \
    && rm dotnet.tar.gz \
    && ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet
```

So far so good.

I then upgrade the projects to target aspnetcore2.0, updated packages to use the new 2.0 packages, and replaced a number of aspnet core nuget package references with the single meta package:

```
<PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.3" />
```

Builds work, we have some shiney new images, I then attempt to deploy them.

The console apps we are using for background jobs deploy effortlessly.

The aspnet.core apps didn't go so smoothly though, and we get this interesting failure on startup:

```
Error: An assembly specified in the application dependencies manifest (MyCompany.MyService.deps.json) was not found:

package: 'Microsoft.ApplicationInsights.AspNetCore', version: '2.1.1'

path: 'lib/netstandard1.6/Microsoft.ApplicationInsights.AspNetCore.dll'

This assembly was expected to be in the local runtime store as the application was published using the following target manifest files:
aspnetcore-store-2.0.0-linux-x64.xml;aspnetcore-store-2.0.0-osx-x64.xml;aspnetcore-store-2.0.0-win7-x64.xml;aspnetcore-store-2.0.0-win7-x86.xml
```   

Curious - we are not using application insights (and don't have the package referenced, or application insights initialized) - time to investigate.
 
# Upgrade attempt #2

After some investigation we identify that we are missing the aspnet store files necessary to run ASP.Net core 2.0.0 or 2.0.3 based projects.

To remedy this we need to install the runtime store into our images - referring to the docker files once more, but this time for the aspnet-docker image:

https://github.com/aspnet/aspnet-docker/blob/master/2.0/jessie/runtime/Dockerfile

We can see how this can be done..

```
# set up the runtime store
RUN for version in '2.0.0' '2.0.3'; do \
        curl -o /tmp/runtimestore.tar.gz https://dist.asp.net/runtimestore/$version/linux-x64/aspnetcore.runtimestore.tar.gz \
        && export DOTNET_HOME=$(dirname $(readlink $(which dotnet))) \
        && tar -x -C $DOTNET_HOME -f /tmp/runtimestore.tar.gz \
        && rm /tmp/runtimestore.tar.gz; \
    done
```

I update my runtime base image and attempt to build it.

I'm getting an error about the `aspnetcore.runtimestore.tar.gz` file being and invalid tar archive.

I hit the URL directly and get an XML 404 response

```
<Error>
<Code>ResourceNotFound</Code>
<Message>
The specified resource does not exist. RequestId:b396a958-001e-0005-79e5-6ce076000000 Time:2017-12-04T09:54:51.0138005Z
</Message>
</Error>
```

I search around but alas I'm unable to find any replicas of this file mirrored anywhere else.

I then investigate further and discover that I can disable this meta-package behavior (though I end up with slower to build images and no pre-JIT'd assemblies) by updating my .csproj
to contain this:

```
  <PropertyGroup>    
    <PublishWithAspNetCoreTargetManifest>false</PublishWithAspNetCoreTargetManifest>
  </PropertyGroup>
```

So I update the .net core projects as necessary, build new images and attempt to deploy again.

But it fails in the same way.

# Upgrade attempt #3

Though I couldn't source a runtime store archive from a reputable source, ubuntu packages are available for aspnet core hosting which do include it.

I update the my base runtime image docker file to first install a dependency to allow me to use https package repositories:

```
RUN apt-get update \
      && apt-get install -y --no-install-recommends \
         apt-transport-https \
         ... (and some more things you don't care about) 
```

And I then replace the previous installation process with this:

```
# package feed for .net core 2.0 packages
RUN curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
RUN mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
RUN echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-xenial-prod xenial main" > /etc/apt/sources.list.d/dotnetdev.list

# Use the microsoft package feed to install dotnet hosting (which includes dotnet store, runtime and hostfixr packages etc.)
RUN apt-get update && apt-get install -y --no-install-recommends dotnet-hosting-2.0.3
```

(FYI don't worry about the lack of SUDO on those commands, we just change to a non-priveleged user in the dockerfile for our containers rather then base images)

At this point we have a working base image

But the deploys are still failing.  This time the instances startup, Kestrel reports it's ready to recieve traffic, but then there is no evidence of the container serving responses to health checks.

# Upgrade attempt #4

In the process of updating from ASP.Net core 1.1 to 2.0 some things had to be changed.

We have our containers sitting in front of AWS ALB (Application load balancer) with TLS to the origin.

For this to work, previously our Program.cs would look something like this:

```
var host = new WebHostBuilder()
    .UseKestrel(options => options.InstallTransportSecurity())
    .UseContentRoot(Directory.GetCurrentDirectory())
    .UseIISIntegration()
    .UseStartup<Startup>()
    .UseUrls("http://*:5008", "https://*:5018")
    .Build();
```

Where .UseUrls() would handle the binding to IPAddress.Any (0.0.0.0) on specific ports when using * in the path.

FYI `options.InstallTransportSecurity()` is just an extension method on a helper class we have that sets up transport security and optionally client-certificate validation, via some strongly typed configuration.

When upgrading to aspnet core 2.0 Kestrel can no longer utilize UseUrls for setting up bindings for TLS... So referring to the endpoint configuration guide:

https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore2x#endpoint-configuration

There is an example:

```
public static IWebHost BuildWebHost(string[] args) =>
       WebHost.CreateDefaultBuilder(args)
           .UseStartup<Startup>()
           .UseKestrel(options =>
           {
               options.Listen(IPAddress.Loopback, 5000);
               options.Listen(IPAddress.Loopback, 5001, listenOptions =>
               {
                   listenOptions.UseHttps("testCert.pfx", "testPassword");
               });
           })
           .Build();
```

Problematically though - the example uses IPAddress.Loopback - this works fine for local development, but doesn't fare so well when utilized inside your docker container where those localhost/loopback ports can not be mapped to external ports for the container.

The fix was fairly simple, which is just to switch it back to being an Any Address `IPAddress.Any` - at which point things began working again.

Well almost...

# Upgrade finale

After upgrading to .net core 2.0 we found some code we had in place that used a client certificate for inter-service communication that was causing outbound connections to fail.

We had a feature flag to control this behavior (and don't rely on it because the services sit behind an ALB, which strips the client certificate anyway) - so we switched it off and things began to work correctly again.

The code for configuration the client certificate was fairly rudimentary - aproximately it looks like this:


```
var handler = new HttpClientHandler();
handler.ClientCertificateOptions = ClientCertificateOption.Manual;
handler.ClientCertificates.Add(_clientCertificate.Value);
if (IgnoreSelfSignedCertificates) {
    handler.ServerCertificateCustomValidationCallback = (msg, cert, chain, sslPolicyErrors) => true;
}
return new HttpClient(handler);
```

The failure mode was unusual in that the connection was not refused, and no internal errors were returned.  

It was just the request would eventually timeout due to defaults in the HttpClient class (which has a 100second out of the box timeout) - this surfaces as a task cancellation.

At this stage we have not determined the root cause in this behavior change, though a possibly candidate could be the move in .net core 2.0 to use libcurl defaults if no SSL protocols are specified.

Interestingly while investigating the issue I cam across this thread:

https://github.com/dotnet/corefx/issues/9728

Which indicates ServerCertificateCustomValidationCallback will just not work in MacOS currently.   I develop in Rider on Ubuntu 17.10, so don't have these issues, but for my workmates who mostly use macbooks this would have been quite problematic with my solution to ignore self-signed certs.

In fact doing a google search for .net core + libcurl raises a lot of interesting questions/concerns, such as a dramatically poor performance for libcurl requests in older linux distros and quite varied behavior depending on underlying O/S.

# Conclusion

The fun thing about this story is we didn't take production or even QA environments down to test and roll out a significant .net core upgrade.

We also maintained our ability to roll back to a known good state with a simple revert or re-deploy of a previously built .net core 1.1 image.