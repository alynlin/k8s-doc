---
title: "Kubesphere in Openshift" # Title of the blog post.
date: 2020-09-27T13:59:28+08:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Technology
tags:
  - kubesphere
  - openshift
---
### kubesphere in openshift
```shell script
oc adm policy add-scc-to-user privileged system:serviceaccount:kubesphere-system:ks-installer

###
oc adm policy add-scc-to-user anyuid system:serviceaccount:kubesphere-system:redis-ha-haproxy
oc adm policy add-scc-to-user anyuid system:serviceaccount:kubesphere-system:redis-ha
oc adm policy add-scc-to-user privileged system:serviceaccount:kubesphere-system:openldap-admin
# muti cluster
oc adm policy add-scc-to-user anyuid system:serviceaccount:kube-federation-system:kubefed-controller
oc adm policy add-scc-to-user anyuid system:serviceaccount:kube-federation-system:kubefed-admission-webhook

# logging
oc adm policy add-scc-to-user hostaccess system:serviceaccount:kubesphere-logging-system:fluentbit-operator
oc adm policy add-scc-to-user hostaccess system:serviceaccount:kubesphere-logging-system:fluent-bit
oc adm policy add-scc-to-user anyuid system:serviceaccount:kubesphere-logging-system:elasticsearch-logging-data
oc adm policy add-scc-to-user anyuid system:serviceaccount:kubesphere-logging-system:elasticsearch-logging-master




oc apply -f https://raw.githubusercontent.com/kubesphere/ks-installer/v3.0.0/deploy/kubesphere-installer.yaml
oc apply -f https://raw.githubusercontent.com/kubesphere/ks-installer/v3.0.0/deploy/cluster-configuration.yaml
```
