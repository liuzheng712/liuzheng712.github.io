---
layout: post
title: "Activating a volume group"
description: ""
category: 
tags: [运维]
---
{% include JB/setup %}

After rebooting the system or running vgchange -an, you will not be able to access your VGs and LVs. To reactivate the volume group, run:

    # vgchange -ay my_volume_group

