---
title: "Nuking a Fly with an Orbital Laser"
description: "Setting up K3 cluster with ArgoCD and GitHub ARC just to host a static Astro site."
date: "November 30 2025"
draft: False
---

**Hello World! Welcome to the first post on my blog**

Usually, when people launch a personal portfolio site using Astro, they do the sensible thing. They connect their GitHub
repo to Vercel or Netlify, click "Deploy," and go enjoy a coffee. It takes about 45 seconds. It is free. It is reliable.

Naturally, I looked at that efficient, cost-effective workflow and thought, "Absolutely not. Where is the suffering?
Where is the unnecessary complexity?"

So, instead of a managed CDN, I decided to deploy this static collection of HTML and CSS onto a bare-metal, colocated
server running a custom Kubernetes cluster with self-hosted CI/CD pipelines.

Is it overkill? Yes. Is it essentially using a Ferrari to drive to the mailbox at the end of the driveway? Also yes. But
is it cool? Definitely yes.

### The "Hardware": My Slice of the Cloud

This all runs on my colocated HP DL380 server. I wanted full control, which is another way of saying I wanted full
responsibility for when things break at 3 AM.

Software-wise, I’m running K3s[^1] (though I’m flirting with the idea of switching to RKE2[^2]). While this process wasn't completely
painless learning about Helm[^3], Ingress[^4] and more, I chose this over spinning up raw Kubernetes from scratch because while I enjoy pain, I am only a partial masochist.

I’ve split the setup into 2 Virtual Machines:

- A Single Control Plane: My K3S Server Node
- The Worker Nodes: Where my K3S agents live

An edited architecture diagram from K3s might help visualize this:
<figure>
<picture class="block dark:hidden">
  <img src="/k3s-architecture-single-server.svg" alt="Diagram for light mode" />
</picture>

<picture class="hidden dark:block">
  <img src="/k3s-architecture-single-server-dark.svg" alt="Diagram for dark mode" />
</picture>

<figcaption class="mt-2 text-xs text-gray-500 dark:text-gray-400">
  Image Source: 
  <a
      href=" https://docs.k3s.io/architecture"
      target="_blank"
      rel="noreferrer"
      class="underline hover:no-underline"
  >
    https://docs.k3s.io/architecture
  </a>
</figcaption>
</figure>

**A Note on "High Availability[^5]"** One may note that a single Control Plane and Worker node is not 
High Availability (HA). It is, in fact, a Single Point of Failure (SPOF).

However, this architecture is "eventually consistent" — meaning eventually, I will buy more hardware/eventually set up those other 2 server and worker nodes. 
If anyone has two spare servers collecting dust that they’d like to donate to the cause, my DMs are open. 
Help me achieve the dream of a fully HA, 3-node server cluster on top of a 3 Server Node Kube cluster
for a static website that updates once a month.


### The Full Picture

To make this setup complete, I needed a few key players in the new cluster:

- ArgoCD[^6]: For GitOps.
- GitHub ARC[^7] (Actions Runner Controller): Self-hosted GitHub runners. Why use GitHub’s free cloud runners when I can
  burn my own CPU cycles?
- This Site: The site you are on right now. 

### The Workflow

Here is the life cycle of any potential change on this website.

If I were using Vercel, the flow would be: Push Code -> Site Updates. I copied that... on my own infrastructure.

So my flow is: Push Code -> See Things Light Up Green -> Site Updates.

#### The Workflow, but a bit more indepth ####
<!-- empty line -->
1. The Push: I push a commit or tag to the ```main``` branch, this triggers the GitHub Action
2. The Runner Wakes Up: The GitHub ARC controller on my cluster sees the job and spins up a runner pod on my Worker VM.
3. The Build: The runner builds the Astro site into a container image and pushes it to GitHub's container registry, no need
for a private registry for this site.
4. The GitOps change: The runner doesn't deploy the app. Instead, I have the Action use sed to update the image name
in [k8/deployment.yaml](https://github.com/amitai-debiche/the-styx.net/blob/main/k8/deployment.yaml) to match the new container image.
5. The Watcher & Sync: ArgoCD notices the manifest changed in the repo triggering a Sync, applying the new manifest to the cluster API. 
Kubernetes sees the update image in the manifest, terminating old pods and pulling the fresh image to start new ones.

<figure>
<picture class="block dark:hidden">
  <img src="/pipeline_flowchart.svg" alt="Diagram for light mode" />
</picture>

<picture class="hidden dark:block">
  <img src="/pipeline_flowchart-dark.svg" alt="Diagram for dark mode" />
</picture>

<figcaption class="mt-2 text-xs text-gray-500 dark:text-gray-400">
  Image Source: 
    Me, which is why the SVG is ugly. Creating pretty SVG images is hard...:(
    <p>
Maybe I should have used draw.io not inkscape but too late now
    </p>
</figcaption>
</figure>


### Why did I do this?

Make no mistake, serving static files via Nginx inside a container, inside a pod, inside a VM, inside a
physical server is the definition of overkill for a home setup.

But, it was a fantastic way to learn. Setting up GitHub ARC gave me a deep appreciation for ephemeral build
environments. Configuring ArgoCD forced me to be disciplined about declarative infrastructure.

And most importantly, now when I update a typo on my "About Me" page, I can watch a dozen different dashboards light up
green. And really, isn't that what software engineering is all about?

#### The Source Code
Want to see the chaos yourself? Almost the entire infrastructure for the site is Open Source. Feel free to 
[check out the repository here](https://github.com/amitai-debiche/the-styx.net) to poke around my configuration.

--- 
**Stay tuned for part two, after I set up enterprise-grade monitoring and alerting and explain the process for a blog 
that probably gets less than three visitors a month.**

[^1]: https://docs.k3s.io/
[^2]: https://docs.rke2.io/ 
[^3]: https://helm.sh/
[^4]: https://kubernetes.io/docs/concepts/services-networking/ingress/
[^5]: https://en.wikipedia.org/wiki/High_availability
[^6]: https://argo-cd.readthedocs.io/en/stable/
[^7]: https://docs.github.com/en/actions/concepts/runners/actions-runner-controller