---
title: "Operator Sdk" # Title of the blog post.
date: 2020-09-08T11:30:07+08:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
thumbnail: "/images/operator_logo_sdk_color.svg" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - openshift
  - k8s
---


## Documentation

Docs can be found on the [Operator SDK website][sdk-docs].

## Overview

This project is a component of the [Operator Framework][of-home], an
open source toolkit to manage Kubernetes native applications, called
Operators, in an effective, automated, and scalable way. Read more in
the [introduction blog post][of-blog].

[Operators][operator-link] make it easy to manage complex stateful
applications on top of Kubernetes. However writing an operator today can
be difficult because of challenges such as using low level APIs, writing
boilerplate, and a lack of modularity which leads to duplication.

The Operator SDK is a framework that uses the
[controller-runtime][controller-runtime] library to make writing
operators easier by providing:

- High level APIs and abstractions to write the operational logic more intuitively
- Tools for scaffolding and code generation to bootstrap a new project fast
- Extensions to cover common operator use cases

## License

Operator SDK is under Apache 2.0 license. See the [LICENSE][license_file] file for details.

[controller-runtime]: https://github.com/kubernetes-sigs/controller-runtime
[license_file]:./LICENSE
[of-home]: https://github.com/operator-framework
[of-blog]: https://coreos.com/blog/introducing-operator-framework
[operator-link]: https://coreos.com/operators/
[sdk-docs]: https://sdk.operatorframework.io

