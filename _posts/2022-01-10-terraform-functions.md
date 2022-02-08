---
layout:     post
title:   "Terraform Function, Expression And Loop Example"
date:       2022-01-10 15:13:18 +0200
image: 36.png
tags:
    - Terraform
    - IAC
categories: Terraform
---

<h1> Introduction </h1>

Terraform is an open-source infrastructure as code software tool that provides a consistent CLI workflow to manage hundreds of cloud services. Terraform codifies cloud APIs into declarative configuration files.

to manage the entire lifecycle of infrastructure using infrastructure as code. That means declaring infrastructure components in configuration files that are then used by Terraform to provision, adjust and tear down infrastructure in various cloud providers.

In this article, we will discuss the HCL’s core – built-in functions, expressions and loops. What are those, and what are they used for? Let’s dive in.

<h1> Terraform Functions </h1>

We can use built-in Terraform functions to perform various mathematical computations related to a deployment and to perform operations, such as to encode and decode, or to capture and display timestamps. The Terraform language only supports the functions built into it; custom or user-defined functions are not available.

Use this Terraform tutorial to walk through the basics of functions, as well as some common uses for them in enterprise deployments.

<h2> Start with the syntax </h2>

The syntax for a Terraform function starts with the function name, followed by parentheses that contain zero to multiple comma-separated arguments:

function_name(arg-1, arg-2, … arg-n)

[Here](https://www.terraform.io/language/functions) is the official reference for terraform built-in functions.

<h3> Join Function </h3>

Join produces a string by concatenating together all elements of a given list of strings with the given delimiter.

```bash
join(separator, list)
```
<h4> Examples </h4>

```bash
> join(", ", ["foo", "bar", "baz"])
foo, bar, baz
> join(", ", ["foo"])
foo
```

