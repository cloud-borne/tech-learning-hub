---
title: Containers
linktitle: Containers
type: book
date: "2019-05-05T00:00:00+01:00"
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 30
---

What is a Container

<!--more-->

## Overview

{{% callout note %}}

A container is a standard unit of software that packages up code and all of its dependencies, so the application runs quickly and reliably from one computing environment to another.

> as per Docker.com

{{% /callout %}}

But what does that mean! :thought_balloon:. Let's take an analogy:

Containers are more like laptop computers than desktop computers.

![Containers Analogy](/images/uploads/desktop-computer-vs-laptop.jpg)

So here we have a desktop computer and a laptop computer, but our desktop has certain dependencies. If we're going to use a desktop, I have to set up and manage multiple external components in order to use it. So I need my own separate screen. I need a computer mouse. I need a keyboard, all these different external dependencies that are not part of the desktop computer itself because this desktop computer requires so much infrastructure, all these external components. It's not very portable. I can't easily take a desktop computer to a coffee shop and I definitely can't use it on an airplane. So it kind of has to remain in place because of all of these external dependencies.

A laptop computer on the other hand has everything that I need packaged into a single device. So all of those external dependencies are essentially packaged into the laptop. It contains its own screen. It contains its own touchpad. It contains its own keyboard. Everything's all packaged together and this makes the laptop computer more portable and allows it to run anywhere. So I can go use it at the coffee shop, or I can use it on an airplane because everything I need is all in that single device. And that's essentially what containers do for software.

They take all of the components that your software needs and put them all together into one portable package and run Anywhere i.e **On-Prem** or in the **Cloud** seamlessly.

I don't have to worry about whether the production or development or cloud servers have different operating system dependencies than my laptop that might cause my code not to work. It takes out that ```Works in my Machine``` problem out of the park.

![Containers Analogy](/images/uploads/works-on-my-machine.jpg)

## Quick Start

* Test your setup:
  ```
  docker version
  ```
* Create a directory to house your Docker project.
  ```
  mkdir my-website
  cd my-website
  ```
* Create a simple homepage for your website.
  ```
  vi index.html
  ## Add some basic content to the html file.
  Hello, World!
  ```
* Create a Dockerfile.
  ```
  ## This will create an image with Nginx and include our html file as the default homepage.
  FROM nginx:stable
  COPY index.html /usr/share/nginx/html/
  ```
* Build the image.
  ```
  docker build -t my-website:0.0.1 .
  ```

* Test the image.
  ```
  docker run --rm --name my-website -d -p 8080:80 my-website:0.0.1
  curl localhost:8080
  ```

* Clean up the container.
  ```
  docker stop my-website
  ```

* Save the image to an archive.
  ```
  docker save -o /home/cloud_user/my-website_0.0.1.tar my-website:0.0.1
  # View the archived image file.
  ls /home/cloud_user
  ```
