---
icon: spider
cover: ../.gitbook/assets/image (1).png
coverY: 0
layout:
  width: default
  cover:
    visible: true
    size: hero
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
---

# NetExec

## <mark style="color:$primary;">ABOUT</mark>

<mark style="color:$danger;">**NetExec**</mark> (a.k.a `nxc`) is a <mark style="color:purple;">**network service exploitation tool**</mark> that helps automate assessing the security of large networks.

{% hint style="info" %}
If you are seeing somewhere the commands with crackmapexec, you can just change it to nxc because it inherited most of the syntax, and you can say netexec is continuation/evolution of crackmapexec
{% endhint %}

## <mark style="color:$primary;">MODULES</mark>

The killing feature of nxc is modules, which is automated post-exploitation scripts. As example you can use `--coerce_plus` module to check and execute 5 coercing techniques for Relay attacks.

## <mark style="color:$primary;">RESOURCES</mark>

{% embed url="https://www.netexec.wiki" %}

{% embed url="https://www.netexec.wiki/getting-started/using-modules" fullWidth="false" %}
