---
layout: post
title: "Drupal 8 release candidate is officially out"
description: "After anouncement in Barcelona by @Dries Drupal 8 release candidate was released on October 7"
thumb_image: "posts/d8-out.png"
tags: [web, drupal]
---

{% include image.html path="posts/d8-out.png" path-detail="posts/d8-out.png" alt="Drupal8 rc" %}

[Drupal 8 Release candidate one](https://www.drupal.org/drupal-8.0.0-rc1) is out!

After anouncement in Barcelona by [@Dries](https://twitter.com/Dries) Drupal 8 release candidate was released on October 7\. Time to start using Drupal 8 in a more serious way now that we are at zero criticals.

Still there is a lot of modules that need porting to Drupal 8 so development will still be slow and some major issues might still arrive during RC phase.

However its now easier and safer than ever to start working on sites with Drupal 8. Acquia is already pushing Drupal8 to their customers and this is a sign that there is some pressure to get things going on a more quick paste.

## How can we start using the latest version of Drupal

### Launching new sites

With Drupal 8 many of the top Drupal 7 modules are now included in core like Views or REST and several features like translation interfaces have been made more flexible to avoid the need to install many other modules. Evaluate your needs, and you may easily find that everything you need for a project is already included in Drupal 8 core. Check out the [slides about Drupal 8](https://docs.google.com/presentation/d/1GXK1dBSe6_QMhSkNwsgocWynlzdQFrMUouaOqA8wyUI) to learn about the changes introduced from Drupal 8.

### Updating existing sites

The new version of Drupal also includes a brand new Migrate module to update existing Drupal 6 and 7 sites to Drupal 8 directly. The migration feature is currently marked _"experimental,"_ meaning it is not yet fully supported and there is some work still being made to improve it. For this reason, the Drupal 8 release candidate does not yet provide a user interface for migrations. [Use the Migrate Plus and Migrate Upgrade modules](https://www.drupal.org/node/2257723) to test migrations now, or [read more about Migrate in core](https://www.drupal.org/upgrade/migrate).

### Contributed modules and themes

There are [a number of modules already ported to Drupal 8](https://www.drupal.org/project/project_module?f%5B0%5D=&f%5B1%5D=&f%5B2%5D=&f%5B3%5D=drupal_core%3A7234&f%5B4%5D=sm_field_project_type%3Afull&text=&solrsort=iss_project_release_usage+desc&op=Search) as well as [themes already being developed](https://www.drupal.org/project/project_theme?f%5B0%5D=&f%5B1%5D=&f%5B2%5D=drupal_core%3A7234&f%5B3%5D=sm_field_project_type%3Afull&text=&solrsort=iss_project_release_usage+desc&op=Search). [The contrib tracker project](https://www.drupal.org/project/contrib_tracker) makes it easier to track the status of the ports of contributed modules.