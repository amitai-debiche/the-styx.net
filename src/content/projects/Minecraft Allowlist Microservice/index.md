---
title: "Minecraft Allowlist Microservice"
description: "Automates allowlist management for Minecraft servers"
date: "Sep 9 2024"
demoURL: ""
repoURL: "https://github.com/amitai-debiche/minecraft_allowlist_automation"
---

Automates allowlist management for a Minecraft server, letting users request access through a web form or Discord, and
letting privelged discord users approve or deny those requests - then applying changes automatically so admins don’t
have to manually edit server configs.

## Overview

Minecraft Server Allowlist Microservice is a containerized, RESTful service featuring both a
web-form frontend, an integrated Discord bot, and a server agent. Instead of manually adding or removing players from
the
allowlist through server console or config files, this system lets users request access via web or Discord, and lets
server moderators or admins approve/deny those requests. Simplifying and automating the whole access-management
process.

### Inspiration

I host a Minecraft Server on my own infrastructure, and it used to be tedious to constantly whitelist my friends and
friends of friends. So instead I spent way longer than manually entering them ever would and figured out a way to
automate the process :).

## Key Features

- User Request Interface (Web + Discord): Provides a web form where players can request access.

- Privileged-User Approval Workflow: Privileged users (admins/moderators) can approve or deny requests,
  allowing controlled, role-based access management.

- Automatic Server Agent: Once a request is approved, the server-side agent automatically applies the allowlist change
  to
  the Minecraft server configuration.

- Docker-Friendly Deployment: The entire service is containerized for easy deployment, portability, and separation from
  the main Minecraft server process.

## Impact

This microservice transforms allowlist management from a manual, annoying chore into a streamlined and user-friendly
workflow. Users can request access without needing direct server permissions; admins can control membership through an
approval step; and server configuration changes happen automatically. For communities (especially small-mid or
private servers), this reduces admin workload, lowers error risk, and improves accessibility and security for server
membership control.

## Future

All layers: Improve observability of services, add reporting for health checks, cleanup codebase.
Backend: Improve error handling, same user requests
Frontend: Needs work on image load time
