render-infrastructure-as-code-headstart
# Render Infrastructure-as-Code - Headstart

Based on "Infrastructure as Code (IaC)" at https://render.com/docs/infrastructure-as-code

## 100 - Introduction

You can use your Render dashboard to create and manage services, but if you have a lot of services or your services require a lot of options, it can be helpful to define your Render infrastructure (services, databases, and environment groups) [as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) in a ```render.yaml``` file.

You can add this file to source control to track changes to your infrastructure over time. Render also automatically updates your infrastructure when you push changes to ```render.yaml```, so your Render resources always remain in sync with the values defined in your code.
