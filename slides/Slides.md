---
title       : Buildpacks for Ops Teams
author      : Daniel Mikusa
description : Cloud Native Buildpacks make containerization easy for developers, but what's in it for ops teams? This talk demonstrates how buildpacks can transform operations workflows, offering tangible benefits for the teams responsible for deploying and maintaining applications in production.
keywords    : cloud native, buildpacks, paketo
marp        : true
theme       : jobs
paginate    : true
---

<style>
.columns {
    display: flex;
}
.column {
    flex: 1;
}
.center-img img {
    display: block;
    margin-left: auto;
    margin-right: auto;
}
.only-img img {
    margin-top: 1.5em;
    display: block;
    margin-left: auto;
    margin-right: auto;
}
</style>

<!-- 
_class: titlepage
_footer: Photo by <a href="https://unsplash.com/@michelebit_?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Michele Bitetto</a> on <a href="https://unsplash.com/photos/yellow-curtained-glass-building-84ZA1jFsfzM?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
_paginate: false
-->
![bg left:40%](https://raw.githubusercontent.com/dmikusa/buildpacks-for-ops/refs/heads/main/slides/img/michele-bitetto-84ZA1jFsfzM-unsplash.jpg)

# Buildpacks for Ops Teams

<!-- 
Welcome! This is my session. Let's get started!
-->

---

<div class="columns">
<div class="column">

## Daniel Mikusa

- Staff Software Engineer @ 7SIGNAL, Inc
- Paketo Steering Committee Member
- Cloud-Native Buildpacks Maintainer

### Contact Me

- <small>dan@mikusa.com</small>
- <small>https://github.com/dmikusa</small>
- <small>https://www.mikusa.com</small>

</div>
<div class="column center-img">

![drop-shadow width:10em](https://raw.githubusercontent.com/dmikusa/buildpacks-for-ops/refs/heads/main/slides/img/7SIGNAL-LOGO-RGB-CLR-LGHT-BG.svg)
![drop-shadow width:10em](https://paketo.io/v2/images/logo-paketo-dark.svg)
![drop-shadow width:10em](https://buildpacks.io/images/buildpacks-logo.svg)

</div>
</div>

<!--
Who am I? I work at 7SIGNAL, a leader in WiFi optimization. In a nutshell, we write performance monitoring software that helps make your WiFi awesome.

I also help with a couple of OSS projects, Cloud-Native Buildpacks and Paketo Buildpacks. Both, great tools for building your container images and the subject of today's discussion.

While I'm not covering WiFi in this talk, if you're curious feel free to come chat after the session.
-->

---

# Slides

<div class="center-img">

![drop-shadow height:12em](https://raw.githubusercontent.com/dmikusa/buildpacks-for-ops/refs/heads/main/slides/img/qr-code.png)

</div>
<div style="text-align: center; padding-top: 0.5em;">

[https://github.com/dmikusa/buildpacks-for-ops]()

</div>

<!--
Slides are available at the link above.
-->

---

# Why are we here today?

<div class="columns" style="margin-top: 3em;">
<div class="column">

1. A Quick primer
2. Problems Solved
3. Why Ops Teams Should Care?

</div>
<div class="column">

![drop-shadow width:10em](https://buildpacks.io/images/buildpacks-logo.svg)
![drop-shadow width:10em](https://paketo.io/v2/images/logo-paketo-dark.svg)

</div>

---

# What are Cloud Native Buildpacks?

<div class="columns">
<div class="column">

<h2>What they are?</h2>
<ul>
    <li>Source Code => OCI</li>
    <li>Multiple Languages</li>
    <li>Integrated into Platforms</li>
    <li>A specification</li>
</ul>

</div>
<div class="column">

<h2>What they're not?</h2>
<ul>
    <li>Total replacement for Dockerfiles</li>
    <li>Replacement for CI/CD</li>
    <li>Dev-only Tool</li>
    <li>An implementation</li>
</ul>

</div>
</div>

<!--
What they are:

- CNBs have a focused purpose, to turn source code into OCI images.
- They are a general standard and support many different languages.
- They are suited well for integration into platforms and are in many already (CF, Heroku, Fly.io, AWS AppRunner, Google Cloud Run)
- They are a specification (platform + buildpacks). It defines how buildpacks work and how you integrate them into your platforms.

What they are not:

- They do not attempt to completely replace Dockerfiles. There are still some good use cases for Dockerfiles. They do replace Dockerfiles for use when it comes to building application containers.
- They're not replacing CI/CD, although there are some common parts to both. Both build your app, both create artifacts. CNBs are usually integrated into your existing CI/CD pipelines some point after tests have run.
- They're not a dev-only tool. They're great for Ops folks too, and that's what we're talking about today.
- They're not an implementation. CNB does not provide a working set of buildpacks, which is where Paketo buildpacks come in. Paketo is an implementation of the CNB specification for most popular languages: Java, .NET, Python, Node.js, Ruby, Go, Rust, PHP, and Web Servers.
-->

---

# What are Cloud Native Buildpacks?

<div class="columns">
<div class="column">

Technically:

- A build + run executable 
- A metadata config file called `buildpack.toml`
- Bundled as an OCI image

</div>
<div class="column">

![drop-shadow width:10em](https://raw.githubusercontent.com/dmikusa/buildpacks-for-ops/refs/heads/main/slides/img/buildpack-contents.png)


</div>


---
<!-- 
_footer: Photo by <a href="https://unsplash.com/@punttim?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Tim Gouw</a> on <a href="https://unsplash.com/photos/man-wearing-white-top-using-macbook-1K9T5YiZ2WU?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>
-->

![bg right:40%](https://raw.githubusercontent.com/dmikusa/buildpacks-for-ops/refs/heads/main/slides/img/tim-gouw-1K9T5YiZ2WU-unsplash.jpg)

# Problems Solved: Dockerfiles

- Dockerfile copy & paste
- Dockerfile sprawl

<!--
You need to containerize an app. Where do you get the Dockerfile?
a.) Copy & Paste from another repo
b.) Copy & Paste from Stack Overflow or a blog post
c.) Let AI tools generate it

Most people don't have time to sit and generate one from scratch, check that it's following all the best practices (or even know of them), so any issues or problems get copied and pasted with the Dockerfile.

If you have enough applications at your company, then you hit a new issue. What happens when you want to update all of these unique Dockerfiles? You need to analyze them, because there may be differences in the files, and then buckle in to submit a whole bunch of pull requests. It's not a fun process.

Cloud Native buildpacks have no Dockerfiles, so this problem is gone. The knowledge and commands in the Dockerfiles are codified in the buildpacks & shared that way. 
-->

---

# Problems Solved: Best Practices

<div class="columns">
<div class="column">

<ul>
    <li>Base image too big</li>
    <li>Base image too small</li>
    <li>Base image not maintained</li>
    <li>Not your golden base image</li>
    <li>Forgets to use multi-stage builds</li>
    <li>Implements multi-stage incorrectly</li>
</ul>

</div>
<div class="column">

<ul>
    <li>Bad layer ordering (cache invalidation)</li>
    <li>Bad layer breakdown (decreases sharing)</li>
    <li>Runs the app as root</li>
    <li>No PID1 handler</li>
    <li>Forget company CA certs</li>
    <li>exec vs shell to start process</li>
</ul>

</div>
</div>

<!--
Dockerfiles are easy to write, but hard to get correct. A quick search will show you tons of articles talking about the "best practices" you should follow when writing a Dockerfile. There are dozens of them, some are obvious and some can be pretty subtle.

As a busy developer, you may be unaware of these best practices or you may not have the time to go through them all. As mentioned earlier, copy & paste is the most common way to create a Dockerfile, so you're just hoping someone else before you put in the time to do this right.

The great thing about Buildpacks is that you don't have to hope. The buildpack authors put in the time to get it right, and codify that knowledge so every time you run the buildpack, things are done correctly and you get the best possible image.
-->

---

# Why should Ops care?

## App Problems == Ops Problems

<!--
# TODO: add graphic
The app team might write the Dockerfile, but the Ops teams are running these images or are running the platforms running these apps. When there are vulnerabilities or problems with the app, it's often the Ops teams that have to step in and help.
-->

---

# How can Buildpacks Help Ops

1. Ops gets control

<!--
There's no more Dockerfile, so no more mess. Builds are done in a unified way, according to your designs. You define the base images, you define the buildpacks, the tools run and produce safe, consistent images.
-->

---

# How can Buildpacks Help Ops

1. Ops gets control
2. Know what's running

<!--
Buildpack generated images automatically have SBOMs created for them by default, so you know what's in the image and what's running.

In addition, buildpack builds are reproducible. If there's ever a question about the contents of an image, you can rebuild from source and assert down to the byte that nothing has changed.
-->

---

# How can Buildpacks Help Ops

1. Ops gets control
2. Know what's running
3. Building a Platform

<!--
Buildpacks are more than just a way to create images, they're a way to allow Ops teams to provide an almost PaaS-like platform to their Dev teams. This is a win-win and makes life easier for both teams.
-->

---

# How can Buildpacks Help Ops

1. Ops gets control
2. Know what's running
3. Building a Platform
4. Fast rebuilds & rebase support

<!--
Buildpacks efficiently cache build metadata, and allow for very quick rebuilds of applications.

If that's not fast enough though, buildpacks support the ability to rebase images. When there are OS updates to your base image, this allows you to efficiently swap out the base layer of your application images with a new, updated base layer in a fraction of the time for a rebuild.

If you're building a platform, rebase is huge because you can confidently patch applications en masse when that next CVE strikes.

//TODO: graphic for rebase
-->

---

# How can Buildpacks Help Ops

1. Ops gets control
2. Know what's running
3. Building a Platform
4. Fast rebuilds & rebase support
5. Buildpacks are multi-language & multi-arch

<!--
Buildpacks are a language agnostic tool, so you can find buildpacks to support many different languages and even create your own. Dev teams don't need to worry, they just initiate a build and the buildpacks handle all the language specifics.

In addition, buildpacks support multiple architectures, so you can build app images for AMD64 or ARM64 or whatever comes along next.
-->

---

# Levels of Customization

Out-of-the-box: Paketo Buildpacks

- Base images: Ubuntu Jammy and Noble or UBI 8-10
- Four run image sizes: static, tiny, base & full*
- Languages: Java, Java Native Image, Go, .NET, Node.js, Python, Ruby, Rust, and web servers.
- Buildpack-less builders as well

<!--
Buildpacks can be customized in a number of ways to fit your needs, but let's cover what you get out-of-the-box without any customization.
-->

---

# Levels of Customization

Custom Builders:

- Build image
- Run image
- Set of buildpacks
- Optional custom env variables

<!--
This is the first level of customization. A builder is a base image + run image + buildpacks + metadata. This level of customization allow you to make your own collection for your company.

For example, if you only do Java and you prefer a RedHat base, then you can make a builder with UBI9 build/run images and just two buildpacks, Java & Java Native Image.

// TODO: image of using a custom builder
-->
---

# Levels of Customization

Custom Paketo Run Image

<!--
The next level is a custom Paketo run image. Let's say you need some additional packages for your apps or you need to add some CA certs, so create a Dockerfile and base from your Paketo run image of choice. You can then add customize it, build and publish to a registry.

Your users can then use the custom run image directly with the `--run-image` flag when building or you can create a custom builder with the Paketo build image and your custom run image.
-->

---

# Levels of Customization

Custom Base Images

- Pick your base image
- Customize it
- Add CNB user/metadata
- Push to OCI Registry
- Repeat for the run image
- Publish a custom builder
// TODO: this is quite complicated for a single slide, is this worth the time to explain? Or should this just be a mention that you can do that
<!--
Custom build images are not the highest amount of customization, but they are a good bit of work. If you want to customize the build image, then you need to create your own build and run image set. This can be done with a standard Dockerfile and some custom metadata.

For example, if you need to use your company's golden docker image for build and run images. You'd create a Dockerfile for the build image, a Dockerfile for the run image, both of which would base off your company's golden image, and then you'd add the CNB metadata. Finally, publish them to a registry and you have CNB suitable build + run images.

Instructions for [build](https://buildpacks.io/docs/for-platform-operators/how-to/build-inputs/create-builder/build-base/) and [run](https://buildpacks.io/docs/for-platform-operators/how-to/build-inputs/create-builder/run-base/). Once you have those, you then create the [builder](https://buildpacks.io/docs/for-platform-operators/how-to/build-inputs/create-builder/builder/).
-->

---

# Levels of Customization

Custom Buildpacks

- Enforce requirements
- Add files or resources to the image
- Set env variables/embed env variables into image
- Custom image labels
- Add CA certificates
- Interact with other buildpacks

<!--
Custom buildpacks let you tap into the build process. You can do virtually anything with them. They have the power to break a build, so you can use them to enforce requirements. They can add layers, set env variables, add CA certificates, and interact with other buildpacks through the build plan.

A minimal buildpack is simple. You can write one with just a couple bash scripts and a `buildpack.toml` file. If you're doing basic things, that's where I would recommend you start.

Buildpacks can do almost anything though, but as you do more the complexity goes up and you may not want to implement them in bash scripts. Bash scripts can't be tested easily, and the complexity of them grows fast. You can write buildpacks in other languages too, although it helps to have one with a buildpacks SDK. There are buildpacks SDKs in Python, Go and Rust.

If you notice, these are mostly statically compiled languages. That is because it's easier to ship buildpacks that way. If you use Python, or another language that requires a runtime, then your buildpacks will only run on a base image that includes a runtime for that language. Static binaries don't need an external runtime, so they are more versatile.
-->

---

# CI/CD Builds

- `pack` cli runs in some CI/CD systems like GitHub Actions
- `kpack` can build on top of Kubernetes
- CircleCI, Gitlab, Jenkins and Tekton are also supported
- `lifecycle` direct

<!--
For some CI/CD systems, you can just run `pack` directly. For example, GitHub actions have a Docker Daemon available, so you can use pack to do builds.

There are some CI/CD systems that run in containers, so don't have a Docker daemon (don't mess with Docker in Docker). These have specific integrations provided by the CNB project, CircleCI, Gitlab, Jenkins, and Tekton fall in this category.

Finally, there are some CI/CD systems where you can pick the image in which your job will run. Concourse is like this. In this case, you can set the image for the job to be your builder image. Then you can invoke the lifecycle directly and it will execute the build. This is essentially what pack is doing for you, it creates a container with the builder and runs the lifecycle. In this case, the CI/CD system is creating the container and we manually invoke lifecycle to run the build.

The only note with this last approach is that you will need to publish directly to the OCI registry, since there is no daemon. This can actually be faster though, so it's not necessarily a bad thing.
-->

---

# Enterprise Ready

Paketo Buildpacks Support

- Custom CA certificates
- Proxies for Internet access
- Local mirrors
- Air Gapped Environments

---

# Summary

- Great for Ops Teams
- Great for Dev Teams
- Everyone can is happy

<!--
// TODO: summary
-->

---

# Slides

<div class="center-img">

![drop-shadow height:12em](https://raw.githubusercontent.com/dmikusa/buildpacks-for-ops/refs/heads/main/slides/img/qr-code.png)

</div>
<div style="text-align: center; padding-top: 0.5em;">

[https://github.com/dmikusa/buildpacks-for-ops]()

</div>

<!--
Slides are available at the link above.
-->

---

<style scoped>
img {
    padding-top: 1.5em;
    height: 300px;
    width: 300px;
}
</style>

# Questions?

<div class="only-img">

:thinking:

</div>
