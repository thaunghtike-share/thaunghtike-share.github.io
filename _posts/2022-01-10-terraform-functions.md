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
<h3> Split Function </h3>

split produces a list by dividing a given string at all occurrences of a given separator.

```bash
split(separator, string)
```
<h4> Examples </h4>

```bash
> split(",", "foo,bar,baz")
[
  "foo",
  "bar",
  "baz",
]
> split(",", "foo")
[
  "foo",
]
> split(",", "")
[
  "",
]
```
<h3> Length Function </h3>

length determines the length of a given list, map, or string.

If given a list or map, the result is the number of elements in that collection. If given a string, the result is the number of characters in the string.

```bash
> length([])
0
> length(["a", "b"])
2
> length({"a" = "b"})
1
> length("hello")
5
```
<h3> Lookup Function </h3>

lookup retrieves the value of a single element from a map, given its key. If the given key does not exist, the given default value is returned instead.

```bash
lookup(map, key, default)
```
<h4> Examples </h4>

```
> lookup({a="ay", b="bee"}, "a", "what?")
ay
> lookup({a="ay", b="bee"}, "c", "what?")
what?
```
NOTE: Terraform does not allow you to create your own functions, so you’re bound to using what is provided by default.

<h1> Terraform Expressions </h1>

Expressions are used to refer to or compute values within a configuration. The simplest expressions are just literal values, like "hello" or 5, but the Terraform language also allows more complex expressions such as references to data exported by resources, arithmetic, conditional evaluation, and a number of built-in functions.

Expressions can be used in a number of places in the Terraform language, but some contexts limit which expression constructs are allowed, such as requiring a literal value of a particular type or forbidding references to resource attributes. Each language feature's documentation describes any restrictions it places on expressions.

You can experiment with the behavior of Terraform's expressions from the Terraform expression console, by running the terraform console command.

<h2> Terraform Operators </h2>

<h3> Arithmetic Operators </h3>

The arithmetic operators all expect number values and produce number values as results:

- a + b returns the result of adding a and b together.
- a - b returns the result of subtracting b from a.
- a * b returns the result of multiplying a and b.
- a / b returns the result of dividing a by b.
- a % b returns the remainder of dividing a by b. This operator is generally useful only when used with whole numbers.
- -a returns the result of multiplying a by -1.

Terraform supports some other less-common numeric operations as functions. For example, you can calculate exponents using the pow function.

```bash
> pow(3, 2)
9
> pow(4, 0)
1
```
<h3> Equality Operators </h3>

The equality operators both take two values of any type and produce boolean values as results.

- a == b returns true if a and b both have the same type and the same value, or false otherwise.
- a != b is the opposite of a == b.

<h3> Comparison Operators </h3>

The comparison operators all expect number values and produce boolean values as results.

- a < b returns true if a is less than b, or false otherwise.
- a <= b returns true if a is less than or equal to b, or false otherwise.
- a > b returns true if a is greater than b, or false otherwise.
- a >= b returns true if a is greater than or equal to b, or false otherwise.

<h3> Logical Operators </h3>

The logical operators all expect bool values and produce bool values as results.
    
- a || b returns true if either a or b is true, or false if both are false.
- a && b returns true if both a and b are true, or false if either one is false.
- !a returns true if a is false, and false if a is true.
    
Terraform does not have an operator for the "exclusive OR" operation. If you know that both operators are boolean values then exclusive OR is equivalent to the != ("not equal") operator.    
 
<h3> Conditionals </h3>

Sometimes, you might run into a scenario where you’d want the argument value to be different, depending on another value. The conditional syntax is as such:

```bash
condition ? true_val : false_val
```
The condition part is constructed using previously described operators. In this example, the bucket_name value is based on the “test” variable—if it’s set to true, the bucket will be named “dev” and if it’s false, the bucket will be named “prod”:

```bash
bucket_name = var.test == true ? "dev" : "prod"
```
<h3> Splat Expressions </h3>

A splat expression provides a more concise way to express a common operation that could otherwise be performed with a for expression.

If var.list is a list of objects that all have an attribute id, then a list of the ids could be produced with the following for expression:

```bash
[for o in var.list : o.id]
```
This is equivalent to the following splat expression:

```
var.list[*].id
```
Do note, that this behavior applies only if splat was used on a list, set, or tuple. Anything else (Except null) will be transformed into a tuple, with a single element inside, null will simply stay as is. This may be good or bad, depending on your use case.

<h3> Terraform Count </h3>

Count is the most primitive—it allows you to specify a whole number, and produces as many instances of something as this number tells it to. For example, the following would order Terraform to create ten S3 buckets:

```bash
resource "aws_s3_bucket" "test" {

 count = 10

[...]

}
```
When count is in use, each instance of a resource or module gets a separate index, representing its place in the order of creation. To get a value from a single resource created in this way, you must refer to it by its index value, e.g. if you wished to see the ID of the fifth created S3 bucket, you would need to call it as such:

```bash
aws_s3_bucket.test[5].id
```
Although this is fine for identical, or nearly identical objects, as previously mentioned, count is pretty primitive. When you need to use more distinct, complex values – count yields to for_each.
