---
title: "Options Pattern With Configuration Reload in .NET"
date: 2023-10-08T00:00:00-07:00
draft: false
---

In summary, IOptions provides no reload capabilities. Once the configurations are registered during start-up, any changes to a configuration will not take effect within the application. On the other hand, IOptionsSnapshot and IOptionsMonitor have reload capabilities, and changes to a configuration will eventually be portrayed within the application. The main difference between IOptionsSnapshot and IOptionsMonitor is their lifetimes.

...

This is only a snippet of the post, the rest can be found on the repo itself: https://github.com/GeovanniG/OptionPatterns