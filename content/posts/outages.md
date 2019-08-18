---
title: "Outages? Downtime?"
date: 2018-12-03T19:13:51-07:00
draft: true
---

## Looking Back

Life is funny sometimes, when everything is rolling along according to plan, it seems like that's when one should _most_ be on their toes. It's almost expected to think that **during** the week of my year anniversary at SourceClear, the largest outage (complete with a two day stretch of down time over multiple teams) of my career would hit. What's the _best_ part of this story? The outage was caused in no small part, by yours truly. So for the sake of documentation, and transparency, follow along with me as we explore what I did wrong to break everything, and what _we_ did right to fix it. 

So to start, this was an out in our QA/Dev Kubernetes cluster, not production cluster. I can hear you now...
> "QA outage? I'd _kill_ for a QA Outage! That's not real downtime! 

Well you know what? When the environment you're in charge of has been pretty stable under your watch, any downtime is downtime. Any lost productivity is a bummer, so there you go.

## Let's Break It Down...then fix it.

The cause of the issues? A lack of attention to detail while planning upgrades, and unfamiliarity with tooling. No excuses, this was 100% avoidable.

But the TLDR? It was fixed, I (we, there were others involved) learned from it. And we're better because of it, and we won't make that mistake again.

### First Mistake

The back story is that we have been preparing for an AWS region migration, and I have been spinning up (and then tearing down) multiple Kubernetes clusters daily, using [kubespray](https://github.com/kubernetes-sigs/kubespray), to ensure I know what to do when it doesn't work, and document all steps involved in the process.

During one of these cycles, I accidentally deleted some IAM roles (but I did re-add them!) with [Terraform](https://www.terraform.io/). Turns out, IAM roles and instance profiles for EC2 instances? Kubernetes _needs_ those for the `cloud-controller-manager`! (Yes...I know those are needed. It's not like I deleted them with malicious intent...) I re-added the roles with the proper permissions baed on the `kubespray` documentation, and all seemed fine. Spoiler: Things were not fine. 

The signs of trouble we first saw (which later came back around a week later) were errors in the `journalctl` logs from our friend `kubelet`, along the lines of "no credentials in the provider chain", in other words, "kubernetes needs to make changes to the cloud provider, but I'm not authorized." Well I had re-added the instance profile and IAM roles weeks ago, so that couldn't be the issue.  

So _since_ the cluster was provisioned with `kubespray`, after a few hours of manual troubleshooting, the team and I opted to attempt to rerun `kubespray`'s `cluster.yml` playbook. This _should_ detect any anomalous settings and such in our cluster, and make those changes back to what is defined in the playbook. Turns out, that lead to mistake two. 

### Second Mistake 

So I'm not quite _sure why_ I thought that a `kubespray` run would fix things. It doesn't lay anything down for IAM roles (that's terraform), but I guess at this point I was grasping at straws? We had been seeing some intermittent issues with `kubelet` since the week of Thanksgiving, and as some (probably more than I think) people know, troubleshooting on a holiday break is no fun. 

So I attempt to re-run `kubespray` and it fails once or twice, I've been having issues with later versions of `kubespray` (hence the reason I've started looking closer at `kops`) but I eventually had a successful run, and to no one's surprise, it didn't fix anything, in fact, it made things worse...

It was at that point I realized, that `kubespray` had run successfully, with the default variables, _not_ my specific environment variables. So at this point, everything _not_ in the  `kube-system` namespace was falling over. But wait, it get's **better**.

I hadn't known this was possible, but looking back on it, I see no reason _why_ it shouldn't be possible, but due to a misplaced variable in my vars file....I now had **two** copies of all the control plane pods.

> What's that? We laid down **duplicate** deployments of the `api-server`, `controller-manager` and `scheduler`?

> Oh and now this cluster has **two** CNI plugins fighting to network my pods?

> Oh **AND** it changed the API-sever ELB to the wrong port!?

So at this point I was looking around for other things to try and get me back in a steady state, but it was rather late in the evening, we'd hit 13 hours in the day. It was time to call it a night, our QA outage would roll into day 2. This is especially painful when you have a team based ~12 hours away that was looking to use this cluster during their day, I had wasted a US based teams day, and would now waste another's.

### End of Day One

Multiple user errors on my part had caused an issue with the Instance Profile for the Kubernetes Nodes, and a botched `kubespray` run lead to default environment variables adding a second CNI Plugin to the cluster, as well as duplicate control plane pods.


## Day Two

The next day, with a good night's sleep, I learn by reading the Ansible docs, that the `group_vars` are based on the relative location of the Ansible `inventory` file. Well I should mention that I had checked in my inventory files to _another_ git repo, that I was referencing by absolute path. And _that_ was where Ansible/Kubespray was looking for environment variables. Oh joy. 

On a hunch, prior to the re-run of `kubespray`, I checked my IAM roles. And as it turns out, I had missed a permission for the role. It didn't match what we had in another account's cluster. Adding that permission **immediately** resolved the issue with the `kubelet` logging it's inability to auth to the cloud provider. Now as a reminder, that was the _initial_ issue, and by trying to fix that, I had just made things _worse_.

So with IAM straightened out, we go back to `kubespray`. I moved my inventory file to where it needed to be, and we kicked off another `kubespray` run. I'd lost track of how many times I'd run that `ansible-playbook` command. This time? It worked! We were down to the proper number of control-plane bits and pieces. But we still had two CNI plugins, and I can't run `kubectl` commands. Ok then!

Let's get low hanging fruit first. Scale the second CNI plugin to 0 pods, and fix the ELB port. Back in business. Well halfway in business. We were so close to being in an _almost_ good spot. Our cluster was up and accessible, we could make API calls, and pods were scheduling...but

Endless `CrashLoopBackOff` on all new pods. _WHYYYYY?_

### Third Mistake

At this point, the team started looking more into DNS. If the CNI is correct, then maybe it's a `KubeDNS` issue? We tried a rolling restart on all those and see if anything clears up, but to no avail. I noticed at this point that `KubeDNS` was actually _removed_ from Kubernetes 1.13, and while we weren't running 1.13 yet, maybe this was the right time to tear out `KubeDNS` and add `CoreDNS`?

Well that's just what I did. 

I ran the script to replace `KubeDNS` with `CoreDNS` and waited. No luck. So now we check the `CoreDNS` config, well it turns out I have _never_ used `CoreDNS`, and a Core File (the `CoreDNS` config file) is foreign to me. A little poking around in the documentation and I found out what I needed in my Core File...and we were already set up properly.

Now you may think this wasn't a mistake, and it truly wasn't, but it was a decent chunk of wasted time as `KubeDNS` was in no way causing issues, but we _did_ get the current Kubernetes DNS when all was said and done, something that would have had to happen eventually.

### Things Start Looking Up

This is when we hit a turning point of our troubleshooting process. It turns out, another one of those `kubespray` default values was a pod CIDR. Our desired CNI plugin was sitting in the right subnet, but pods were still getting the wrong CIDR, but from _where_? I have removed the secondary plugin. 

I started going down my list of things I know to check (I'm sure I missed some, but it was late afternoon on day two. Patience was wearing thin.)

Stop/start `kubelet`.  
Bounce the CNI networking pods  
What about the Docker network?  
Nothing. No luck. All new pods have the incorrect (`kubespray` default) CIDR. But _where_ are they getting that value from?! There aren't any rogue CNI bits left in the cluster, right? No pods, nothing!

Finally...on a hunch, we deleted a config file, _on_ the node, and rebooted the node  
  
**IT**  
**WORKED**  

Turns out that secondary CNI pods had left a config file in the `/etc/cni/net.d/` directory. This is the directory that our chosen CNI reads from (populated by ConfigMap) but for some reason, that file (`10-<secondary-plugin>.conf`) was causing issues by coexisting in that directory with `10-<primary-plugin>.conf`.

So we **NUKED** those secondary configs from orbit, reboot all our nodes...

wait for it...  

wait for it...

PODS! WITH PROPER CIDRs! Miraculous! 

So all we have to do is go through all the nodes and delete that file and reboot? I know how to do all those things! (Side Note: We could have most likely done less than a full reboot, but in the interest of being thorough, we did one anyway). It was at this point, I made my...

### Fourth Mistake

I began moving through the list of nodes. `ssh` into the node, `rm` the config file in question, and `sudo reboot`. Piece of cake. I get a message from another engineer that I had missed a node. We have a decent number so that is entirely possible. They're already `ssh`-ed in so they handled it. Then they found another I missed, and another. 

I was offended. How could they assume me of missing more than half? I asked what commands they ran, are they rebooting through the console or terminal? It all lined up! What did I do wrong? So I asked...

> "What file are you deleting?"

> "`10-<secondary-plugin>.conf`, what did you delete?"

> "Oh, I deleted `10-<primary-plugin>.conf`"

_Why_ did I delete the config file from the CHI Plugin that we wanted you might ask? Well I have a valid reason, that definitely sounded more intelligent before I actually typed my reply. I thought that perhaps that config was corrupted in some way, and deleting it would load a new config from the `ConfigMap`. I promise it sounded good in my head, but I needed to delete `10-<secondary-plugin>.conf`, as that was causing the issues. Well now I know, and believe me, this other engineer with **never** let me live this down. We deleted the proper file (as I had deleted the wrong file on _all_ the nodes) and everything came back up to were we wanted to be. Service restored, and it only took two days. 

### What Did We Learn? 

Looking back on all this wonderful time spent troubleshooting, overly nervous that I'm wasting my developers time, and my own time meant for other planned things, I had some take aways. 

This was started by a mistake. A medium sized mistake, but a mistake I made nonetheless. Two days of down time cascaded from missing a detail in my process. I then followed that up with 3-4 other mistakes that got us no closer to fixing the issue. 

I am proud of my troubleshooting process for the most part. I think while in the thick of things, I was asking the right questions and looking in the right places. By being unaware the Instance Profile issue persisted, all my other troubleshooting wasn't helping, and may have in fact made things worse. 

But, this is where it all pays off. I took a lot of notes. A LOT. And I learned about some parts of the infrastructure that I hadn't had a lot of exposure to. I also learned a lot about `CoreDNS` and CNI Plugins and got to configure both, even though the DNS was never the issue, and the CNI Plugins were "too configured" if that is possible. 

So to sum up. It wasn't production downtime but it was still a wild ride. It was not as scary as it could have been, especially if I wasn't wasting developer time, but customer time too. A shout out to the team that helped me resolve this, and the teams I held up from writing code, for being patient as I waded through a mess of my own creation. We all came right-side up at the end, and if we don't learn from our mistakes, then we're not doing it right.